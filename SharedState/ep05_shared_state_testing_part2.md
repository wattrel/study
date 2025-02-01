# Shared State: Testing, Part 2
일시: 1월 31일 오후 10시

@김성현

내비게이션 스택에서 액션 보내는 방법
```swift
// 1. enum 사용 (가장 기본적이지만 2,3번으로 완전 대체 가능)
await store.send(
  .path(
    .element(
      id: 2,
      action: .topics(.nextButtonTapped)
    )
  )
)
// 2. keypath 사용
await store.send(\.path[id: 2].topics.nextButtonTapped)
// 3. associated value가 있을 때
await store.send(\.path[id: 2].topics, .nextButtonTapped(someValue))
```

CasePathable 매크로는 Action에 내포된 Delegate enum 까지 적용되지 않기 때문에 직접 넣어주어야 간편 문법 사용 가능.

PrintChanges 기능을 제대로 동작하게 하기 위해 TestStore에 적용했던 diff 관련 트윅을 비슷하게 적용하는 듯.
ChangeTracker를 클래스로 (deinit 사용) 바꾸어 스냅샷이 액션 처리가 완료될 때 nil로 초기화되도록 함.
자세하게 이해할 필요성을 느끼지 못함.

의문점:
stack의 원소를 거의 index로 접근하는 것처럼 보이는데, 원소의 순서가 바뀌어도 이 id는 유지되어야 하는가? (순서가 바뀌지 않을수도 있음)
테스트니까 이렇게 접근을 하는 것은 문제가 없는 것처럼 보이긴 하나, 같은 원소에 접근하고 액션을 보내기 위해 너무 반복적인 코드를 작성해야 하는듯. (환경이 바뀔 경우 테스트가 자꾸 깨질것 같음, keyPath를 저장해두고 사용하면 어느정도 해결될 수도)

후기:
Value 타입을 사용하기 위해 이렇게까지 고도화 시키는 것이 크게 의미가 있는지 모르겠다. Reference 타입 또한 Value semantics 처럼 Reference semantics를 갖고 있다고 생각하기 때문에(특히 Sendable이 나오고선 더욱 신뢰하기 쉬워짐), TCA 처럼 이미 다 만들어 두었을때 편리하게 사용하는 것 까진 좋지만, 별것 아닌 것에 내부 구현이 이렇게나 복잡해진 것에 대해 알면 알수록 이걸 위해 소스코드를 더 많이 컴파일 해야하는게 맞는지 의문이 든다.

@홍승현

@윤용운

- testStore 사용법
- casePath 매크로 
- local variable 활용법

이쯤 되면 Enum KeyPath 애플에서 공식적으로 만들어줬으면 좋겠어요… ㅋㅋ

매크로를 사용해서 CasePath랑 KeyPath를 브릿징해주는 로직을 전부 숨겨버림.
``` swift
await store.send(.path(.element(id: 2, action: ….))
await store.send(\.path[id: 2].topics….)
``` 

타입 기반으로 자동완성을 높게 평가하는 거 같음.

send 후행 클로저는 엑션으로 인해서 상태가 변했을 때만 비교하면 됨.
다른 엑션을 발생시킬 경우, receive 메소드를 활용해서 처리.

StackNavigation에서 처리할 수 없는 상황이라면 기존 구조를 유지하기 굉장히 버거워 보임.

TaskLocal 조금 더 자세하게 공부하면 좋을 거 같음.

deinit 시점을 활용하기 위해서 class를 사용했는데 ~Copyable을 활용해서 처리할 수도 있어보임.

``` swift
struct ChangeTracker: ~Copyable {
  var onDeinit: () -> Void
  var didChange = false

  deinit { self.onDeinit() }
}

``` 
