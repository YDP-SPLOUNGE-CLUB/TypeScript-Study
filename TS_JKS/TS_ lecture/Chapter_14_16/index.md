## readonly, 인덱스드 시그니처, 맵드 타입스

```typescript
interface A {
  readonly a: string;
  b: string;
}

const aaaa: A = { a: 'hello', b: 'world' };
aaaa.a = 's'; // Error
```

인덱스드 시그니쳐

```typescript
type A = { [key: string]: number };
const aaaa: A = {a: 1, b: 2, c: 3};
```

맵드 타입스

```typescript
type B = "Human" | "Mammal" | "Animal";
type A = { [key in B]: number};

const aaaa: A = { Human: 'Human', Animal: "Animal", Mammal: "Mammal"};
```

## 클래스

```typescript
class A {
  a: string;
  b: number;

  constructor(a: string, b: number = 123) {
    this.a = a;
    this.b = b;
  }
  
  method() {
    
  }
}

const a = new A('123', 456); // OK
const a2 = new A('123'); // OK

type AA = A;
// 클래스 이름은 그 자체로 타입이 될 수 있다.
// 여기서 A 는 new A(); 를 나타내고 있다.

// 클래스 자체의 타입은 typeof A
// 클래스 이름은 인스턴스 new A() 를 나타내는 것.
```

**클래스의 강의 내용은 protected, private, readonly, implements 같은 기능에 관한 내용**
**해당 내용은 많이 다루었기 떄문에 스킵**

## 제네릭

extendes 를 붙여서 제한을 붙이는 예시

```typescript
function add<T extends number | string>(x: T, y: T): T {
  return x + y;
};

add(1,2); // OK
add('1', '2'); // OK
```

```typescript
function add<T extends number, U extends string>(x: T, y: U): T {
  return x + y;
};

add(1,'2'); // OK
```

정말 제한이 없도록 하고 싶다면 이럴 때는 any 를 사용해도 무관하다.

```typescript
function add<T extends (...args: any) => any>(x: T): T {
  return x
};
```

클래스 생성자만 뽑아서 넣고 싶다면?

```typescript
function add<T extends abstract new (...args: any) => any>(x: T): T {
  return x
};

class A {}

add(A); // OK
add(new A()); // Error
```

제네릭에서 extends 기본값을 사용하면 좀 더 가독성을 좋게 표현 가능하다.

```typescript
// , 는 모든 타입을 허용한다는 뜻이지만 코드에서 그뜻을 유추하기 어렵다.
const add = <T,>(x: T, y: T) => ({x, y});

// 기본값을 타이핑 하여 좀 더 가독성 좋게 표현 가능하다.
const add2 = <T = unknown>(x: T, y: T) => ({x, y});
```