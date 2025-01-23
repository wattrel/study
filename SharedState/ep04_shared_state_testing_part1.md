# Shared State: Testing, Part 1
일시: 1월 20일 오후 10시

@김성현

참조타입을 사용해서 공유 상태를 잘 처리했지만, 이제 테스트가 문제가 된다.
참조 타입은 값 복사본을 만들지 않기 때문에, 이전 상태와 새로운 상태를 참조의 복사본으로는 비교할 수가 없다.

snapshot 값을 둠으로서 테스트 시 같은 참조 값으로도 쉽게 비교할 수 있다.
테크닉:
TaskLocal을 사용해서 테스트 환경을 구분해서 스냅샷에 set할지와 스냅샷과 비교할지를 결정하도록 한다.

여전히 참조 타입의 이전값 기반의 변경은(예: increment) 테스트에 문제가 있다:
액션에서 한번 실행되고, 테스트 어썰션에서 한번 더 실행하니 문제가 되는 것이다.
이를 해결하기 위해 get 시에도 스냅샷을 반환하도록 분기를 처리한다.

+문제: 공유 상태만 변경되었을 때 테스트스토어가 상태변화가 있었다고 감지하지 못함
changeTracker를 두어 변경이 발생했을 때 실행되도록 하고 그걸 기반으로 변경됨을 TestStore가 알 수 있도록 수정.

+문제: 공유 상태가 전혀 바뀌지 않아 snapshot이 nil인 경우 처리 불가
equatable 구현에 snapshot이 nil일 때 기본값 처리를 해준다.

+문제: 테스트 환경이 아닐때 불필요한 리소스 사용
changeTracker를 옵셔널로 변경하여 트래킹 환경이 아닐때 snapshot 복사를 차단.

@홍승현

@윤용운
값 타입처럼 살기 - 참조 편

 참조는 복사할 수 없기 때문에 값이 변경하기 전과 후가 없습니다. 비교가 불가능하기 때문에 테스트가 불가능함.
(복사는 가능하다. 새로운 객체를 생성함으로써 가능해짐, 그리고 테스트의 경우, 모든 필드를 비교하면 되긴 함.)

(값타입을 사용할 경우, 비교적 안전하게 사용할 수 있는장치가 많음, 하지만 오히려 잘못사용하면 성능적으로 손해를 보게 된다는걸 잊지말아야함.)

모든 필드를 비교하더라도 문제가 발생 참조의 대한 동등성은 비교를 했지만, 피처의 상태에선 동등성이 고려되지 않기 때문 (정확하게 의미 전달이 안됨.)

Swift는 스레드 안전성과 로컬 범위 안정성을 모두 갖춘 방식으로 글로벌을 처리할 수 있는 도구를 제공하는데 바로 TaskLocals입니다.

 SharedLocals.isAsserting은 TestStore를 사용하는지에 대한 플래그값  후행 클로저를 실행시켰을 때, 원본 값이 변경되면 안되고 snapshot이 변경되어야함.

But we need an additional trick. We want to snapshot the current value before making the mutation, but also we only want to do it for the first mutation:
이해가 안감.
``` swift
set {
  if SharedLocals.isAsserting {
    self.snapshot = newValue
  } else {
    if self.snapshot == nil {
      self.snapshot = self.currentValue
    }
    self.currentValue = newValue
  }
}
``` 

이번 세션은 설명이 잘 이해가 안감. 다시 정리 필요.
