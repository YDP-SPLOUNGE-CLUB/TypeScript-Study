## never 타입과 느낌표 (non-null assertion)

타입스크립트의 never 타입은 다른 타입만큼 흔하게 사용되거나 피할 수 없는 것이 아니기 떄문에 충분히 논의되고 있지 않다.

조건부 타입 같은 고급 타입을 처리하거나 이해하기 어려운 오류 메세지를 읽을 떄만 나타내는 never 타입을 무시할 수 있다.

- never의 의미와 필요성

- never 애플리케이션 실 사용과 함정

### never 타입이란

TS 에서는 never 타입은 값의 공집합이다. never 타입은 empty 타입과 같다.
집합에 어떠한 값도 없기 때문에 never 타입은 any 타입의 값을 포함해 어떤 값도 가질 수 없다.

```typescript
declare const any: any;

// Error
const never: never = any;
```

### never 타입이 왜 필요한가.

숫자 체계에 아무것도 없는 양을 나타내는 0 처럼 문자 체계에도 불가능을 나타내는 타입이 필요하다.

불가능 이라는 단어 자체는 모호하다. TS 에서는 불가능을 아래와 같은 방법으로 나타내고 있다.

- 값을 포함할 수 없는 빈 타입
    - 제네릭과 함수에서 허용되지 않는 매개변수
    - 호환되지 않는 타입들의 교차 타입
    - 빈 합집합 (무의 합집합)

- 실행이 끝날 떄 호출자에게 제어를 반환하지 않는 함수의 반환 타입
    - Node process.exit
    - void 함수

- 절대로 도달할 수 없는 else 분기의 조건 타입

- 거부된 프로미스에서 처리된 값의 타입

```typescript
const p = Promise.reject('foo'); // const p: Promise<never>
```

## 유니언 교차 타입과 never 의 동작

숫자 0이 덧셈과 곱셈에 작용 방법과 유사하듯이 never 타입은 유니언 / 교차 타입에서 특별할 속성을 지닌다.

- 숫자에 0을 더하면 동일한 숫자가 나오는 것과 같이 never 타입은 유니언 타입에서 없어진다.
```typescript
type Res = never | string // string
```

숫자에 0을 곱하면 0이 나오는 것과 같이 never 타입은 교차 타입을 덮어쓴다.
```typescript
type Res = never & string // never
```

### never 타입은 어떻게 쓸까?

never 타입을 많이 사용하지 않을 수 있지만 아래와 같이 적절한 사용 사례가 있다.

허용할 수 없는 함수 매개변수에 제한을 가한다.

never 타입을 이용해서 다양한 사용 사례에 놓인 함수에 제안을 걸 수 있다.

switch, if-else 문의 모든 상황을 보장한다.

함수가 단 하나의 never 타입 인수만을 받을 수 있는 경우 해당 함수를 never 타입 이외의 값으로 호출할 수 없다.

```typescript
function fn(input: never) {}

// 오직 `never` 만 받는다.
declare let myNever: never
fn(myNever) // ✅

// 아무 값이나 전달하거나 아무 값도 전달하지 않으면 타입 에러 발생
fn() // ❌ 인자 'input'에 아무 값도 주어지지 않음
fn(1) // ❌ 'number' 타입은 'never' 타입에 할당할 수 없음
fn('foo') // ❌ 'string' 타입은 'never' 타입에 할당할 수 없음

// `any`도 통과할 수 없다.
declare let myAny: any
fn(myAny) // ❌ 'any' 타입은 'never' 타입에 할당할 수 없음
```

이런 함수를 이용하면 switch, if-else 문의 모든 상황을 보장할 수 있다. 이를 기본 케이스(default case)로 이용하면
남아있는 것은 never 타입이어야 하기 때문에 모든 상황에 대처하는 것을 보장할 수 있다.

```typescript
function unknownColor(x: never): never {
    throw new Error("unknown color");
}


type Color = 'red' | 'green' | 'blue'

function getColorName(c: Color): string {
    switch(c) {
        case 'red':
            return 'is red';
        case 'green':
            return 'is green';
        default:
            return unknownColor(c); // 'string' 타입은 'never' 타입에 할당할 수 없음
    }
}
```

### 타이핑을 부분적으로 허용하지 않는다.

VariantA 또는 VariantB 타입의 매개변수를 받는 함수가 있다고 하자. 그러나 사용자는 두 타입의 모든 속성을 모두 포함하는 하위 타입을 전달해서는 안된다.

매개변수에는 유니언 타입 VariantA | VariantB를 활용할 수 있다.
단, 타입스크립트의 타입 호환성은 구조적 서브 타이핑을 기반으로 하기 때문에
객체 리터럴을 전달하지 않는 한 파라미터의 타입보다 더 많은 속성을 가진 객체를 함수에 전달하는 것은 허용된다.

```typescript
type VariantA = {
    a: string,
}

type VariantB = {
    b: number,
}

declare function fn(arg: VariantA | VariantB): void


const input = {a: 'foo', b: 123 }
fn(input) // 타입스크립트 컴파일러는 아무런 문제도 지적하지 않지만, 우리의 목적에는 맞지 않는다.
```

never 를 사용하면 구조적 타이핑을 비활성화하고 사용자가 두 속성을 모두 포함하는 객체를 전달하지 못하도록 할 수 있다.

```typescript
type VariantA = {
    a: string
    b?: never
}

type VariantB = {
    b: number
    a?: never
}

declare function fn(arg: VariantA | VariantB): void


const input = {a: 'foo', b: 123 }
fn(input) // ❌ 속성 'a'의 타입은 호환되지 않는다.
```

### 의도하지 않은 API 사용을 방지한다.

데이터를 읽고 저장하는 Cache 인스턴스를 생성할 때.

```typescript
type Read = {}
type Write = {}
declare const toWrite: Write

declare class MyCache<T, R> {
  put(val: T): boolean;
  get(): R;
}

const cache = new MyCache<Write, Read>()
cache.put(toWrite) // ✅ 허용
```

get 메서드를 통해서만 데이터를 읽을 수 있는 읽기 전용 캐시를 만들때
put 메서드의 인수에 never를 전달해 어떤 값도 전달하지 않을 수 있다.

```typescript
declare class ReadOnlyCache<R> extends MyCache<never, R> {} // 이제 MyCache 내부의 타입 매개변수 `T`가 `never`가 된다.

const readonlyCache = new ReadOnlyCache<Read>()
readonlyCache.put(data) // ❌ 'Data' 타입의 인자는 'never' 타입의 매개변수에 할당될 수 없다.
```

### 이론적으로 도달할 수 없는 분기를 표현한다.

infer 를 사용해 조건부 타입 내에 추가 타입 변수를 생성할 경우 모든 infer 키워드에 대해 else 분기를 추가해야 한다.

```typescript
type A = 'foo';
type B = A extends infer C ? (
    C extends 'foo' ? true : false// 이 표현식 내에서 'C'는 'A'를 나타낸다.
) : never // 이 분기는 도달할 수 없지만, 생략도 할 수 없다.
```

**해당 내용은 infer 에 대한 내용을 이해 못해서 다음에 추가로 기록하기로**

### 유니언 타입에서 멤버 필터링

도달할 수 없는 분기를 나타내는 것 외에도 never 타입은 조건부 타입에서 원하지 않는 타입을 필터링할 수 있다.

앞서 말한 바와 같이, never 타입은 유니언 타입에서 자동으로 제거된다. 즉, never 타입은 유니언 타입에서는 쓸모가 없다.

특정 기준에 따라 유니언 타입에서 멤버를 선택하는 유틸리티 타입을 작성할 때, never 타입은 쓸모가 없기 때문에 else 분기에 배치하기에 완벽한 타입이다.

```typescript
type Foo = {
    name: 'foo'
    id: number
}

type Bar = {
    name: 'bar'
    id: number
}

type All = Foo | Bar

type ExtractTypeByName<T, G> = T extends {name: G} ? T : never

type ExtractedType = ExtractTypeByName<All, 'foo'> // 결과 타입은 Foo
```

다음과 같은 단계를 거친다.

1. 조건부 타입은 유니언 타입에 걸쳐 할당된다.
```typescript
type ExtractedType = ExtractTypeByName<All, Name>
// ⬇️
type ExtractedType = ExtractTypeByName<Foo | Bar, 'foo'>
// ⬇️
type ExtractedType = ExtractTypeByName<Foo, 'foo'> | ExtractTypeByName<Bar, 'foo'>
```

2. 구현을 대체하고 개별적으로 평가한다.

```typescript
type ExtractedType = Foo extends {name: 'foo'} ? Foo : never | Bar extends {name: 'foo'} ? Bar : never
// ⬇️
type ExtractedType = Foo | never
```

3. 유니언에서 never 를 제거한다.

```typescript
type ExtractedType = Foo | never
// ⬇️
type ExtractedType = Foo
```

### 매핑된 타입의 키 필터링

타입스크립트에서 타입은 불변이다. 객체 타입에서 한 속성을 제거하고 싶으면 기존 타입을 변환하고 필터링해 새로운 타입을 만들어야 한다.
매핑된 타입의 키를 never 로 조건부로 다시 매핑하면 해당 키가 필터링 된다.

```typescript
type Filter<Obj extends Object, ValueType> = {
    [Key in keyof Obj 
        as ValueType extends Obj[Key] ? Key : never]
        : Obj[Key]
}

interface Foo {
    name: string;
    id: number;
}

type Filtered = Filter<Foo, string>; // {name: string;}
```

### 제어 흐름 분석의 좁은 타입

함수 반환 값을 never로 설정하면 함수는 실행이 끝났을 때 제어권을 호출자에게 반환하지 않는다.
이를 활용하면 흐름 분석을 제어해서 타입을 좁힐 수 있다.

```typescript
function throwError(): never {
    throw new Error();
}

let foo: string | undefined;

if (!foo) {
    throwError();
}

foo; // string
```

또는 || 또는 ?? 연산자 다음에 throwError 를 호출한다.

```typescript
let foo: string | undefined;

const guaranteedFoo = foo ?? throwError(); // string
```

### 호환되지 않는 타입의 불가능한 교차 타입 표시

이는 never에 대한 실제 애플리케이션이라기보다는, 타입스크립트 언어의 동작/특성처럼 느껴질 수 있다.
그럼에도 불구하고 마주할 수 있는 일부 애매한 오류 메시지를 이해하는 것은 중요하다.

호환되지 않는 타입의 교차해서 never 타입을 얻을 수 있다.

```typescript
type Res = number & string // never

type Res = number & never // never
```

객체 타입을 교차할 때 개별 속성이 판별 속성(기본적으로 리터럴 타입 또는 리터럴 타입의 유니언 타입)으로 간주되는지
여부에 따라 전체 타입이 never로 축소될 수도 있고 그렇지 않을 수도 있다.

아래 예에서는 string과 number가 판별 속성이 아니기 때문에 name 속성만 never가 되었다.

```typescript
type Foo = {
    name: string,
    age: number
}

type Bar = {
    name: number,
    age: number
}

type Baz = Foo & Bar // {name: never, age: number}  
```

불리언이 판별 속성(true | false의 유니언 타입)이기 때문에 전체 타입 Baz가 never로 축소된다.

```typescript
type Foo = {
    name: boolean,
    age: number
}

type Bar = {
    name: number,
    age: number
}

type Baz = Foo & Bar // never
```

## never 타입은 어떻게 읽을까?

명시적으로 never를 사용하지 않은 코드에서 never 타입과 관련된 오류 메시지를 받아본 적이 있을 수 있다.

타입스크립트가 일반적으로 타입을 교차하기 때문이다. 타입 안정성을 유지하고 건전성을 보장하기 위해 암묵적으로 이 작업을 수행한다.

```typescript
type ReturnTypeByInputType = {
  int: number
  char: string
  bool: boolean
}

function getRandom<T extends 'char' | 'int' | 'bool'>(
        str: T
): ReturnTypeByInputType[T] {
   if (str === 'int') {
      // generate a random number
      return Math.floor(Math.random() * 10) // ❌ 'number' 타입은 'never'타입에 할당할 수 없다.
   } else if (str === 'char') {
      // generate a random char
      return String.fromCharCode(
              97 + Math.floor(Math.random() * 26) // ❌ 'string' 타입은 'never'타입에 할당할 수 없다.
      )
   } else {
      // generate a random boolean
      return Boolean(Math.round(Math.random())) // ❌ 'boolean' 타입은 'never'타입에 할당할 수 없다.
   }
}
```

위 함수는 전달하는 인수 타입에 따라 숫자, 문자열 또는 불리언을 반환한다.
인덱스 접근 ReturnTypeByInputType[T]을 사용하여 해당 반환 타입을 검색한다.

그러나 모든 반환 문에 대해 타입 오류(X 타입은 never타입에 할당할 수 없다.)가 발생한다.
이때 X는 분기에 따라 숫자, 문자열 또는 불리언이다.

❌가 표시된 곳은 타입스크립트가 우리 프로그램에서 문제가 발생할 수 있는 가능성을 좁혀주도록 하는 곳이다.
각 반환 값은 ReturnTypeByInputType[T](이는 런타임에 숫자, 문자열 또는 불리언 중 하나다.)에 할당할 수 있어야 한다.

리턴타입이 가능한 모든ReturnTypeByInputType[T]타입(number, string 그리고 boolean 교차 타입)에 할당이 가능해야 타입이 안정하다고 할 수 있다.
숫자, 문자열, 불리언의 교차 타입은 서로 호환되지 않으므로 never 타입이 되고, 이것이 오류 메시지에 never가 표시되는 이유이다.

```typescript
function f1(obj: { a: number, b: string }, key: 'a' | 'b') {
    obj[key] = 1;    // 'number' 타입은 'never'타입에 할당할 수 없다.
    obj[key] = 'x';  // 'string' 타입은 'never'타입에 할당할 수 없다.
}
```

obj[key]는 런타임에 key 값에 따라 문자열이나 숫자가 될 수 있다.
그러므로 타입스크립트는 안전을 위해 obj[key]에 쓰이는 모든 값은 string과 number 타입 모두와 호환되어야 한다는 제약 조건을 추가한다.
따라서 두 타입을 교차하게 되고, never 타입을 주게 된다.

### never 타입은 어떻게 검사할까?

타입이 never 인지 확인하는 것은 생각보다 어렵다.

```typescript
type IsNever<T> = T extends never ? true : false

type Res = IsNever<never> // never 🧐
```

타입스크립트는 조건부 타입에 대해 자동적으로 유니언 타입을 할당한다.

never은 빈 유니언 타입이다.

그러므로 할당이 발생하면 할당할 것이 없으므로 조건부 타입은 never로 평가된다.

유일한 해결 방안은 암묵적 할당을 막고 타입 매개변수를 튜플에 래핑하는 것이다.

```typescript
type IsNever<T> = [T] extends [never] ? true : false;
type Res1 = IsNever<never> // 'true' ✅
type Res2 = IsNever<number> // 'false' ✅
```