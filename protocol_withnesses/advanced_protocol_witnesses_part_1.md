# Advanced Protocol Witnesses: Part 1

### 성현

### 승현

### 용운
pullback -> scope

반공변성이라는 단어는 참 어려운듯
A가 B의 서브타입이라면, (B) -> Void가 (A)->Void의 서브타입이다.


``` swift
struct Predicate<A> {
  let contains: (A) -> Bool
  func contramap<B>(_ f: @escaping (B) -> A) -> Predicate<B> { // <- 반공변성 포인트
    return Predicate<B> { self.contains(f($0)) }
  }
}
```

꿀팁: 제네릭 타입을 확장할 때 var을 사용함으로써 피할 수 있다.

제네릭의 조합은 예전에도 느꼈지만 확장방향이 무한해보임.

반공변성에 대해서 다시 정리가 필요함.
