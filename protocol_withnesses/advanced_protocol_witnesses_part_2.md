# Advanced Protocol Witnesses: Part 2

### 성현

### 승현

### 용운
환경을 조성하는 방식으로 제네릭과 클로저를 통해서 여러가지 조합을 만들고 이를 중요하게 생각하는거 같음.
Swift의 한계가 존재했었기 때문, 부분적으로 수정됐지만 아직도 존재하긴 함.

wrapping, unwrapping dance이 부분이 정확하게 이해가 안됨.
endo 예시가 우리가 실제로 사용하는 store 트리 구조의 가장 기초가 되는 개념처럼 보임.

예전에 확인했을 때는 Resuing 구조체가 너무 오버스러웠지만, 지금와서 보니 크게 차이나지 않아보임.
애당초에 reuseIdentifier를 여러번 호출하지 않을뿐더러 여러곳에서 필요하면 Dependency처럼 다루면됨.
