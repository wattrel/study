#Protocol Withness: Part 1
### 성현

### 승현

### 용운
때때로 동일한 타입이 하나의 프로토콜을 여러 방식으로 준수하는 것이 완전히 타당하고 기술적으로도 올바른 경우가 있습니다.
``` swift
extension PostgresConnInfo: Describable {
  var description: String {
    return """
      PostgresConnInfo(
        database: "\(self.database)",
        hostname: "\(self.hostname)",
        password: "\(self.password)",
        port: "\(self.port)",
        user: "\(self.user)"
      )
      """
  }
}

extension PostgresConnInfo: Describable {
  var describe: String {
    return """
      postgres://\(self.user):\(self.password)@\
      \(self.hostname):\(self.port)/\(self.database)
      """
  }
}
```

프로토콜이란 특정 타입이 수행할 수 있는 동작을 추상적으로 설명하는 방법으로, 함수 시그니처와 프로퍼티들의 모음을 지정하는 것

Swift에서 프로토콜을 정의하고, 특정 타입이 이를 준수하도록 만들 때, 컴파일러는 내부적으로 아주 특별한 작업을 수행하여 두 요소 간의 관계를 추적합니다. 이 과정이 어떻게 이루어지는지 정의하고 Swift 코드로 직접 재구현할 것입니다.

원래 컴파일러가 자동으로 처리해주는 작업을 우리가 직접 수행하는 것이지만, 직접 제어함으로써 훨씬 더 유연하고 조합 가능한 구조로 만들 수 있습니다 -> 제네릭 구조체
