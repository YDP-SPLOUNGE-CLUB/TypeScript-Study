## 15.1 매핑된 타입

타입스크립트는 다른 타입의 속성르 기반으로 새로운 타입을 생성하는 구문을 제공한다.

```typescript
// example
type NewType = {
  [K in OriginalType]: NewProperty;
}
```

매핑된타입에 대한 일반적인 사용 사례는 유니언 타입을 혼합하여 상용하여 각 문자열 리터럴 키를 가진 객체를 생성하는 것이다.

```typescript
type Animals = "alligator" | "baboon" | "cat";

type AnimalCounts = {
  [K in Animals]: number;
};
```

### 15.1.1 타입에서 매핑된 타입

일반적으로 매핑된 타입은 존재하는 타입에 keyof 연산자를 사용해 키를 가져오는 방식으로 작동한다.

존재하는 타입의 키를 매핑하도록 탕비에 지시하면 새로운 타입으로 매핑된다.

```typescript
interface AnimalVariants {
  aliigator: boolean;
  baboon: number;
  cat: string;
}

type AnimalCounts = {
  [K in keyof AnimalVariants]: number;
}
```

이전 스니펫에서 K 로 명명된 keyof 에 매핑된 새로운 타입 키는 원래 타입의 키로 알려져 있다.
즉 각 매핑된 타입 멤버 값은 동일한 키 아래에서 원래 타입의 해당 멤버 값을 참조할 수 있다.

원본 객체가 SomeName 이고 매핑이 [K in keyof SomeName]인 경우라면 매핑된 타입의 각 멤버는
SomeName 멤버이 값을 SomeName[K] 로 참조할 수 있다.

```typescript
interface BirdVariants {
  dove: string;
  eagle: boolean;
}

type NullableBirdVarinats = {
  [K in keyof BirdVariants]: BirdVariants[K] | null;
}
```

#### 매핑된 타입과 시그니쳐

인터페이스 멤버를 함수로 선언하는 두 가지 방법을 제시했었다.

- member(): void 구문 : 인터페이스 멤버가 객체의 멤버로 호출되도록 의도된 함수임을 선언
- member: () => void 구문 : 인터페이스의 멤버가 독립 실행형 함수와 같다고 선언

매핑된 타입은 객체 타입의 메서드와 속성 구문을 구분하지 않는다. 매핑된 타입은 메서드를 원래 타입의 속성으로 취급

```typescript
interface Researcher {
  researchMethod(): void;
  researchProperty: () => string;
}

type JustProperties<T> = {
  [K in typeof T]: T[K]
};

type ResearcherProperties = JustProperties<Researcher>;

// 다음과 같다.
// {
//   researchMethod: () => void;
//   researchProperty: () => string;
// }
```

### 15.1.2 제한자 변경

매핑된 타입은 원래 탕비의 멤버에 대해 접근 제어 제한자인 readonly 와 ? 도 변경 가능하다.

전형적인 인터페이스와 동일한 구문을 사용해 매핑된 타입의 멤버에 readonly 와 ? 를 배치 가능하다.

```typescript
interface Enviromentalist{
  area: string;
  name: string;
}

type ReadOnlyEnviromentalist = {
  readonly [K in keyof Enviromentalist]: Enviromentalist[K];
};

// 다음과 같음
// {
//   readonly area: string;
//   readonly name: string;
// }

type OptionalReadOnlyEnviromentalist = {
  [K in keyof ReadOnlyEnviromentalist]?: Enviromentalist[K];
};
```

새로운 타입 제한자 앞에 - 를 추가해 제한자를 제거 가능하다.

```typescript
interface Conservationist {
  name: string;
  catchphrase?: string;
  readonly born: number;
  readonly died?: number;
}

type WrtableConservationist = {
  -readonly [K in keyof Conservationist]: Conservationist[K];
};

// 다음과 같음.
// {
//   name: string;
//   catchprase?: string;
//   born: number;
//   died?: number;
// }

type RequireWritableConservationist = {
  [K in keyof WrtableConservationist]-?: WrtableConservationist[K];
};
```

### 15.1.3 제네릭 매핑된 타입

매핑된 타입의 완전한 힘은 제네릭과 결합해 단일 타입의 매핑을 다른 타입에서 재사용할 수 있도록 하는 것에서 나온다.

매핑된 타입은 매핑된 타입 자체의 타입 매개변수를 포함해 keyof 로 해당 스코프에 있는 모든 타입 이름에 접근할 수 있다.

제네릭 매핑된 탕비은 데이터가 애플리케이션을 통해 흐를 때 데이터가 어떻게 변형되는지 나타낼 때 유용하다.

```typescript
type MakeReadonly<T> = {
  readonly [K in keyof T]: T[K];
}

interface Species {
  genus: string;
  name: string;
}

type ReadonlySpecies = MakeReadonly<Species>;
```

개발자들이 일반적으로 표현해야 하는 또 다른 변환은 임의의 수의 인터페이스를 받고 그 인터페이스의
완전히 채워진 인스턴스를 반환하는 함수이다.

```typescript
interface GenusData {
  family: string;
  name: string;
}

type MakeOptinal<T> = {
  [K in keyof T]?: T[K];
}

function createGenusData(overrides?: MakeOptinal<GenusData>): GenusData {
  return {
    family: 'unknown',
    name: 'unknown',
    ...overrides,
  }
}
```

## 15.2 조건부 타입

기존 타입을 다른 타입에 매핑하는 것은 훌륭하지만 아직 타입 시스템에 논리적 조건이 추가되지 않았다.

타입스크립트의 타입 시스템은 논리 프로그래밍 언어의 한 예이다.

타입 시스템은 이전 타입에 대한 논리적인 검사를 바탕으로 새로운 구성을 생성한다. 조건부 타입의 개념은

기존 타입을 바탕으로 두 가지 가능한 타입 중 하나로 확인되는 타입이다.

조건부 타입 구문은 삼항 연산자 조건문처럼 보인다.

```typescript
LeftType extends RightType ? IfTrue : IfFalse;
```

조건부 타입에서 논리적 검사는 항상 extends 의 왼쪽 타입이 오른쪽 타입이 되는지 또는 할당 가능한지 여부에 있다.

다음 CheckStringAgainstNumber 조건부 타입은 string | number 가 되는지 여부를 검사한다.

string 타입을 number 타입에 할당할 수 있는지 여부이다. 할당할 수 없다면 그 결과 타입은 false 가 된다.

```typescript
type CheckStringAginstNumber = string extends number ? true : false;
```

### 15.2.1 제네릭 조건부 타입

조건부 타입은 조건부 타입 자체의 타입 매개변수를 포함한 해당 스코프에서 모든 타입 이름을 확인할 수 있다.

즉 모든 다른 타입을 기반으로 새로운 타입을 생성하기 위해 재사용 가능한 제네릭 타입을 작성할 수 있다.

```typescript
type CheckAgaubstNumber<T> = T extends number ? true : false;

// 타입 : false
type CheckString = CheckAgaubstNumber<'parakeet'>;

// 타입 : true
type CheckString = CheckAgaubstNumber<1111>;

// 타입 : true
type CheckString = CheckAgaubstNumber<number>;
```

```typescript
interface QuertOptions {
  thorwIfNotFound: boolean;
}

type QueryResult<Options extends QuertOptions> = 
  Options["thorwIfNotFound"] extends true ? string : string | undefined;

declare function retrieve<Options extends QuertOptions>(
  key: string,
  options?: Options,
): Promise<QueryResult<Options>>;

// 반환 타입 string | undefined;
await retrieve("Mary");

// string | undefined;
await retrieve("Jane", { thorwIfNotFound: Math.random() > 0.5 });

// string
await retrieve("Dian", { thorwIfNotFound: true });
```

## 15.2.2 타입 분산

```typescript
type ArrayifUnlessString<T> = T extends string ? T: T[];

// string | number[]
type HalfArrayified = ArrayifUnlessString<string | number>;
```

타입스크립트의 조건부 타입이 유니언에 분산되지 않는다면 string | number 는 string 에 할당할 수 없기 떄문에
HalfArrayified 는 (string | number)[] 가 된다.

### 15.2.3 유츄된 타입

제공된 타입의 멤버에 접근하는 것은 타입의 멤버로 저장된 정보에 대해서는 잘 작동하지만
함수 매개변수 또는 반환 타입과 같은 다른 정보에 대해서는 알 수 없다.

조건부 타입은 extends 절에 infer 키워드를 사용해 조건의 임의의 부분에 접근한다.

extends 절에 타입에 대한 infer 키워드와 새 이름을 배치하면 조건부 타입이 true 인 경우
새로운 타입을 사용할 수 있음을 의미한다.

infer 이란 제네릭 파라미터에 들어간 타입을 뽑아내 다른 변수에 넣어두어 타입을 선언할때 사용하기도한다.

```typescript
type ArrayItems<T> = 
  T extends (infer item)[]
    ? item
    : T;

// string
type StringItem = ArrayItems<string>;

// string
type StringArrayItem = ArrayItems<string[]>;

// string[]
type String2DItem = ArrayItems<string[][]>;
```

타입 매개변수 T를 받고 새로운 Item 타입의 배열인지 확인한다.
새로운 Item 타입의 배열인 경우 결과 타입은 Item 이 되고 그렇지 않으면 T 가 된다.

```typescript
// infer 와 같은 위치에 있는 string 타입을 R 변수에 선언한다.
type ArrType<T> = T extends (infer R)[] ? R : unknown;

type arr = ArrType<string[]>;	// string 타입이다.
type arr2 = ArrType<string>;	// unknown 타입이다.
```

### 15.2.4 매핑된 조건부 타입

매핑된 타입은 기존 타입의 모든 멤버에 변경 사항을 적용하고 조건부 타입은 하나의 기존 타입에 변경 사항을 적용한다.

```typescript
type MakeAllMembersfunctions<T> = {
  [K in keyof T]: T[K] extends (...args: any[]) => any
      ? T[K]
      : () => T[K]
};

type MemberFunctions = MakeAllMembersfunctions<{
  alreadyFunction: () => string,
  notYetFunction: number;
}>

//
// {
//   alreadyfunction: () => string;
//   notYetFUnction: () => number;
// }
```

## 15.4 템플릿 리터럴 타입

문자열값을 입력하기 위한 두 가지 전략을 제시한 적이 있다.

원시 string 타입 : 값이 세상의 모든 문자열이 될 수 있는 경우
"" 와 "abc" 같은 리터럴 타입: 값이 오직 한 가지 타입만 될 수 있는 경우

문자열 타입이 패턴에 맞는지를 나타내는 타입스크립트 구문인 템플릿 리터럴 타입을 입력해보자

```typescript
type Greeting = `Hello${string}`;

let matches: Greeting = "Hello, world!"; // OK

let outOfOrder: Greeting = "World! Hello"; // Error
```

```typescript
type Brightness = "dark" | "light";
type Color = "blue" | "red";

type BrightnessAndColor = `${Brightness}-${Color}`;

let colorOk: BrightnessAndColor = "dark-blue";
```

### 15.4.1 고유 문자열 조작 타입

문자열 타입 작업을 지원하기 위해 타입스크립트는 문자열을가져와 문자열에 일부 조작을 적용하는 고유 제네릭 유틸리티 타입을 제공한다.

- Uppercase
- Lowercase
- Capitalize
- Uncapitalize

```typescript
type FormalGreeting = Capitalize<"hello,">; // Hello,
```

```typescript
const config = {
  location: 'unknown',
  name: "annoymous",
  year: 0,
}

type LayValues = {
  [K in keyof typeof config as `${K}Lazy`]: () => Promise<typeof config[K]>;
};

async function withLazyValues(configGetter: LayValues) {
  await configGetter.locationLazy; // string
  
  await configGetter.missingLazy; // Error Propery missingLazy does not exist
}
```

자바스크립트에서 객체 키는 string 또는 Symbol 이 될 수 있고 Symbol 키는 원시 타입이 아니므로
템플릿 리터럴 타입으로 사용할 수 없다.

```typescript
type TurnIntoGettersDirect<T> = {
  [K in keyof T as `get${K}`]: () => T[K]
  // Error
}
```

이러한 제한 사항을 피하기 위해 string 과 교차 타입 & 을 사용하여 문자열이 될 수 있는 타입만 사용하도록 강제한다.

string & symbol 은 never 가 되므로 전체 템플릿 문자열은 never가 되므로 전체 템플릿 문자열은 never 가 되고
타입스크립트는 이를 무시한다.

```typescript
const someSymbol = Symbol("");

interface HasStringAndSymbol {
  Stringkey: string;
  [someSymbol]: number;
}

type TurnIntoGetters<T> = {
  [K in keyof T as `get${string & K}`]: () => T[K]
};

type GettersJustString = TurnIntoGetters<HasStringAndSymbol>;

// 다음과 같음.
// {
//   getStringkey: () => string;
// }
```