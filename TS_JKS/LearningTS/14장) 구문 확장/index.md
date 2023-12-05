# 14장. 구문 확장

타입스크립트 같은 상위 집합 언어에 특정 새로운 런타임 기능으로 자바스크립트 구문을 확장하는 방식은
다음과 같은 이유로 인해 나쁜 사례로 간주한다.

- 런타임 구문 확장이 최신 버전 자바스크립트의 새로운 구문과 충돌할 수 있다.
- 언어를 처음 접하는 프로그래머가 자바스크립트가 끝나는 곳과 다른 언어가 시작하는 곳을 이해하기 어렵게 만든다.
- 상위 집합 언어 코드를 사용하고 자바스크립트를 내보내는 트랜스파일러의 복잡성을 증가시킨다.


## 14.1 클래스 매개변수 속성

자바스크립트 클래스에서는 생성자에서 매개변수를 받고 즉시 클래스 속성을 할당하는 것이 일반적이다.

```typescript
class Enginner {
  readonly area: string;

  constructor(area: string) {
    this.area = area;
  }
}

new Enginner("mechancial").area;
```

타입스크립트는 이러한 종류의 매개변수 속성을 선언하기 위한 단축 구문을 제공한다. 속성은 클래스 생성자의 시작 부분에
동일한 타입의 멤버 속성으로 할당된다.

생성자의 매개변수 앞에 readonly 또는 public, protected, private 제한자 중 하나를 배치하면 타입스크립트가 동일한 이름과
타입의 속성도 선언하도록 지시한다.

```typescript
class Enginner {
  constructor(readonly area: string) {
    
  }
}
```

매개변수 속성은 클래스 생성자의 맨 처음에 할당된다.
매개변수 속성은 다른 매개변수 또는 클래스 속성과 혼합될 수 있다.

```typescript
class NamedEnginner {
  fullName: string;

  constructor(
    name: string,
    public area: sring,
  ) {
    this.fullName  `${name}, ${area} enginner`
  }
}
```

## 14.2 실험적인 데코레이터

클래스를 포함하는 많은 다른 언어에서는 클래스와 클래스의 멤버를 수정하기 위한 일종의 런타임 로직을
주석을 달거나 데코레이팅할 수 있다.

ECMAScript 에서는 아직 데코레이터를 승인하지 않았다.

타입스크립트는 데코레이터를 실험적으로 사용할 수 있도록 **experimentalDecorators** 컴퍼일러 옵션을 제공한다.

## 14.3 열거형

자바스크립트는 열거형 구문을 포함하지 않으므로 열거형을 배치해야 하는 곳에 일반적인 객체를 사용한다.

```typescript
const StatusCode = {
  InternalServerError: 500,
  NotFound: 400,
  OK: 200,
  ...
} as const;

StatusCode.InternalServerError; // 500
```

타입스크립트에서 열거형 같은 객체를 사용할 때 까다로운 점은
값이 해당 객체의 값 중 하나여야 함을 나타내는 훌륭한 타입 시스템 방법이 없다는 것이다.

keyof 나 typeof 타입 제한자를 함께 함께 사용해 하나의 값을 해킹하는 방법도 있다.
하지만 많은 양의 구문을 입력해야 한다.

타입스크립트는 타입이 number 또는 string 인 리터럴 값들을 갖는 객체를 생성하기 위한 enum 구문을 제공한다.

컴파일된 자바스크립트에서 열거형은 이에 사응하는 객체로 컴파일 된다. 열거형의 각 멤버는 해당 값을 갖는 객체 멤버 키가 되고
그 반대의 경우도 마찬가지이다.

### 14.3.3 const 열거형

열거형은 런타임 객체를 생성하므로 리터럴 값 유니언을 사용하는 일반적인 전략보다 더 많은 코드를 생성한다.

타입스크립트는 const 제한자로 열거형을 선언해 컴파일된 자바스크립트 코드에서 객체 정의와 속성 조희를 생략하도록 지시한다.

```typescript
const enum DisplayHint {
  Opaque = 0,
  Semitransparent,
  Transparent,
}

let displayHint = DisplayHint.Transparent;
```

컴파일된 자바스크립트 코드에는 열거형 선언이 모두 누락되고 열거형의 값에 대한 주석을 사용한다.