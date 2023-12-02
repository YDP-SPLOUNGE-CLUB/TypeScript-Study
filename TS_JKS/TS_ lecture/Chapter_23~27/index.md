## 타입스크립트는 건망증이 심하다

```typescript
interface Axios {
  get(): void;
};

interface CustomError extends Error {
  response?: {
    data: any;
  }
}

declare const axios: Axios;

(async () => {
  try {

  } catch (e: unknown) {
    console.log((e as CustomError)).response?.data;
    // 에러는 CustomError 라고 명시적 표현을 해주었지만 타입스크립트는 다음 줄에 바로 잊어버린다.
    e.response?.data; // Error
    
    // 변수에 저장해서 사용하면 위와 같은 건망증을 치료해줄수 있다.
    const customError = err as CustomError;
    
    // 하지만 as 는 any 만큼 나쁜 코드이기도 해서 조심해서 사용해라.
    // unknown 인 경우 as 를 사용해도 좋지만 가능하면 사용하지 말자.
    
    // 위 코드 또한 문재가 발생한다. CustomError 가 아니라면 어떻게 할것인가?
    
    if (e instanceof CustomError) {
      const customError2 = e;
    }
  }
})();
```

## utility Types

### Partial

```typescript
interface Profile {
  name: string;
  age: number;
  married: boolean;
}

const zerocho: Profile = {
  name: 'zerocho',
  age: 29,
  married: false,
}

// 만약 결혼 여부를 체크 안하고 싶다면 어떻게 해야할까?

interface NewProfile {
  name: string;
  age: number;
}

// married 를 지워서 반복되는 인터페이스를 작성할 수도 있겠지만
// 유틸리티 함수 Partial 를 사용하면 좋다.
// Profile 의 모든 내용물을 옵셔널로 사용할 수 있게 해주었다.

const newZeroCho: Partial<Profile> = {
  name: 'zerocho',
  age: 29,
}

// Partial 을 직접 만들어 보자.

type P<T> = {
  [Key in keyof T]?: T[Key];
}
```

## Pick

```typescript
interface Profile {
  name: string;
  age: number;
  married: boolean;
}

const zerocho: Profile = {
  name: 'zerocho',
  age: 29,
  married: false,
}

// Partial 은 옵셔널로 전부 바꾸기 떄문에 그렇게 추천하지 않는다.

// Pick 유틸리티 타입을 사용해서 원하는 속성만 가져올 수 있다.

const newZeroCho: Pick<Profile, 'name' | 'age'> = {
  name: 'zeroCho',
  age: 29,
}

// Pick 함수를 직접 만들어보자.

type P<T, S extends keyof T> = {
  [Key in S]: T[Key]
}

const newZeroCho2: P<Profile, 'name' | 'age'> = {
  name: 'zerocho',
  age: 29,
}
```

## Omit, Exclude, Extract 

Omit 을 설명하기 전에 Exclude 를 설명해야한다.

Exclude 는 타입에서 어느 특정 key 값을 제외한 값을 검색하는 타입 유틸리티이다.

```typescript
interface Profile {
  name: string;
  age: number;
  married: boolean;
}

type Animal = 'Cat' | 'Dog' | 'Human';
type Mammal = Exclude<Animal, 'Human'>;
// 'Cat' | 'Dog'

type A = Exclude<keyof Profile, 'married'>
// 'name', 'age'

type O<T, S> = Pick<T, Exclude<keyof T, S>>

// 그런데 위 코드에는 약간의 불안요소가 있다.
// 밑 코드같은 경우에도 정상적으로 작동하기 떄문이다.
const newZeroCho2: O<Profile, Profile> = {
  name: 'zeroCho',
  age: 29,
}

// 타입을 바로 넣지 못하게 제한을 시켜줘야 한다.
type O2<T, S extends keyof any> = Pick<T, Exclude<keyof T, S>>

const newZeroCho3: O2<Profile, "married"> = {
  name: 'zeroCho',
  age: 29,
}
```

Extract, Exclude 를 분석해보자.

조건부 타입과 never 의 조합으로 이루어진 타입이다.
never 를 통해서 타입을 날려버릴 것인지 아닐것인지 체크 해주는 코드를 확인 가능하다.

```typescript
// type Exclude<T, U> = T extends U ? never : T;
// type Extract<T, U> = T extends U ? T : never;
```

