# Creational - Factory Method

---

>구체적으로 어떤 인스턴스를 만들지는 서브 클래스가 정한다.

- 객체를 생성하는 인터페이스는 미리 정의
- 인스턴스를 만들 클래스의 결정은 서브클래스 쪽에서 내리는 패턴
- 클래스의 인스턴스를 만드는 시점을 서브클래스로 미룬다.





















## Factory Method VS Abstract Factory

### 공통점

- 형태와 효과는 비슷
  - 구체적인 객체 생성 과정을 추상화한 인터페이스 제공

### 차이점

- 관점이 다르다
  - Factory Method 는 "팩토리를 구현하는 방법( inheritance )" 에 초점
  - Abstract Factory 는 "팩토리를 사용하는 방법 (composition)" 에 초점
- 목적이 다른다
  - Factory Method 은 구체적인 객체 생성 과정을 하위 또는 구체 클래스로 옮기는 것이 목적
  - Abstract Factory 는 관련있는 여러 객체를 구체적인 클래스에 의존하지 않고 만들 수 있게 해주는 것이 목적

















---

참고

