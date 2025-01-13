# Shared State: The Problem
일시: 1월 13일 오후 10시

@김성현 
# Shared State: The Problem

### Value Type vs Reference Type
- Value Type 선호 이유:
  - 단순하고 로직이 없는 데이터 구조
  - 컴파일러가 강제하는 안전한 mutation
  - 명확한 동등성(equality) 비교
  - 테스트하기 쉬움
  - 복사 가능한 특성으로 인해 철저한 테스트 가능

- Reference Type의 문제점:
  - 동등성 비교가 복잡함
  - 네트워크 요청 등 동작을 포함할 때 동등성 판단이 어려움
  - 보통 같은 참조인지 여부로만 동등성 확인 가능

### 공유 상태를 처리하기 위한 시도들

**1. 값이 변경될 때를 감지해서 하위로 복사**
- onChange를 사용하여 한 탭의 상태 변경을 다른 탭으로 동기화
- 부모 상태의 변경을 자식들에게 전파

구현 시 단점:
- 많은 동기화 코드 필요
- 실수하기 쉬움
- 새로운 Observation 도구들과 충돌할 수 있음
- 복잡한 앱에서는 권장되지 않음

테스트 시 단점:
- 여러 복사본의 상태를 모두 검증해야 함
- 테스트 코드가 복잡해짐
- 상태 변경을 모든 복사본에 대해 명시적으로 검증 필요

**2. 하위 상태를 Computed Property로 선언하여 공유 상태를 조립하도록 하기**
- 부모의 상태를 computed property를 통해 자식들에게 전달
- 동기화 로직 제거 가능

구현 시 단점:
- 모든 하위 상태 정보를 부모가 가지고 있어야 함

**3. Dependency를 활용**
- StatsClient와 같은 의존성을 통해 상태 공유
- 각 기능이 자신의 동기화 로직만 책임짐
- 사실상 reference type을 도입한 것과 같음
  - Closure를 포함한 구조체는 reference type의 crude한 근사치
  - 앱 어디서나 상태 업데이트 가능
  - 모든 부분에서 즉시 변경사항 확인 가능

구현 시 단점:
- 보일러플레이트 코드가 많이 필요
- View에서 직접 접근 불가
- 상태 변경 구독을 위한 추가 작업 필요
- stream 엔드포인트 구현 필요

테스트 시 단점:
- 의존성 오버라이드 필요
- onAppear 등 추가 액션 처리 필요
- 테스트 코드가 복잡해짐



@홍승현
여러 독립적이고 모듈화된 기능 간에 동일한 상태를 어떻게 공유할 수 있는가에 대한 문제
- 애플리케이션에서 참조 타입과 값 타입의 진짜 의미에대해

이번 시리즈를 통해 아래의 두가지 목표를 이룰 것
1. 공유 상태는 완벽하게 테스트될 수 있다.
    * 심지어 철저하게 테스트 할 수 있다.
2. 상태를 자동으로 유지하는 데 필요한 도구를 제공할 것
    * 상태의 모든 변경사항은 자동으로 외부에 저장되어 다음에 애플리케이션을 시작할 때 사용할 수 있다

- 값 타입을 선호하는 주된 이유는 간단하고 논리가 없는 데이터의 덩어리로 즉시 이해할 수 있기 때문에 선호한다.
그러나
- 값 타입은 “상태 공유” 라는 개념과 잘 어울리지 않는다.
- 결국 값을 전달할 때 값은 복사되고, 이 카피는 원래 값과 전혀 관련이 없어진다.
- 복사본에 발생한 변형은 원본에 아무런 영향을 미치지 않는다.
- 이를 해결하기 위해 늘어나는 여러 동기화 로직
    - 정확한 동작을 하기 어려워지는 부분 중 하나
- TCA는 상태를 모델링하기 위해 값 타입을 사용하려고 노력하고 있지만, 그렇게 하려면 잘못되기 쉬운 온갖 종류의 동기화 로직을 구현해야한다.

참조 타입의 경우 equality와 얽히는 문제개 매우 복잡하다.
- 동작이 포함되어있기 때문에


Dependency를 활용하면,

어떤 의미에서는 이미 참조 타입을 기능안에 도입함
사실상 구조체 + 클로저는 참조타입을 흉내낸 것

Dependency는 매우 “참조”성격이 강하다.

장점: 전체 앱의 어느 곳에서나 상태를 업데이트 할 수 있고 앱의 다른 모든 부분에서도 이러한 변경사항을 즉시 확인할 수 있다.
단점: Dependency가 state에서 작동하려면 많은 설정이 필요하다.

애초에 State에 그냥 클래스 같은 참조 타입을 넣으면, 상태 공유 문제를 해결할 수 있지 않을까?




@윤용운 
이 아저씨들도 TCA에서 장점으로 내세운 single source of truth에 대해서 명확한 답을 내릴 수 없다고 말하심
1. 상태가 자동으로 공유되는것인가(참조에 의해서)
2. 루트 저장소의 상태 중 하나를 변경하면 모든 뷰에서 해당 상태를 즉시 볼 수 있는건지(동기화 로직)

상황에 따라서 다르게 사용되지만 진정한 문제는 앱 구조에서 독립적이거나 분리된 모듈에서 사용되는 공용 상태값을 말합니다.
Swift’s observation machinery가 가능하게 해줌(아마도 Macro 부분을 말하는 거 같음)

Shared 라이브러리의 경우 테스터블하며, 상태를 자동으로 유지하는 데 필요한 도구를 제공할 예정.
UserDefaults, FileManager 등등

값 타입은 현재 컨텍스트에서 벗어날 때, 복사가 발생하기 때문에 여러 스레드에서 안전하게 전달이 가능하다.
그렇기 때문에 원론적으로 접근하면 값 타입을 shared state로 사용할 수 없습니다. 기존의 경우 shared state로 사용하기 위해서 여러 가지 동기화 메커니즘을 사용했습니다.
- Reducer.onChange + State의 computed property 등등
(onChange에 클로저를 등록할 경우, 상태가 복잡했을 때 부하가 없을까?)
(자식과 부모의 관계를 모두 Shared State로 처리하면 안됩니다. (혼동하면 안됨.))

공유 상태의 경우 모든 중간 계층을 통해 전달할 필요 없이 애플리케이션 깊숙이 종속성을 전파하는게 중요.
의존성 자체를 상태값으로 사용할 수 있게 만들수도 있었음. 결국 로컬 변수를 어떻게 활용하는지에 따라서 동시성 활용 능력이 달라지는 거 같음.
(객체를 바라볼 때, 의미론적으로 접근하면 다르게 사용할 수 있는 방법이 보이는 거 같음.)
@dynamicMemberLookup
struct StatsClient: Sendable {
  var get: @Sendable () -> Stats
  var set: @Sendable (Stats) -> Void
  var stream: @Sendable () -> AsyncStream<Stats>

  func modify(_ operation: (inout Stats) -> Void) { // -> transaction
    var stats = self.get()
    defer { self.set(stats) }
    operation(&stats)
  }

  subscript<Value>(dynamicMember keyPath: KeyPath<Stats, Value>) -> Value {
    self.get()[keyPath: keyPath]
  }
}

extension StatusClient: DependencyKey {
    static var liveValue: StatsClient {
      //var stats = Stats() <- Sendable closure에서 캡처할 수 없음. 이를 회피하기 위해서 Locking System을 활용해야함.
      let stas = LockIsolated(Stats()) <- Value must be Sendable
      let subject = PassthroughSubject<Stats, Never>() 

      return StatsClient(
        get: { stats.value },
        set: { newValue in
            stats.withValue { 
               $0 = Value
               subject.send(newValue)
            },
        stream: { subject.values.eraseToStream() } /// <- AsyncStream 재활용하는 방향은 여러 에러를 발생시킴 그렇기 때문에 중간에 combine을 래핑해서 사용했음. 모든 걸 Swift Concurrency를 사용하려고 하지말고 적제적소에 다른 장치를 사용해서 Wrapping하는 방식만 선택하자. 
      )
    }
}

Reducer내부에서 문제가 발생하지 않지만 View에서 사용하는 Store의 경우 State에 대한 인터페이스를 노출하기 때문에 Environment를 접근할 수 없다. 그렇기 때문에 역시 아직은 Environment를 동기화 시켜주는 코드가 필요.

Tips
- 프로퍼티를 생성할 때 type을 따로 명시하는 것보다 값을 할당하는게 좋다는 생각을 하게 됨.
var currentTab = Tab.counter <-
var currentTab: Tab = .counter

- 상태를 만들 때, 의미있는 단위로 묶는게 중요.
상태를 직접 mutate할 때도 있지만 단위를 사용할 때 동작을 만드는것도 좋아보임.
struct Stats: Equatable {
  private(set) var count = 0
  private(set) var maxCount = 0
  private(set) var minCount = 0
  private(set) var numberOfCounts = 0
  mutating func increment() {
    count += 1
    numberOfCounts += 1
    maxCount = max(maxCount, count)
  }
  mutating func decrement() {
    count -= 1
    numberOfCounts += 1
    minCount = min(minCount, count)
  }
  mutating func reset() {
    self = Self()
  }
}
logic-less bags of data that are immediately understandable
Value types can only be mutated in very tight lexical scopes.
또한 복사가 가능하기 때문에 이전 값과 이후값을 비교할수 있기 때문에 철저한 테스트에도 적합하다.
참조 타입의 경우 동등성 비교에서 보다 복잡한 문제를 갖고 있다. 왜나하면 행동을 갖고 있기 때문
행동으로 표현할 수 있는건 네트워크 요청과 같은 Task를 들고 있을 때, Task를 어떻게 비교할 것인지에 대한 내용. 실행을 시켰는지 혹은 레퍼런스가 같은지 와 같은 내용.
그렇기 때문에 참조 타입의 경우 참조 동일성을 비교하는 경우가 대부분

- 자식 리듀서에서 상위로 이벤트를 전달해야할 때는 Delegate 네임스페이스를 사용해서 분리하는게 좋아보입니다.
enum ChildAction {
    enum Delegate { caes update(String) }
}

case .tapButton
return .send(.delegate(.update(...))
