## unknown 과 any

any 를 사용할 바에 unknown 을 사용하라.

any 는 타입검사의 포기선언과 같다.
unknown 은 알 수 없다는 뜻이다. 차라리 타입을 unknown 으로 지정한 다음 지정해서 사용하라

```typescript

interface A {
    talk: () => void;
}

const a: A = {
    talk() {return 3};
}

const b: unknown = a.talk();
(b as A).talk();
```

보통 에러같은 경우 unknown 으로 나오는데 
error 를 any 로 정의해서 사용하지 말고 unknown 을 다른 타입으로 지정해서 사용하면 좋다.

```typescript
try {
    
} catch (error) {
    (error as Error).message
}
```

unknown , any 같은 경우 전부 대입이 가능하다.

![179646513-3c3be896-3bbc-4784-848b-06bc47e8b129.png](..%2F..%2F..%2F..%2F..%2FDesktop%2F179646513-3c3be896-3bbc-4784-848b-06bc47e8b129.png)


## 타입 좁히기 (타입 가드)

> 개인적으로 타입스크립트의 가장 장점이면서 단점이라고 생각합니다.
> 타입스크립트의 코드의 정확성을 맞추기 위해서 코드량이 늘어나는 구간이기 떄문에 항상 의아하게 사용하고있습니다.

```typescript
function numOrStr1(a: number | string) {
    a.toFixed(); // Error string | number is not has toFixed
}

function numOrStr2(a: number | string) {
    if (typeof a === 'string') {
        a.toUpperCase(); // OK
    }
}

// 잘못된 사용법
function numOrStr3(a: number | string) {
    (a as number).toFixed(); // 작동은 하지만 정확하지 않음. 실수 가능성이 있다.
}

// 다른 예시
function numOrNumArray(a: number | number[]) {
    if (Array.isArray(a)) {
        a.concat(4);
    } else {
        a.toFixed(3);
    }
}
```

```typescript
type B = { type: 'b', bbb: string };
type C = { type: 'c', ccc: string };
type D = { type: 'd', ddd: string };

function typeCheck(a: B | C | D) {
    if (a.type === 'b') {
        // a 는 B 라는 타입검사와 함께 추론이 되므로 bbb의 타입을 가져옴.
        a.bbb
    } else if (a.type === 'c') {
        a.ccc
    } else {
        a.ddd
    }
}
```

> 실제로 위와 같이 사용하는 경우가 있을 수도 있지만.
> 개인적인 의견으로서는 저렇게 사용하는 것은 상당히 위험한 코드가 아닌가 싶기도 하다?

타입에 라벨을 붙이는 습관을 들이면 나중에 타입 검사를 할 때 빠르게 추론이 가능해진다.

```typescript
const human = { type: 'human' };
const dog = { type: 'dog' };
const cat = { type: 'cat' };
```

## 커스텀 타입 가드

```typescript
// 타입을 구분해주는 커스텀 함수를 직접 만들 수 있다.
interface Cat { meow: number };
interface Dog { bow: number };
function catOrDog(a: Cat | Dog): a is Dog {
    if ((a as Cat).meow) { return false }
    return true;
}

function pet(a: Cat | Dog) {
    if (catOrDog(a)) {
        console.log(a.bow);
    }
    if ('meow' in a) {
        console.log(a.meow);
    }
}
```

## as, is 

as 키워드는 **컴파일** 단계에서 타입 검사를 할 때 
타입스크립트가 감지하지 못하는 애매한 타입 요소들을 직접 명시해주는 키워드

```typescript
const myCanvas = document.getElementById('main') as HTMLCanvasElement;
```

as 키워드는 오직 컴파일 단계에서만 실행되며 런타임 단계에서는 삭제된 채로 실행된다.

```typescript
function isString(test: any): test is string {
    return typeof test === "string";
}

function example(foo: any) {
    if (isString(foo)) {
        console.log("it is a string" + foo);
    }
}
```

is 키워드는 타입 가드로 as 가 특정 변수 하나에 국한된 것이라면
is 키워드는 한정된 범위 내의 모든 변수에 대해서 일괄적으로 적용 가능한 키워드이다.

[참고내용](https://velog.io/@kwak1539/%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-is-as-%EB%AC%B8%EB%B2%95-%EC%A0%95%EB%A6%AC)

```typescript
const isRejected = (input: PromiseSettledResult<unknown>): input is PromiseRejectedResult => {
    return input.status === 'rejected';
}

const isSetteld = (input: PromiseSettledResult<T>): input is PromiseFulfilledResult<T> => {
    return input.status === 'fulfilled';
}

const promises = await Promise.allSettled([Promise.resolve('a'), Promise.resolve('b')]);
const errors = promises.filter(isRejected);
```