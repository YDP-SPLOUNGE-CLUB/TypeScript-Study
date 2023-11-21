# 06. 배열

자바스크립트 배열은 매우 유연하고 내부의 모든 타입의 값을 혼합해서 자장할 수 있다.

```typescript
const element = [true, null, undefined, 22];

element.push("even", ["more"]);
```

그러나 대부분의 개별 자바스크립트 배열은 하나의 특정 타입의 값만 가진다.

다른 타입의 값을 추가하게 되면 배열을 읽을 때 혼란을 줄 수 있으며 프로그램에 문제가 될 만한 오류가 발생할 수도 있다.

## 6.1 배열 타입

다른 변수 선언과 마찬가지로 배열을 저장하기 위한 변수는 초깃값이 필요하지 않다.
변수는 undefined 로 시작해서 나중에 배열 값을 받을 수 있다.

타입스크립트는 변수에 타입 애너테이션을 제공해 배열이 포함해야 하는 값의 타입을 알려주려고 한다.

### 6.1.1 배열과 함수 타입

배열 타입은 함수 타입에 무엇이 있는지를 구별하는 괄호가 필요한 구문 컨테이너의 예이다.

괄호는 애너테이션의 어느 부분이 함수 반환 부분이고 어느 부분이 배열 타입 묶음인지를 나타내기 위해 사용한다.

```typescript
// string 배열을 반환하는 함수
let creatrStrings: () => string[];

// 각각의 string을 반환하는 함수 배열
let createCreators: (() => string)[];
```

### 6.1.2 유니언 타입 배열

배열의 각 요소가 여러 선택 타입 중 하나일 수 있음을 나타내려면 유니언 타입을 사용한다.

유니언 타입으로 배열 타입을 사용할 때 애너테이션의 어느 부분이 배열의 콘텐츠이고 어느 부분이 유니언 타입 묶음인지를 나타내기 위해 괄호를 사용해야 할 수도 있다.

```typescript
// 타입은 string 또는 number 의 배열
let stringOrArrayOfNumber: string | number[];

// 타입은 각각 number 또는 string 요소와 배열
let arrayOfStringOrNumbes: (string | number)[]
```

## 6.3 스프레드와 나머지 매개변수

### 6.3.1 스프레드

... 스프레드 연산자를 사용해 배열을 결합한다. 타입스크립트는 입력된 배열 중 하나의 값이 결과 배열에 포함될 것임을 이해한다.

만약에 입력된 배열이 동일한 타입이라면 출력 배열도 동일한 타입이다. 서로 다른 타입의 두 배열을 함께 스프레드에 새 배열을 생성하면

새 벼열은 두 개의 원래 타입 중 어느 하나의 요소인 유니언 타입 배열로 이해된다.

```typescript
// string[]
const soldires = ["a", "b", "c"];

// number[]
const sodierAges = [1,2,3];

// (string | number)[]
const conjoined = [...soldires, ...sodierAges];
```

### 6.3.2 나머지 매개변수 스프레드

타입스크립트는 나머지 매개변수로 배열을 스프레드하는 자바스크립트 실행을 인식하고 이에 대해 타입 검사를 수행한다.

나머지 매개변수를 위한 인수로 사용되는 배열은 나머지 매개변수와 동일한 배열 타입을 가져야 한다.

## 6.4 튜플

자바스크립트 배열은 이론상 어떤 크기라도 될 수 있다. 하지만 떄로는 튜플이라고 하는 고정된 크기의 배열을 사용하는 것이 유용하다.

튜플 배열은 각 인덱스에 알려진 특정 타입을 가지며 배열의 모든 가능한 멤버를 갖는 유니언 타입보다 더 구체적이다. 투플 타입을 선언하는 구문은 배열 리터럴 처럼 보이지만

요소의 값 대신 타입을 적는다.

```typescript
let yearAndWarrior: [number, string];

yearAndWarrior = [1, "a"]; // OK

yearAndWarrior = [false, "1"]; // Error

yearAndWarrior = ["a", 1]; // Error

yearAndWarrior = [1]; // Error
```

### 6.4.1 튜플 할당 가능성

타입스크립트에서 튜플 타입은 가변 길이의 배열 타입보다 더 구체적으로 처리된다. 가변 길이의 배열 타입은 튜플 타입에 할당할 수 없다.

pairLoose 내부에 [boolean, number] 가 있는 것을 볼 수 있지만 타입스크립트는 더 일반적인 (boolean | number)[] 타입으로 유추한다.

```typescript
const pariLoose = [false, 123];
```

### 나머지 매개변수로서의 튜플

튜플은 구체적인 길이와 요소 타입 정보를 가지는 배열로 간주되므로 함수에 전달할 인수를 저장하는 데 특히 유용하다.

타입스크립트는 ... 나머지 매개변수로 전달된 튜플에 정확한 타입 검사를 제공할 수 있다.

```typescript
function looPar(name: string, value: number) {
    console.log(`${name} has ${value}`);
}

const pairArray = ["Amage", 1];

looPar(...pairArray); // Error

const pariTupleIncorrect: [string, number] = ["Amage", 1];

looPar(...pairArray); // OK
```

나머지 매개변수 튜플을 사용하고 싶다면 여러 번 함수를 호출하는 인수 목록을 배열에 저장해 함께 사용할 수 있다.

```typescript
function logTrio(name: string, value: [number, boolean]) {
    console.log(`${name} has ${value[0]} (${value[1]}`);
};

const trios: [string , [number, boolean][]] = [
    ["a", [1, true]],
    ["b", [2, false]],
    ["c", [3, true]],
]

trios.forEach(trio => logTrio(...trio)); // OK
```

### 6.4.2 튜플 추론

타입스크립트는 생성된 배열을 튜플이 아닌 가변 길이의 배열로 취급한다. 배열이 변수의 초깃값 또는 함수에 대한 반환값으로 사용되는 경우 고정된 크기의 튜플이 아니라
유연한 크기의 배열로 가정한다.

```typescript
function firstCharAndSize(input: string) {
    return [input[0], input.length];
}

// firstChar Type = string | number;
// size Type = string | number;
const [firstChar, size] = firstCharAndSize("abc");
```

타입스크립트에서는 값이 일반적인 배열 타입 대신 좀 더 구체적인 튜플 타입이어야 함을 다음 두 가지 방법으로 나타낸다. 명시적 튜플 타입과 const 어시션을 사용한 방법이다.

#### 명시적 튜플 타입

함수에 대한 반환 타입 애너테이션처럼 튜플 타입도 타입 애너테이션에 사용할 수 있다. 함수가 튜플 타입을 반환한다고선언되고 배열 리터럴을 반환한다면
해당 배열 리터럴은 일반적인 가변 길이의 배열 대신 튜플로 간주된다.

```typescript
function firstCharAndSizeExplicit(input: string): [string, number] {
    return [input[0], input.length];
};
```

#### const 어시션

명시적 타입 애너테이션에 튜플 타입을 입력하는 작업은 명시적 타입 애너테이션을 입력할 때와 동일한 이유로 고통스러울 수 있다. 즉 코드 변경에 따라 작성 및 수정이 필요한
구문을 추가해야 한다.

하지만 그 대안으로 타입스크립트는 값 뒤에 넣을 수 있는 const 어시션인 as const 연산자를 제공한다.

```typescript
const unionArray  = [1156, "a"]; // (string | number)[];

const readOnlyTuple = [11, "a"] as const; // readonly [1157, "a"]
```