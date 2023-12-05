## map 만들기

map 함수를 Number 에 알맞는 함수로 만들어 보았다.

const b 는 반드시 number 의 값을 받고 number 값을 내보내도록 만들었기에

((v: T) => T): T[] 로 해결이 된다.

```typescript
interface Arr<T> {
  map(callback: (v: T) => T): T[];
}

const a: Arr<number> = [1, 2, 3];

const b = a.map((v) => v + 1); // [2, 3, 4]
```

하지만 v.toString() 같은 문자열로 바꾸게 되면 위 함수는 통하지 않는다.

number[] 이 아닌 string[] 를 내뱉기 떄문이다.

제네릭 함수로 추가로 작성해보면.

```typescript
interface Arr<T> {
  map<S>(callback: (v: T) => S): S[];
}

const a: Arr<number> = [1, 2, 3];

const b = a.map((v) => v + 1); // [2, 3, 4]
const c = a.map((v) => v.toString()); // ['2', '3', '4'];
const d = a.map((v) => v % 2 === 0); // [false, true, false];
```

만약에 index 매개변수가 필요하다면 map 함수에 추가해서 작성해야한다.

```typescript
interface Arr<T> {
  map<S>(callback: (v: T, i: number) => S): S[];
}

const a: Arr<number> = [1, 2, 3];

const b = a.map((v, i) => v + 1); // [2, 3, 4]
```

## filter 만들기

밑 함수의 b는 마찬가지로 T 자체만을 계속 내보내는 경우는 문제가 없다.

하지만 다른 타입을 내보내는 경우에는 그렇지 않다.

Arr<number | string> 의 경우에서 string 만을 필터링 해서 내보내고 싶은데
필터 typeof v === string 에서의 결과는 (number | string)[] 이다.

해당 문제를 해결하기 위해서는 커스텀 타입 가드를 통해서 타입 추론을 변경시켜줘야 한다.

```typescript
interface Arr<T> {
  filter(callback: (v: T) => boolean): T[];
}

const a: Arr<number> = [1, 2, 3];

const b = a.filter((v) => v % 2 === 0); // [2]

const c: Arr<number | string> = [1, '2', 3, '4', 5];

const d = c.filter((v) => typeof v === string); // ['2', '4'] string[] // 작동 안함
```

커스텀 타입 가드를 사용해서 v is S 라는 커스텀 타입 가드를 붙여보았다.

하지만 정상적으로 작동하지 않는다. 
v: T 라고 선언하였는데 v 는 S 다 라고 명시적으로 선언한 것이
타입스크립트가 받아들이지 못하였다.

```typescript
interface Arr<T> {
  filter<S>(callback: (v: T) => v is S): S[];
  // Error 
  // A type predicate's type must be assignable to its parameter's type.
  // Type 'S' is not assignable to type 'T'.
  // 'T' could be instantiated with an arbitrary type which could be unrelated to 'S'.

}

const a: Arr<number> = [1, 2, 3];

const d = c.filter((v) => typeof v === 'string');
```

해당 문제를 해결하기 위해서 S 는 T에 속한다는 것을 명시해줘야 한다.
또한 v 는 string | number 이기 떄문에 v 에 관한 벨류를 커스텀 타입가드로 명시해줘야 한다.

```typescript
interface Arr<T> {
  filter<S >(callback: (v: T) => v is S): S[];
}

const a: Arr<number | string> = [1, 2, 3, '4', '5'];

const d = c.filter((v): v is string => typeof v === 'string');
```

## 공변성, 반공변성

함수간에 서로 대입이 가능한가? 에 대한 구별법

예시 코드
```typescript
function a(x: string): number {
  return +x;
};

a('1'); // 1

type B = (x: string) => number | string;

const b: B = a; // OK
```

function a 는 number 를 반환값으로 설정했지만

type B 는 number | string 을 반환값으로 받고있다.

그런데 const b 는 a 라는 대입을 하였을 떄 정상적으로 작동한다.

리턴값은 더 넓은 타입으로 대입이 가능하다.

주의점이 있다. 매개변수애서는 이러한 상황이 역전이 된다.

```typescript
function a(x: string | number): number {
  return 0;
}

type B = (x: string) => number;
let b: B = a;
```
