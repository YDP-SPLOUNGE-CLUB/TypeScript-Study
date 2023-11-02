## 기본 문법

자바스크립트에서 제공하는 데이터 타입을 지원한다.

```typescript
const a: number = 5;
const b: string = '5';
const c: boolean = true;
const d: undefined = undefined;
const e: null = null;
```

함수, 반환값, 또한 타입을 설정 가능하다.

```typescript
function add(x: number, y: number): number { return x + y }

const add2: (x: number, y: number) => number = (x, y) => x + y;
```

타입에 대한 별칭을 정할 수 있다.

```typescript
type Add = (x: number, y: number) => number;

const add: Add = (x, y) => x + y;
```

interface 타입 정의 방식 또한 존재

```typescript
interface Add {
   (x: number, y: number): number;
}

const add: Add = (x, y) => x + y;
```

리스트형 타입 정의

```typescript
// 일반 리스트 타입 정의
const arr: string[] = ['123', '234'];

// 제네릭형 리스트 타입 정의
const arr2: Array<number> = [123, 345];

// 튜플형 리스트 타입 정의
// 순서와 튜플의 리스트 length를 엄격하게 정의
const arr3: [number, number, string] = [123, 345, 'hello'];
```

정확한 원시값 타입을 정의 가능
```typescript
const t: true = true;
```

---

## 타입 추론

tsc --noEmit 에서는 타입을 추론해주는 기능도 제공한다.

```typescript
const a = '5';
```

타입 추론을 통해서 불필요한 타입 정의를 피할 수 있다.

example

**강의를 한 사람은 타입의 추론을 정확하게 정의가 된 상태에서는 따로 Type 을 건드는 것을 추천하지 않았다.**

```typescript
const a:string = '5';
```

타입 추론은 굉장히 불안정하다. 예시로

```typescript
const arr3 = [123, 345, 'hello'];

// 해당 타입은 const arr3: (number | string)[] 로 나오는데 튜플로 정하였을 떄의 타입 추론과 맞지 않는다.
// 이럴 경우에는 기존에 사용한 튜플로 타입을 정의하면 좋다. 

// 요약. 타입추론을 실패했을때 직접 타입을 정해주는것이 가장 안전할 수 있다.
```
