# sending parameter and result values
### 성현

### 승현

### 용운
함수의 매개변수나, 변환값을 명시적으로 Sendable 상태로 만들기 위한 새로운 애노테이션이다.

지역 격리(SE-0414)를 도입해서 non-sendable 타입의 값이 격리 경계를 안전하게 넘을 수 있도록 변경됨.
대부분의 경우 함수의 인자와 반환값은 호출시 동일한 지역으로 병합됨. 이 경우, non-sendable type은 전송될 수 없음.
(Non-sendable의 경우 다른 격리 지역을 넘어다닐 수 없는 타입)

sending 애노테이션을 통해 이런 제한을 해소하기 위해, 매개변수와 반환값이 함수 경계를 넘을 때 분리되어 안전하게 전송될 수 있는 역할을 한다.

``` swift
class NonSendable {}

// MainActor에서 격리된 함수
@MainActor func main(ns: NonSendable) {}

// 파라미터 ns의 경우, 어느 격리 지역에서 할당받는지 명시되지 않았음. 그렇기 때문에 에러가 발생하는것,
ns가 전달된 격리 구역하고 main함수가 실행되는 구역하고 다를 수도 있기 때문
func trySend(ns: NonSendable) async {
  // error: sending 'ns' can result in data races.
  // note: sending task-isolated 'ns' to main actor-isolated 
  //       'main' could cause races between main actor-isolated
  //       and task-isolated uses
  await main(ns: ns)
}

```

actor 초기화의 규칙
init의 경우 nonisolated이며, 초기화의 매개변수 값은 반드시 엑터 격리 구역으로 보내짐.
그렇기 때문에 초기화가 끝나고 나서 호출자쪽에서는 전달한 인자를 사용할 수 없다.
``` swift
func invalidSend() {
  let ns = NonSendable()

  // error: sending 'ns' may cause a data race
  // note: sending 'ns' from nonisolated caller to actor-isolated
  //       'init'. Later uses in caller could race with uses on the actor.
  let myActor = MyActor(ns: ns)

func send(ns: NonSendable) {  <- sending을 붙이지 않았기 때문에 발생, 지역 격리를 간섭할 경우 더 명확한 기준의 코드를 작성해야함.
  // error: sending 'ns' may cause a data race
 let myActor = MyActor(ns: ns)
}
```

``` swift
@MainActor var mainActorState: NonSendable?

nonisolated func test() async {
  let ns = await withCheckedContinuation { continuation in
    Task { @MainActor in
      let ns = NonSendable()
      // Oh no! 'NonSendable' is passed from the main actor to a
      // nonisolated context here!
      continuation.resume(returning: ns)

      // Save 'ns' to main actor state for concurrent access later on
      mainActorState = ns
    }
  }

  // 'ns' and 'mainActorState' are now the same non-Sendable value;
  // concurrent access is possible!
  ns.mutate()
}
```
격리 경계 변경 정리
withCheckedContinuation <- nonisolated 
Task <- MainActor
return value nonisolated

swift 6 아래 버전에서는 데이터 레이스가 발생할 수 있는 상황임에도 경고가 발생하지 않음.
추가적으로 resume 호출 이후 해당 지역의 값이 다시 사용되지 않는다면, non-sendable 타입의 값을 전달하는 것도 안전하다. <- 이게 포인트임.
모든 값을 sendable로 다룰 필요는 없음 foundation도 아직 준비되지 않았을뿐더러 얘기한것 처럼 너무 엄격한 제한을 통해 사용자가 피곤함을 느낄 가능성이 큼.

Sendable 프로토콜을 준수하는 타입은 thread-safe 타입으로, 해당 값을 여러 동시성 컨텍스트에서 공유되고 안전하게 사용될 수 있다.
non-sendable일 경우, 해당 값을 절대로 동시에 사용되지 않도록 보장해야한다. 
“하지만 동시성 컨텍스트 간에 전송이 불가능한 건 아니다. 값을 다른 동시성 컨텍스트로 보낼 때, 해당 값의 전체 영역이 완전히 이전되어야한다.”
- 원래 동시성 컨텍스트에서 해당 값이 더 이상 사용되면 안된다.
- 새로운 동시성 컨텍스트로 전달된 후에만, 해당 컨텍스트에서 사용될 수 있음.

기존 지역과 연결되지 않는 새로 생성된 값은 항상 “sending”이 된다. sending 값은 다른 격리 지역과 병합될 수 있으며, 병합된 후 해당 지역이 분리되지 않는 상태라면 다른 격리 도메인으로 다시 보낼 수 없다. 즉 sending이 아니게 됨.

sending function parameter는 인자로 전달되는 값이 반드시 분리지역에 속한다.
매개변수 값이 호출자와 연결이 끊어지고, 피호출자가 자유롭게 새로운 격리 지역으로 보낼 수 있다.

반대로 반환값이 sending일 경우, 반드시 분리된 지역에 속하는 값을 반환해야함. 더 이상 기존 격리 영역에 속하지 않음.
``` swift
func acceptSend(_: sending NonSendable) {}

func sendToMain() async {
  let ns = NonSendable()

  await acceptSend(ns)

@MainActor
struct S {
  let ns: NonSendable

  func getNonSendableInvalid() -> sending NonSendable {
    // error: sending 'self.ns' may cause a data race
    // note: main actor-isolated 'self.ns' is returned as a 'sending' result.
    //       Caller uses could race against main actor-isolated uses.
    return ns
  }

  func getNonSendable() -> sending NonSendable {
    return NonSendable() // okay
  }
}

```

sending이라는 규격이 붙으면 격리되지 않는 타입을 적용할 수 없다. sending 키워드가 제약사항을 추가하는 것이기 때문에 당연하다.
일반적인 T 타입의 값을 분리된 지역에 속한다는 가정은 성립할 수 없다.
``` swift
let first: (sending NonSendable) -> Void 
first = (NonSendable) -> Void // error

let first_: () -> sending NonSendable
first_ = () -> NonSendable // error

let second = (NonSendable) -> Void
second = (sending NonSendable) -> Void // ok

let second_: () ->  NonSendable
let second_ = () -> sending NonSendable // ok
```

프로토콜 부분은 프로포절이랑 다른 부분이 존재. 추가 확인 필요

inout sending 매개변수
함수에 전달될 때, 인자 값은 분리된 지역에 속하며, 함수가 반환될 때, 매개변수 값도 분리된 지역에 속하게 된다.
actor 격리된 피호출자와 병합하거나 다른 지역으로 전달할 수 있다. 다만 함수 종료되는 시점에는 해당 매개변수에 분리된 지역에 속하는 값이 할당되어야만 한다.

함수 호출 시, sending 매개변수로 인자를 전달하면 피호출자가 반환된 이후 호출자는 해당 인자 값을 다시 사용할 수 없다.
기본적으로 함수 매개변수에 sending이 적용되면 피호출자가 해당 매개변수를 소비(consume)하는 것과 같은 의미를 갖는다.
매개변수는 소유권이 피호출자로 이전되며, 함수 내부에서 다시 할당 될 수 있다. <- inout 내용

하지만 sending 매개변수는 consuming 매개변수와 달리 암시적인 복사 금지 규칙을 따르지 않는다. 기본 소유권 규칙을 변경하거나 암시적인 복사 방지하려면 sending과 함게 소유권 수정자를 사용해야한다.

명시적인 borrowing 애노테이션은 항상 암시적 복사 금지를 의미한다. sending 매개변수의 기본 소유권 규칙을 변경하려면 반드시 암시적 복사 금지 규칙도 함께 적용해야한다. 즉 sending 매개변수의 기본 소유권 규칙을 변경하는 방법은 잇지만, 암시적 복사 금지를 피하는 방법은 존재할 수 없다.


-> 나중에는 Disconnected types가 나올것이다.

다음 proposals 추천 
[region based isolation](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0414-region-based-isolation.md)
