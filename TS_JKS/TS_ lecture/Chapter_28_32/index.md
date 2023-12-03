## Required, Record, NotNullable

만약 밑에 있는 Profile 같이 모든 속성이 옵셔널인 인터페이스에서
모든 속성을 필수로 바꾸기 위해서는 어떻게 해야할까?

```typescript
interface Profile {
  name?: string,
  age?: number,
  married?: boolean,
}

const zeroCho: Required<Profile> = {
  name: 'zero',
  age: 29,
  married: false,
}

// 직접 Requried 를 구현해보자.

type R<T> = {
  [Key in keyof T]-?: T[Key];
}

const zeroCho2: R<Profile> = {
  name: 'zero',
  age: 29,
  married: false,
}
```

Readonly 도 마찬가지로 모든 속성에 readonly 를 붙여서 만들거나
Readonly 에 - 를 붙여 없앨 수 있다.

```typescript
interface Profile {
  name?: string,
  age?: number,
  married?: boolean,
}

type R<T> = {
  -readonly [Key in keyof T]: T[Key];
}
```

Record 는 밑의 코드를 간단히 바꾼 코드이다.

```typescript
interface Obj {
  [key: string]: number;
};

// 동일
const a: Record<string, number> = {a: 3}

// Record 를 직접 만들어보자.

type R<T extends keyof any, S> = {
  [Key in T]: S;
}
```

NonNullable 은 Null 이나 undefined 를 제외하고 불러올 수 있다.

```typescript
type A = string | null | undefined | boolean | number;
type B = NonNullable<A>; // string boolean number;

// 직접 만들어보자

type N<T> = T extends null | undefined ? never: T;
```

## infer

```typescript
function zip(x: number, y: string, z: boolean): {
  x: number,
  y: string,
  z: boolean
} {
  return { x, y, z }; 
}

// zip 의 파라미터들에게 접근 가능하다.
type Params = Parameters<typeof zip>;

// 또한 Parmas 는 리스트형이기 떄문에 [0] x: number 에 접근 가능하다.
type First = Params[0];

```

Parameter 를 직접 만들어보자.

```typescript
function zip(x: number, y: string, z: boolean): {
  x: number,
  y: string,
  z: boolean
} {
  return { x, y, z }; 
}

// infer 란 추론이란 뜻이다.

// Paramemter 를 의미하는 P 함수는 infer A [parameter] 를 추론한다.
type P<T extends (...args: any) => any> = T extends (...args: infer A) => any ? A : never;

// ReturnType 를 의미하는 R 함수는 infer A [returns 출력값] 를 추론한다.
type R<T extends (...args: any) => any> = T extends (...args: any) => infer A ? A : never;

type Params = P<typeof zip>;
// 다음과 같다.
// [x: number, y: string, z: boolean]
type Returns = R<typeof zip>;
// 다음과 같다.
// type Returns = {
//   x: number;
//   y: string;
//   z: boolean;
// }
```

또 다른 예

```typescript
class A {
  a: string;
  b: number;
  c: boolean;

  constructor(a: string, b: number, c: boolean) {
    this.a = a;
    this.b = b;
    this.c = c;
  }
}

const c = new A('123', 345, true);

// ConstructorParameter 를 infer 로 추론하여 가져온다.
type C = ConstructorParameters<typeof A>;
// 다음과 같다. [a: string, b: number, c: boolean]

// Instance 자체를 추론해서 가져온다.
type I = InstanceType<typeof A>;
// 다음과 같다. A


type C2<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: infer P) => any ? P : never;

type I2<T extends abstract new (...args :any) => any> =
  T extends abstract new (...args: any) => infer I ? I : never;
```

## infer 심화

더 어려운 예제를 찾아왔다..

```typescript
const p1 = Promise.resolve(1).then((a) => a + 1).then((a) => a + 1).then((a) => a.toString());
const p2 = Promise.resolve(2);
const p3 = new Promise((res, rej) => {
  setTimeout(res, 1000);
});

Promise.all([p1, p2, p3]).then((result) => {
  console.log(result); // 타입 추론 [string, number, unknown]
});
```

all 에 대한 타입을 보면 다음과 같다.

```typescript
all<T extends readonly unknown[] | []>(values: T): Promise<{-readonly [P in keyof T]: Awaited<T[P]>}>;
```

해석 순서

1. Promise.all 안에 있는 객체들을 readonly 로 읽어들인다.
2. Promise 해석이 끝난 상태에서는 readonly 를 삭제시켜 자유로운 상태로 변환시켜 준다.
3. Awaited 라는 타입 내에서 then 의 값을 Return 시켜준다.

Awaited 라는 함수에서 어떤 일이 일어났는가?

```typescript
type Awaited<T> =
  T extends null | undefined ? T : // special case for `null | undefined` when not in `--strictNullChecks` mode
    T extends object & { then(onfulfilled: infer F): any } ? // `await` only unwraps object types with a callable `then`. Non-object types are not unwrapped
      F extends ((value: infer V, ...args: any) => any) ? // if the argument to `then` is callable, extracts the first argument
        Awaited<V> : // recursively unwrap the value
        never : // the argument to `then` was not callable
      T; // non-object or non-thenable
```

p1 을 예시로 순차적으로 해석해보면

1. T가 null 또는 undefined 인가? false 이므로 T 값을 내보내지 않고 다음 타입을 해석한다.
2. T extends object & { then(onfulfilled: infer F): any } <- 덕타이핑 을 통해서 then 이 있는 함수인지를 판별
3. F 가 true 라면 Awaited<V> 를 통해서 다시한번 재귀함수를 실행시키며
4. F 라는 함수가 extends 를 만족시키지 않는 함수라면 함수에 아무런 값을 내보내지 않는다.
5. T extends object & { then(onfulfilled: infer F): any } 가 아니라면 T 벨류 자체를 내보낸다.
6. p1 의 첫 값은 number.then(...) 이므로 then(onfulfilled: inter F): any 에 적합하므로
7. F extends ((value: infer V, ...args: any) => any) ? (a + 1) 를 읽어들이며 Awaited 재귀를 발생시킨다.
8. 재귀를 반복하며 마지막 v.toString() 까지 도달하였고 뒤에 추가적으로 붙는 then 이 없으므로
   T extends object & { then(onfulfilled: infer F): any } 조건에 불만족한다.
9. T 라는 값을 내보내며 p1 의 resolve 값은 string 이 된다.

## flat

```typescript
const a = [1,2,3, [1,2], [[1], [2]]].flat(); // (number| number[])[]
const a2 = [1,2,3, [1,2], [[1], [2]]].flat(2); // number[]
```

FlatArray 에 관한 타입은 다음과 같다.

```typescript
type FlatArray<Arr, Depth extends number> = {
  "done": Arr,
  "recur": Arr extends ReadonlyArray<infer InnerArr>
      ? FlatArray<InnerArr, [-1,0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20][Depth]>
      : Arr
}[Depth extends -1 ? "done": "recur"];
```

쉽게 설명하면 depth 가 끝났다면 done 을 호출하여 배열 그대로를 반환하며
아니라면 Arr 을 Depth 를 풀어줘서 재귀함수를 호출한다.

만약 flat(1) 이라면 -> 0 을 호출하고 flat(0) 을 호출하고 그 이후
-1 을 호출하여 "done" 을 반환하게 된다.

20차원의 리스트를 고려해줬지만 25차원 쯤 되면 타입스크립트가 이해를 못한다.

다시 돌아가서 

```typescript
// 해당 함수에서 infer InnerArr 는 리스트 안쪽에 있는 배열들이다.
// infer InnerArr = number, number[], number[][]

const a = [1,2,3, [1,2], [[1], [2]]].flat(2);
```
순차적으로 정리하면 다음과 같다.

1. FlatArray<(number[] | number[][] | number[][][]), 2>[] 상태에서 FlatArray 를 호출
FlatArray Depth 가 2 이므로 ReadOnlyArray number[] | number[][] | number[][][] 의 안쪽 배열을 읽어 한 차원 밑으로 내린다.
또한 FlayArray<InnerArray, -1,0,1,2,...[2] = 1 를 불러내서 재귀 함수를 실행 
2. FlatArray<(number | number[] | number[][]), 1>[]
반복
3. FlatArray<(number | number[]), 0>[]
반복
4. FlatArray<number, -1>[]
-1 이므로 Arr 자체를 반환
5. number []