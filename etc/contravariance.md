# Contravariance
### 성현

### 승현

### 용운
서브타이핑의 변성은 아저씨들도 표현하기 어려워함 :)

왜 반변성에 대해서 그렇게 알고 싶었던걸까?
-> 함수형 프로그래밍에서는(Store.Scope) 어떻게 사용하는지 파악하고 싶었던 거 같음.

이론은 간단하다.
A가 B의 서브타입이라면
() -> A는 () -> B의 서브타입이며 - 공변성
(B) -> Void는 (A) -> Void의 서브타입이다. - 반변성
 
  
![Image](https://github.com/user-attachments/assets/7c09fa4f-1df5-4d33-a3c6-3965b8ec4b22)

함수형 프로그래밍 관점에서 살펴보게 되면
입력값을 변환하고 싶은 경우에는 반변성이 필요하며 출력값을 변환하고 싶다면 공변성을 활용해야한다.
흔하게 사용하는 배열의 map은 공변성이다.
``` swift
map(_ f: (Element) -> B) -> [B]
```

그렇다면 왜 직관적이지 않는 반변성을 활용해야하는걸까? 로직을 재활용할 수 있기 때문이다.
아저씨들도 가장 큰 강점이라고 생각하는 거 같다. (Compositional) 
``` swift
struct Predicate<A> {
  let matches: (A) -> Bool
  
  func contramap<B>(_ f: @escaping (B) -> A) -> Predicate<B> {
    .init { b in
      let a = f(b)
      return self.matches(a)
    }
  }
}

let isLongString = Predicate<String> { $0.count > 5 }
let isLargeNumber: Predicate<Int> = isLongString
  .contramap { String($0) }

isLongString.matches("ASDF")

isLargeNumber.matches(123334)
isLargeNumber.matches(3)
```

서브타이핑의 변성에 대해서 궁금증이 어느정도 해결된거 같아서 아주 행복합니다 :)
