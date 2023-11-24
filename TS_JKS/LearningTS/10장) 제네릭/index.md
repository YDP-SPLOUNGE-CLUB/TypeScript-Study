# 10. 제네릭

타입스크립트에서 함수와 같은 구조체는 제네릭 타입 구조체를 원하는 수만큼 선언할 수 있다. 제네릭 타입 매개변수는
제네릭 구조체의 각 사용법에 따라 타입이 결정된다. 이러한 타입 매개변수는 구조체의 각 인스턴스에서 서로 다른 일부 타입을 나타내기 위해
구조체의 타입으로 사용된다. 

이러한 타입 매개변수는 구조체의 각 인스턴스에서 서로 다른 일부 타입을 나타내기 위해 구조체의 타입으로 사용된다.

타입 매개변수는 구조체의 각 인스턴스에 대해 타입 인수라고 하는 서로 다른 타입을 함께 제공할 수 있지만 타입 인수가 제공된 인스턴스 내에서는 일관성을 유지한다.

타입 매개변수는 전형적으로 T 나 U 같은 단일 문자 이름 또는 Key Value 같은 파스칼 케이스 이름을 갖는다.

## 10.1 제네릭 함수

매개변수 괄호 바로 앞 홀화살괄호(<, >) 로 묶인 타입 매개변수에 별칭을 배치해 함수를 제네릭으로 만든다.

그러면 해당 타입 매개변수를 함수의 본문 내부의 매개변수 타입 애너테이션, 반환 타입 애너테이션, 타입 애너테이션에 사용할 수 있다.

```typescript
function identity<T>(input: T) {
  return input;
}

const numberic = identity("me"); // 타입 : me;
const stringy = identity(123); // 타입 : 123;
```

```typescript
const identity = <T>(input: T) => input;
identity(123); // 타입 :123
```

### 10.1.2 다중 함수 타입 매개변수

임의의 수의 타입 매개변수를 쉼표로 구분해 함수를 정의한다. 제네릭 함수의 각 호출은 각 타입 매개변수에 대한 자체 값 집합을 확인할 수 있다.

함수가 여러 개의 타입 매개변수를 선언하면 해당 함수에 대한 호출은 명시적으로 제네릭 타입을 모두 선언하지 않거나 모두 선언해야 한다.

```typescript
function makeTuple<First, Second>(fist: First, second: Second) {
  return [fist, second] as const;
}
```

## 10.2 제네릭 인터페이스

인터페이스도 제네릭으로 선언할 수 있다. 인터페이스는 함수와 유사한 제네릭 규칙을 따르며 인터페이스 이름 뒤 <과 > 사이에 선언된 임의의 수의 타입 매개변수를 갖는다.

```typescript
interface Box<T> {
  inside: T;
};

let stringBox: Box<string> = {
  inside: 'abc',
}

let numberBox: Box<number> = {
  inside: 1111,
}
```

재밌는 사실은 타입스크립트에서 내장 Array 메서드는 제네릭 인터페이스로 정의된다는 점이다.

### 10.2.1 유추된 제네릭 인터페이스 타입

제네릭 함수와 마찬가지로 제네릭 인터페이스의 타입 인수는 사용법에서 유추할 수 있다.

```typescript
interface LinkedNode<Value> {
  next?: LinkedNode<Value>;
  value: Value;
}

function getLast<Value>(node: LinkedNode<Value>): Value {
  return node.next ? getLast(node.next) : node.value;
}

// 유추된 Value 타입: Date
let lastDate = getLast({
  value: new Date("09-13-1993");
});
```

## 10.3 제네릭 클래스

인터페이스 처럼 클래스도 나중에 멤버에서 사용할 임의의 수의 타입 매개변수를 선언할 수 있다.
클래스의 각 인스턴스는 타입 매개변수로 각자 다른 타입 인수 집합을 가진다.

```typescript
class Secret<Key, Value> {
  key: Key;
  value: Value;

  constructor(key: Key, value: Value) {
    this.key = key;
    this.value = value;
  }
  
  getValue(key: Key): Value | undefined {
    return this.key === key ? this.value : undefined;
  }
}
```

### 10.3.1 명시적 제네릭 클래스 타입

제네릭 클래스 인스턴스화는 제네릭 함수를 호출하는 것과 동일한 타입 인수 유추 규칙을 따른다.

함수 생성자에 전달된 매개변수의 타입으로부터 타입 인수를 유추를 할 수 있다면 유추된 타입을 사용한다.

하지만 생성자에 전달된 인수에서 클래스 타입 인수를 유추할 수 없는 경우에는 타입 인수의 기본값은 unknown 이 된다.

```typescript
class CurriedCallback<Input> {
  #callback: (input: Input) => void;

  constructor(callback: (input: Input) => void) {
    this.#callback = (input: Input) => {
      console.log(input);
      callback(input);
    }
  }
  
  call(input: Input) {
    this.#callback(input);
  }
}

new CurriedCallback((input: string) => {
  console.log(input.length);
});

new CurriedCallback((input) => {
  console.log(input.length); // Error length does not exist on type unknown
})
```

### 10.3.3 제네릭 인터페이스 구현

제네릭 클래스는 모든 필요한 매개변수를 제공함으로써 제네릭 인터페이스를 구현한다.

```typescript
interface ActingCredit<Role> {
  role: Role;
};

class MoviePart implements ActingCredit<string> {
  role: string;
  speaking: boolean;

  constructor(role: string, speaking: boolean) {
    this.role = role;
    this.speaking = speaking;
  }
}

class IncorrectExtension implements ActingCredit<string> {
  role: boolean;
  // Error role in type IncorrectExtension
}
```

### 10.3.4 메서드 제네릭

클래스 메서드는 클래스 인스턴스와 별개로 자체 제네릭 타입을 선언 가능하다.
제네릭 메서드의 대한 각각의 호출은 각 타입 매개변수에 대해 다른 타입 인수를 갖는다.

```typescript
class CreatePairFactor<Key> {
  key: Key;

  constructor(key: Key) {
    this.key = key;
  }
  
  createPair<Value>(value: Value) {
    return { key: this.key, value };
  }
};

// 타입 : CreatePairFactor<string>
const factory = new CreatePairFactor("role");

// 타입 : { key: string, value: number }
const numberPair = factory.createPair(10);

// 타입 : { key: string, value: string }
const stringPair = factory.createPair(10);
```

### 10.3.5 정적 클래스 제네릭

클래스의 정적 멤버는 인스턴스 멤버와 구별되고 클래스의 특정 인스턴스와 연결되어 있지 않다. 클래스의 정적 멤버는 클래스 인스턴스에 접근할 수 없거나
타입 정보를 지정할 수 없다. 

정적 클래스 메서드는 타입 매개변수를 선언할 수 있지만 클래스에 선언된 어떤 타입 매개변수에도 접근할 수 없다.

```typescript
class BothLogger<OnInstance> {
  instanceLog(value: OnInstance) {
    console.log(value);
    return value;
  }
  
  static staticLog<OnStatic>(value: OnStatic) {
    let fromInstacne: OnStatic; // Error Static member cannot reference class type aruments
  }
}
```

## 10.4 제네릭 타입 별칭

타입 인수를 사용해 제네릭을 만드는 타입스크립트의 마지막 구조체는 타입 별칭이다. 각 타입 별칭에는 T 를 받는 Nullish 타입과 같은 임의의 수의 타입 매개변수가 주어진다.

```typescript
type Nullish<T> = T | null | undefined;
```

### 10.4.1 제네릭 판별된 유니언

타입스크립트의 내로잉을 아름답게 결합해주는 필자의 가장 좋아하는 패턴의 기능이다.

```typescript
type Result<Data> = FailureResult | SuccessfulResult<Data>;

interface FailureResult {
  error: Error;
  succeeded: false;
};

interface SuccessfulResult<Data> {
  data: Data;
  succeeded: true;
};

function handleresult(result: Result<string>) {
  if (result.succeeded) {
    console.log(result.data);
  } else if (!result.succeeded) {
    console.log(result.error)
  }
}
```

## 10.5 제네릭 제한자

타입스크립트는 제네릭 타입 매개변수의 동작을 수정하는 구문도 제공한다.

### 10.5.1 제네릭 기본값

지금까지 제네릭 타입이 타입 애너테이션에 사용되거나 extends 또는 implements 의 기본 클래스로 사용되는 경우 각 타입 매개변수에 대한 타입 인수를 제공해야 한다.

타입 매개변수 선언 뒤에 = 와 기본 타입을 배치해 타입 인수를 명시적으로 제공할 수 있다. 기본값은 타입 인수가 명시적으로 선언되지 않고 유추할 수 없는 모든
후속 타입에 사용된다.

```typescript
interface Quote<T = string> {
  value: T;
}

let explicit: Quote<number> = { value: 123 };
```

타입 매개변수는 동일한 선언 안의 앞선 타입 매개변수를 기본값으로 가질 수 있다. 각 타입 매개변수는 선언에 대한 새로운 타입을 도입하므로
해당 선언 이후의 타입 매개변수에 대한 기본값으로 이를 사용할 수 있다.

모든 기본 타입 매개변수는 기본 함수 매개변수철머 선언 목록의 제일 마지막에 와야 한다.

```typescript
// Error Required type parameters may not follow optional type parameters
function inTheMiddle<First, Second = boolean, Third = number, Fourth>() {}
```

## 10.6 제한된 제네릭 타입

기본적으로 제네릭 타입에는 클래스 인터페이스 원싯값 유니언 별칭 등 모든 타입을 제공할 수 있다.
그러나 일부 함수는 제한된 타입에서만 작동해야 한다.

타입스크립트는 타입 매개변수가 타입을 확장해야 한다고 선언할 수 있으며 별칭 타입에만 허용되는 작업이다. 타입 매개변수를 제한하는 구문은 매개변수 이름 뒤에 extends 키워드를 배치하고
그 뒤에 이를 제한할 타입을 배치한다.


제네릭 함수가 T 제네릭에 대한 length 를 가진 모든 타입을 받아들이도록 구현한 코드이다.
문자열 배열 length: number 를 가진 객체는 허용되지만 Date 와 같은 타입 형태에는 숫자형 length 멤버가 없으므로 타입 오류가 발생한다.

```typescript
interface WithLength {
  length: number;
}

function logWithLength<T extends WithLength>(input: T) {
  console.log(`Length: ${input.length}`);
  return input;
}

logWithLength('string'); // 타입 string
logWithLength([false, true]); // boolean[]
logWithLength({ length: 123 }); // {length: number}

logWithLength(new Date()); // Error Date is not assignable to parameter of type WithLength
```

### 10.6.1 keyof 와 제한된 타입 매개변수

extends 와 keyof 를 함께 사용하면 타입 매개변수의 키로 제한할 수 있다.
또한 제네릭 타입의 키를 걱정하는 유일한 방법이기도 하다.

```typescript
function get<T, Key extends keyof T>(container: T, key: Key) {
  return container[key];
}

const role = {
  favorite: "a",
  others: ["b", "c", "d"],
}

const favorite = get(role, 'favorite'); // 타입 string;
const others = get(role, 'others'); // 타입 string[];

const missing = get(role, 'extras'); // Error
```

keyof 가 없었더라면 제네릭 key 매개변수를 올바르게 입력할 방법이 없었을 것이다.

T만 제공되고 key 매개변수가 모든 keyof T 일 수 있는 경우라면 반환 타입은 Container 에 있는 모든 속성 값에 대한 유니언 타입이 된다.

이렇게 구체적이지 않은 함수 선언은 각 호출이 타입 인수를 통해 특정 key 를 가질 수 있음을 타입스크립트에 나타낼 수 없다.

```typescript
function get<T>(container: T, key: keyof T) {
  return container[key];
}

const role = {
  favorite: "a",
  others: ["b", "c", "d"],
}

const found = get(role, 'favorite'); // 타입 string | string[]
```

## 10.8 제네릭 올바르게 사용하기

타입스크립트를 처음 접한 개발자는 이따금 제네릭을 과도하게 사용해 읽기 혼란스럽고 지나치게 복잡한 코드를 만들기도 한다.

타입스크립트의 모법 사례는 필요할 때만 제네릭을 사용하고 제네릭을 사용할 때는 무엇을 위해 사용하느지 명확해야 한다.

### 10.8.1 제네릭 황금률

함수에 타입 매개변수가 필요한지 여부를 판단할 수 있는 간단하고 빠른 방법은 타입 매개변수가 최소 두 번 이상 사용되었는지 확인하는 것이다.

```typescript
function logIput<Input extends string>(input: Input) {
  console.log(`Hi`, input);
}
```

이런 함수의 Input 타입 매개변수를 선언하는 것은 별로 쓸모가 없다.

**이펙티브 타입스크립트** 는 제네릭 황금률을 설명하며 제네릭으로 작업할 때 유용하고 훌륭한 팁도 소개한다.

### 10.8.2 제네릭 명명 규칙

타입스크립틀를 포함한 많은 언어가 지키고 있는 타입 매개변수에 대한 표준 명명 규칙은

기본적으로 T 를 사용하고 후속 타입 매개변수가 존재하면 U, V 등을 사용하는 것이다.