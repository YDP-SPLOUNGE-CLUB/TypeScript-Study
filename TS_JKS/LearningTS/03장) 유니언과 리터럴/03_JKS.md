# 3. 유니언과 리터럴

타입스크립트가 해당 값을 바탕으로 추론을 수행하는 두 가지 핵심 개념을 소개한다.

1. 유니언: 값에 허용된 타입을 두 개 이상의 가능한 타입으로 확장하는 것
2. 내로잉: 값에 허용된 타입이 하나 이상의 가능한 타입이 되지 않도록 좁히는 것

## 3.1 유니언 타입

유니언 타입으로 정의된 여러 타입 중 하나의 타입으로 된 값의 속성을 사용하려면 코드에서 값이 보다 구체적인 타입중 하나라는 것을
타입스크립트에게 알려야 한다. 이 과정을 내로잉 이라 한다.

## 3.2 내로잉

내로잉은 값이 정의 선언 혹은 이전에 유츄된 것보다 구체적인 타입임을 코드에서 유추하는 것이다.

타입을 좁히는 데 사용할 수 있는 논리적 검사를 타입 가드 라고 한다.

TS가 코드에서 타입을 좁히는 데 흔히 사용하는 타입 가드 두 가지를 살펴보자.

### 3.2.1 값 할당을 통한 내로잉

변수에 값을 직접 할당하면 타입스크립트는 변수의 타입을 할당된 값의 타입으로 좁힌다.

```typescript
let admiral: number | string;

admiral = "String";

admiral.toUpperCase(); // OK

admiral.toFixed(); // Error toFixed does not exist on type string
```

### 3.2.2 조건 검사를 통한 내로잉

```typescript
let scientist = Math.random() > 0.5 ? "Rosalin Franlin" : 51;

if (scientist === "Rosalin Franlin") {
    scientist.toUpperCase(); // OK
}

scientist.toUpperCase(); // Error does not exist on type string | number
```

### 3.2.3 typeof 검사를 통한 내로잉

---

## 3.3 리터럴 타입

리터럴 타입은 좀 더 구체적인 버전의 원시 타입이다.

```typescript
const philsopher = "Hypatia";
```

philsopher는 어떤 타입인가? 얼핏 보면 string 타입이라고 말할 수 있고 실제로도 string 타입이다.

philsopher 는 단지 string 타입이 아닌 Hypatia 라는 특별한 값이다.
philsopher 의 타입은기술적으로 더 구체적인 Hypatia 다.

원시 타입 값 중 어떤 것이 아닌 특정 원싯값으로 알려진 타입이 리터럴 타입이다.
원시 타입 string 은 존재할 수 있는 모든 가능한 문자열의 집합을 나타내지만 리터럴 타입인 Hypatia 는 하나의 문자열만 나타낸다.

### 3.3.1 리터럴 할당 가능성

앞서 number 와 string 과 같은 서로 다른 원시 타입이 서로 할당되지 못한다는 것을 보았다. 마찬가지로 0과 1처럼 동일한 원시 타입일지라도
서로 다른 리터럴 타입은 서로 할당할 수 없다.

## 3.4 엄격한 null 검사

리터럴로 좁혀진 유니언의 힘은 타입스크립트에서 엄격한 null 검사라 부르는 타입 시스템 영역인 잠재적으로 정의되지 않은
undefined 값 으로 작업할 때 두드러진다.

### 3.4.1 십억 달러의 실수

십억 달러의 실수는 다른 타입이 필요한 위치에서 null 값을 사용하도록 허용하는 많은 타입 시스템을 가리키는 업계 용어

타입스크립트 컴파일러 옵션에 strictNullChecks 는 엄격한 null 검사를 활성화할지 여부를 결정한다.

strickNullChecks 를 비활성하면 코드의 모든 타입에 null | undefined 를 추가해여 모든 변수가 null 또는 undefined 를 할당가능하다.

### 3.4.2 참 검사를 통한 내로잉

```typescript
if (true) {
    // ...
}
```

### 3.4.3 초기값이 없는 변수

타입스크립트는 값이 할당될 때까지 변수가 undefined 임을 이해할 만큼 충분히 영리하다.

값이 할당되기 전에 속성 중 하나에 접근하려는 것처럼 해당 변수를 사용하려고 시도하면 다음과 같은 오류 메세지가 나타난다.

```typescript
let math: string;

math?.length; // Error variable math is used before being assigned;

math = "Mark";
math.length; // OK
```

변수 타입에 undefined 가 포함되어 있는 경우에는 오류가 보고되지 않습니다. 변수 타입에 | undefined 를 추가하면 유효한 타입이기 때문에
사용 전에는 정의할 필요가 없음을 타입스크립트가 나타낸다.

이전 코드 스니펫에서 string | undefined 라고 선언하면 오류가 발생하지 않는다.

## 3.5 타입 별칭
