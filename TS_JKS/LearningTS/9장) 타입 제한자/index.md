# 09. 타입 제한자

## 9.1 top 타입

top 타입은 시스템에서 가능한 모든 값을 나타내는 타입이다. 모든 다른 타입의 값은 타입이 top 인 위치에 제공될 수 있다.

### 9.1.1 any 다시 보기

any 타입은 모든 타입의 위치에 제공될 수 있다는 점에서 top 타입처럼 작동할 수 있다.

다만 any는 타입스크립트가 해당 값에 대한 할당 가능성 또는 멤버에 대해 타입 검사를 수행하지 않도록 명시적으로 지시한다는 문제점을 갖는다.

이러한 안정성 부족은 타입스크립트의 타입 검사기를 빠르게 건너뛰려고 할 떄 유용하지만 타입 검사를 비활성화 하면 해당 값에 대한 타입스크립트의 유용성이 떨어진다.

### 9.1.2 unknown

타입스크립트에서 unknown 타입은 진정한 top 타입이다. 모든 객체를 unknown 타입의 위치로 전달할 수 있다는 점에서 any 타입과 유사하다.
unknown 타입과 any 타입의 값을 훨씬 더 제한적으로 취급한다는 점이다.

- 타입스크립트는 unknown 타입 값의 속성에 직접 접근 가능하다.
- unknown 타입은 top 타입이 아닌 타입에는 할당할 수 없다.

```typescript
function greetComedian(name: unknown) {
    console.log(`Annoucing ${name.toUpperCase()}!`);
    // Error Object is of type 'unknown'
}
```

타입스크립트가 unknown 타입인 name에 접근할 수 있는 유일한 방법은 instanceof 나 typeof 또는 타입 어시션을 사용하는 것처럼 값의 타입이 제한된 경우이다.

```typescript
function greetComedianSafety(name: unknown) {
    if (typeof value === 'string') {
        console.log(`Announcing ${name.toUpperCase()}!`);
    } else {
        console.log('no')
    }
}
```

## 9.2 타입 서술어

instanceof, typeof 같은 자바스크립트 구문을 사용해 타입을 좁히는 방법을 살펴보았는데
제한된 검사로 이 방법을 직접 사용할 때는 괜찮지만 로직을 함수로 감싸면 타입을 좁힐 수 없게 된다.

```typescript
function isNumberOrString(value: unknown) {
    return ['number', 'string'].includes(typeof value);
};

function logValueIfExists(value: number | string | null | undefined) {
    if (isNumberOrString(value)) {
        value.toString();
        // Error Object is possibly undefined
    } else {
        console.log("value does not exist :", value);
    }
}
```

타입스크립트에는 인수가 특정 타입인지 여부를 나타내기 위해 boolean 값을 반환하는 함수를 위한 특별한 구문이 있다.
타입 서술어 라고 부르며 사용자 정의 타입 가드 라고도 부른다.

타입 서술어는 이미 한 인터페이스의 인스턴스로 알려진 객체가 더 구체적인 인터페이스의 인스턴스인지 여부를 검사하는 데 자주 사용된다.

```typescript
interface Comedian {
    funny: boolean;
}

interface StandupComedian extends Comedian {
    routine: string;
}

function isStandupComedian(value: Comedian): value is StandupComedian {
    return 'routine' in value;
}
```

타입 서술어는 false 조건에서 타입을 좁히기 떄문에 타입 서술어가 입력된 타입 이상을 검사하는 경우 예상치 못한 결과를 얻을 수 있다.


## 9.3 타입 연산자

키워드나 기존 타입의 이름만 사용해 모든 타입을 나타낼 수는 없다. 떄로는 기존 타입의 속성 일부를 변환해서 두 타입을 결합하는 새로운 타입을 생성해야 할 때도 있다.

### 9.3.1 keyof

자바스크립트 객체는 일반적으로 string 타입인 동적값을 사용하여 검색된 멤버를 갖는다. 타입 시스템에서 이러한 키를 표현하려면 상당히 까다로울 수 있다.

string 같은 포괄적인 원시 타입을 사용하면 컨테이너 값에 대해 유요하지 않은 키가 허용된다.

```typescript
interface Ratings {
    audience: number;
    critics: number;
}

function getRating(ratings: Ratings, key: string):number {
    return ratings[key];
    // Error Element implicitly has an any type becuase expression
}

const ratings: Ratings = { audience: 66, critics: 84 };

getRating(ratings, 'audiecne'); // OK

getRating(ratings, 'not valid'); // 허용되지만 사용해서는 안된다.
```

그러나 인터페이스에 수십 개 이상의 멤버가 있다면 상당히 번거로운 작업이 필요하다.

대신 타입스크립트에서는 기존에 존재하는 타입을 사용하고 해당 타입에 허용되는 모든 키의 조합을 반환하는 keyof 연산자를 제공한다.

타입 애너테이션처럼 타입을 사용하는 모든 곳에서 타입 이름 앞에 keyof 연산자를 배치한다.

```typescript
interface Ratings {
    audience: number;
    critics: number;
}

function getCountKeyof(ratings: Ratings, key: string): number {
    return ratings[key]; // Ok
}

const ratings: Ratings = { audience: 22, critics: 44 };

getCountKeyof(ratings, 'audience');

getCountKeyof(ratings, 'not valid'); // Error Argument of type not valid is not
```

keyof는 존재하는 타입의 키를 바탕으로 유니언 타입을 생성하는 훌륭한 기능이다. 또한 타입스크립트의 다른 타입 연산자와도 잘 결합되어 이 자의 후반부와
15장 타입 운영 에서 볼 수 있는 매우 멋진 패턴도 허용한다.

### 8.3.2 typeof

타입스크립트에서 제공하는 또 다른 타입 연산자는 typeof 이다. typeof 는 제공하는 값의 타입을 반환한다.

#### keyof typeof

typeof 는 값의 타입을 검색하고 keyof 는 타입에 허용된 키를 검색한다. 두 키워드를 함께 연결해 값의 타입에 허용된 키를 간결하게 검색 가능하다.

```typescript
const ratings = {
    imdb: 8.4,
    metacritic: 82,
};

function logRating(key: keyof typeof ratings) {
    console.log(ratings[key]);
};

logRating("imdb"); // OK

logRating("invalid"); // Error
```

## 9.4 타입 어서션

타입스크립트는 코드가 강력하게 타입화 될 때 가장 잘 작동한다. 코드의 모든 값이 정확히 알려진 타입을 가지는 경우이다.

타입스크립트의 타입 검사기가 복잡한 코드를 이해할 수 있도록 top 타입과 타입 가드 같은 기능을 제공한다. 그러나 경우에 따라서는 코드가
어떻게 작동하는지 타입 시스템에 100% 정확하게 알리는 것이 불가능할 때도 있다.

예를 들어 JSON.parse 는 의도적으로 top 타입인 any 를 반환한다. JSON.parse 에 주어진 특정 문자열 값이 특정한 값 타입을 반환해야 한다는 것을
타입 시스템에 안전하게 알릴 수 있는 방법은 없다. 

타입스크립트는 값의 타입에 대한 타입 시스템에 이해를 재정의하기 위한 구문으로 타입 어서션을 제공한다. 다른 타입을 의미하는 값의 타입 다음에 as 키워드를 붙인다.

### 9.4.2 non-null 어서션

타입 어서션이 유용한 경우를 하나 더 살펴보자면 실제로는 아니고 이론적으로만 null 또는 undefined 를 포함할 수 있는 변수에서 null 과 undefined 를 제거할 때
타입 어서션을 주로 사용한다. 타입스크립트에서는 너무 흔한 상황이라 이와 관련된 약어를 제공한다. null 과 undefined 를 제외한 값의 전체 타입을 작성하는 대신
! 키워드를 사용하면 된다.

### 9.4.3 타입 어서션 주의 사항

any 타입과 마찬가지로 타입 어서션은 타입스크립트의 타입 시스템에 필요한 하나의 도피 수단이다. 따라서 any 타입을 사용할 때처럼 꼭 필요한 경우가 아니라면
가능한 한 사용하지 말아야 한다. 값의 타입에 대해 더 쉽게 어서션하는 것보다 코드를 나타내는 더 정확한 타입을 갖는 것이 좋다.


#### 어서션 vs 선언

변수 타입을 선언하기 위해서 타입 애너테이션을 사용하는 것과 초깃값으로 변수 타입을 변경하기 위해 타입 어서션을 사용하는 것 사이에는 차이가 있다.

변수의 타입 애너테이션과 초깃값이 모두 있을 때 타입스클비트의 타입 검사기는 변수의 타입 애너테이션에 대한 변수의 초깃값에 대해 할당 가능성 검사를 수행한다.

그러나 타입 어서션은 타입스크립트 타입 검사 중 일부를 건너뛰도록 명시적으로 지시한다.

따라서 타입 애너테이션을 사용하거나 타입스크립트가 초깃값에서 변수의 타입을 유추하도록 하는 것이 매우 바람직하다.

#### 어서션 할당 가능성

타입 어서션은 일부 값의 타입이 약간 잘못된 상황에서 작은도피 수단일 뿐이다.