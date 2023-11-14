# 4. 객체

3장에서는 boolean 같은 원시타입과 true 같은 리터럴 값으로 작동되는 유니언과 리터럴 타입을 설명하였다.

이러한 원시 타입은 자바스크립트 코드가 일바적으로 사용하는 복잡한 객체의 겉만 흝었고 타입스크립트가 이러한 객체를 표현할 수 없다면 사용이 불가능 했을 것이다.

이번 장에서는 박잡한 객체 형태를 설명하는 방법과 타입스크립트가 객체의 할당 가능성을 확인하는 방법을 설명하곘다.

## 4.1 객체 타입

{...} 구문을 사용해서 객체 리터럴을 생성하면 타입스크립튼느 해당 속성을 기반으로 새롱누 객체 타입 또는 타입 형태를 고려한다.

객체 타입은 객체의 값과 동일한 속성명과 원시 타입을 갖는다.

```typescript
const post = {
    born: 1935,
    name: "Mary Oliver"
};

post['born']; // number
post.name; // string
```

객체 타입은 타입스크립트가 자바스크립트 코드를 이해하는 방법에 대한 핵심 개념이다.

### 4.1.1 객체 타입 선언

기존 객체에서 직접 타입을 유추하는 방법도 굉장히 좋지만 타입을 명시적으로 선언해야한다.
객체 타입은 객체 리터럴과 유사하게 보이지만 필드 값 대신 타입을 사용해 설명 가능하다.

```typescript
let poetLater: {
    born: number;
    name: string;
}

poetLater = {
    born: 1935,
    name: 'name'
}

poetLater = "1"; // Error
```

## 4.2 구조적 타이핑

타입스크립트의 타입 시스템은 구조적으로 타입화되어 있다. 타입을 충족하는 모든 값을 해당 타입의 값으로 사용할 수 있다.
매개변수나 변수가 특정 객체 타입으로 선언되면 타입스크립트에 어떤 객체를 사용하든 해당 속성이 있어야 한다고 말해야 한다.

```typescript
type WithFirstName = {
    firstName: string;
};

type WithLastName = {
    lastName: string;
};

const hasBoth = {
    firstName: "1",
    lastName: '2',
};

// OK hasBoth 는 WithFirstName 의 firstName 을 포함
let withFirstName: WithFirstName = hasBoth;

```

구조적 타이핑은 덕 타이핑 과는 다르다. 덕 타이핑은 오리처럼 보이고 오리처럼 꽥꽥거리면, 오리일 것이다. 라는 문구에서 유래했다.

- 타입스크립트의 타입 검사기에서 구조적 타이핑은 정적 시스템이 타입을 검사하는 경우이다.
- 덕 타이핑은 런타임에서 사용될 때까지 객체 타입을 검사하지 않는 것을 말한다.

자바스크립트는 **덕 타입** 이지만 타입스크립트는 **구조적 타입화**를 한다.

### 4.2.1 사용 검사

객체 타입으로 에너테이션된 위치에 값을 제공할 때 타입스크립트는 값을 해당 객체 타입에 할당할 수 있는지 확인한다.
할당하는 값에는 객체 타입의 필수 속성이 있어야 한다. 객체 타입에 필요한 멤버가 객체에 없다면 타입스크립트는 타입 오류를 발생시킨다.

```typescript
type FirstAndLastNames = {
    first: string;
    last: string;
};

// OK
const hasBoth: FirstAndLastNames = {
    first: "first",
    last: "last"
};

// Error Property last is missingin type
const hasOnlyOne: FirstAndLastNames = {
    first: "first"
};
```

### 4.2.4 선택적 속성

모든 객체에 객체 타입 속성이 필요한 건 아니다. 타입의 속성 에너테이션에서 : 앞에 ? 를 추가하면 선택적 속성임을 나타낼 수 있다.

```typescript
type Book = {
    author?: string;
    pages: number;
}

const ok: Book = {
    pages: 1
}
```

## 4.3 객체 타입 유니언

타입스크립트 코드에서는 속성이 조금 다른, 하나 이상의 서로 다른 객체 타입이 될 수 있는 타입을 설명할 수 있어야 한다. 또한 속성값을 기반으로 해당 객체 타입 간에 타입을 좁혀야 할 수도 있다.

### 4.3.1 유츄된 객체 타입 유니언

변수에 여러 객체 타입 중 하나가 될 수 있는 초깃값이 주어지면 타입스클비트는 해당 타입을 객체 타입 유니언으로 유추한다.
유니언 타입은 가능한 각 객체 타입을 구성하고 있는 요소를 모두 가질 수 있다.

객체 타입에 정의된 각각의 가능한 속성은 비록 초깃값이 없는 선택적 타입이지만 각 객체 타입의 구성 요소로 주어진다.

```typescript
const poem = Math.random() > 0.5
    ? { name: "a", pages: 7 }
    : { name: "b", rhymes: true }

// 타입
// {
//     name: string;
//     pages: number;
//     rhymes?: undefined;
// }
// |
//     {
//         name: string;
//         pages?: undefined;
//         rhymes: boolean
//     }
```

### 4.3.2 명시된 객체 타입 유니언

객체 타입의 조합을 명시하면 객체 타입을 더 명확히 정의할 수 있다.
코드를 조금 더 작성해야 하지만 객체 타입을 더 많이 제어할 수 있다는 이점이 있다.
값의 타입이 객체 타입으로 구성된 유니언이라면 타입 시스템은 이런 모든 유니언 타입에 존재하는 속성에 접근을 허용한다.

```typescript
type PoemWithPages = {
    name: string;
    pages: number;
}

type PoemWithRhymes = {
    name: string;
    rhymes: boolean;
}

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem = Math.random() > 0.5
    ? { name: "a", pages: 7 }
    : { name: "b", rhymes: true }

poem.name; // OK

poem.pages; // Error
poem.rhymes; // Error
```

잠재적으로 존재하지 않는 객체의 멤버에 대한 접근을 제한하면 코드의 안전을 지킬 수 있다.

### 4.3.4 판별된 유니언

자바스크립트와 타입스크립트에서 유니언 타입으로 된 객체의 또 다른 인기 있는 형태는 객체의 속성이 객체의 형태를 나타내도록 하는 것이다.

이러한 타입 형태를 판별된 유니언 이라 부르고 객체의 타입을 가리키는 속성이 판별값이다. 타입스크립트는 코드에서 판별 속성을 사용해 타입 내로잉을 수행한다.

```typescript
type PoemWithPages = {
    name: string;
    pages: number;
    type: 'pages';
}

type PoemWithRhymes = {
    name: string;
    rhymes: boolean;
    type: 'rhymes';
}

type Poem = PoemWithPages | PoemWithRhymes;

const poem: Poem = Math.random() > 0.5
    ? { name: "a", pages: 7, type: "pages" }
    : { name: "b", rhymes: true, type: "rhymes" }

if (poem.type === "pages") {
    console.log(`It's got Pages ${poem.pages}`)
} else {
    console.log(`It's got Pages ${poem.rhymes}`)
}
```

## 4.4 교차 타입

타입스크립트 유니언 타입은 둘 이상의 다른 타입 중 하나의 타입이 될 수 있음을 나타낸다.
자바스크립트의 런타임 | 연산자가 & 연산자에 대응하는 역할을 하는 것처럼 타입스크립트에서도 교차 타입을 사용해 여러 타입을 동시에 나타낸다.


### never

교차 타입은 잘못 사용하기 쉽고 불가능한 타입을 생성한다. 원시 타입의 값은 동시에 여러 타입이 될 수 없기 때문에 교차 타입의 구성 요소로 함께 결합할 수 없다.

두 개의 원시 타입을 함께 사용하면 never 키워드로 표시되는 never 타입이 된다.