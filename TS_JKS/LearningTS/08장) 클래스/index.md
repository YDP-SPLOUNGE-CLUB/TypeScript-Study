# 8.1 클래스 메서드

타입스크립트는 독립 함수를 이해하는 것과 동일한 방식으로 메서드를 이해한다.

매개변수 타입에 타입이나 기본값을 지정하지 않으면 any 타입을 기본으로 갖는다.

메서드를 호출하려면 허용 가능한 수의 인수가 필요하고 재귀 함수가 아니라면 대부분 반환 타입을 유추할 수 있다.

클래스 생성자는 매개변수와 관련하여 전형적인 클래스 메서드처럼 취급된다. 타입 스크립트는 메서드 호출 시 올바른 타입의 인수가
올바른 수로 제공되느지 확인하기 위해 타입 검사를 수행한다.

```typescript
class Greeted {
    constructor(message: string) {
        console.log(`As I always say: ${message}`);
    }
}

new Greeted("take chances");

new Greeted(); // Error
```

## 8.2 클래스 속성

타입스크립트에서 클래스의 속성을 읽거나 쓰려면 클래스에 명시적으로 선언해야 한다.

클래스 속성은 인터페이스와 동일한 구문을 사용해 선언한다. 클래스 속성 이름 뒤에는 선택적으로 타입 애너테이션이 붙는다.

**타입스크립트는 생성자 내의 할당에 대해서 그 멤버가 클래스에 존재하는 멤버인지 추론하려고 시도하지 않는다.**

```typescript
class FieldTrip {
    // destination 은 string 으로 명시적으로 선언하였지만
    // nonexisttent 는 속성을 선언하지 않았기에 할당은 허용되지 않는다.
    destination: string;

    constructor(destination: string) {
        this.destination = destination; // OK
        console.log(`We're going to ${this.destination}`);
        
        this.nonexistent = destination; // Error
    }
}
```

클래스 속성을 명시적으로 선언하면 타입스크립트는 클래스 인스턴스에서 무엇이 허용되고 허용되지 않는지.

빠르게 이해 가능하다. 클래스 인스턴스가 사용될 때 코드가 trip.nonexistent 와 같은 클래스 인스턴스에 존재하지 않는
멤버에 접근하려고 시도하면 타입스크립트는 타입 오류를 발생시킨다.

```typescript
class FieldTrip {
    // destination 은 string 으로 명시적으로 선언하였지만
    // nonexisttent 는 속성을 선언하지 않았기에 할당은 허용되지 않는다.
    destination: string;

    constructor(destination: string) {
        this.destination = destination; // OK
        console.log(`We're going to ${this.destination}`);

        this.nonexistent = destination; // Error
    }
}

const trip = new FieldTrip('planetaruim');

trip.destination; // OK
trip.nonexistent; // Error
```

### 8.2.2 초기화 검사

엄격한 컴파일러 설정이 활성화된 상태에서 타입스크립트는 undefined 타입으로 선언된 각 속성이
생성자에서 할당되었는지 확인한다. 이와 같은 엄격한 초기화 검사는 클래스 속성에 값을 할당하지 않는 실수를 예방 가능하다.

```typescript
class WithValue {
    immediate = 0;
    later: number;
    mayBeUndefined: number | undefined;
    unused: number; // error

    constructor() {
        this.later = 1;
    }
}
```

엄격한 초기화 검사가 없다면 비록 타입 시스템이 undefined 값에 접근할 수 없다고 말할지 몰라도

클래스 인스턴스는 undefined 값에 접근할 수 있다.

```typescript
class MissingInitializer {
    property: string;
}

new MissingInitializer().property.length; // TypeError
```

### 선택적 속성 / 읽기 전용 속성

```typescript
class MissingInitializer {
    property?: string;
}

class Quote {
    readonly text: string;

    constructor(text: string) {
        this.text = '';
    }
    
    emphasize() {
        this.text += "!";
        // Error
    }
}
```

원시 타입의 초깃값을 갖는 readonly 로 선언된 속성은 다른 속성과 조금 다르다. 이런 속성은 더 넓은 원싯값이 아니라
값의 타입이 가능한 한 좁혀진 리터럴 타입으로 유추된다.

타입스크립트는 값이 나중에 변경되지 않는다는 점을 알기 떄문에 더 공격적인 초기 타입 내로잉을 편하게 느낀다.

```typescript
class RandomQuote {
    readonly a: string = "a";
    readonly b = "b";

    constructor() {
        if (Math.random() > 0.5) {
            this.a = "learning a";
            this.b = "learning b"; // Error
        }
    }
}

const quote = new RandomQuote();

quote.a; // Type: string;
quote.b; // Type: "b"
```

## 8.3 타입으로서의 클래스

타입 시스템에서의 클래스는 클래스 선언이 런타임 값과 타입 애너테이션에서 사용할 수 있는 타입을
모두 생성한다는 점에서 상대적으로 독특하다.

타입스크립트는 클래스의 동일한 멤버를 모두 포함하는 모든 객체 타입을 클래스에 할당 가능하다.

```typescript
class SchollBus {
    getAbilites() {
        return ["magic", "shapeshifting"];
    }
}

function withSchollBus(bus: SchollBus) {
    console.log(bus.getAbilites());
}

withSchollBus(new SchollBus()); // OK

withSchollBus({
    getAbilites: () => ["bus"], // OK
});
```

## 8.4 클래스와 인터페이스

```typescript
interface Learner {
    name: string;

    study(hours: number): void;
}

class Student implements Learner {
    name: string;

    constructor(name: string) {
        this.name = name;
    }
    
    study(hours: number) {
        for (let i = 0; i < hours; i++) {
            console.log("...studying");
        }
    }
}

class Slacker implements Learner {
    // Error
}
```

인터페이스를 구현하는 것으로 클래스를 만들어도 클래스가 사용되는 방식은 변경되지 않는다.
클래스가 이미 인터페이스와 일치하는 경우 타입 검사기는 인터페이스의 인스턴스가 필요한 곳에서
해당 인스턴스를 사용할 수 있도록 허용한다.

타입 애너테이션을 제공하지 않으면 암시적인 any 타입 오류가 발생한다.

```typescript
interface Learner {
    name: string;

    study(hours: number): void;
}

class Student implements Learner {
    name;
    // Error Meber 'name' implicity ahs an 'any' type
    
    constructor(name) {
        this.name = name;
    }
    
    study(hours) {
    // Error Meber 'hours' implicity ahs an 'any' type
        for (let i = 0; i < hours; i++) {
            console.log("...studying");
        }
    }
}
```

### 8.5.2 재정의된 생성자

바닐라 자바스크립트와 마찬가지로 타입스크립트에서는 하위 클래스는 자체 생성자를 정의할 필요가 없다.

자바스크립트에서 하위 클래스가 자체 생성자를 선언하면 super 키워드를 통해 기본 클래스 생성자를 호출해야 한다.

타입스크립트의 타입 검사기는 기본 클래스 생성자를 호출할 때 올바른 매겨변수를 사용하는지 확인한다.

```typescript
class GradeAnnoucer {
    message: string;

    constructor(grade: number) {
        this.message = grade > 65 ? "maybe next time..." : "you pass";
    }
}

class PassingAnnoucer extends GradeAnnoucer {
    constructor() {
        super(100);
    }
}

class FaillingAnnouncer extends GradeAnnoucer {
    constructor() {
        // Error
    }
}
```

## 8.6 추상 클래스

때로는 일부 메서드의 구현을 선언하지 않고 대신 하위 클래스가 해당 메서드를 제공할 것을 예상하고

기본 클래스를 만드는 방법이 유용할 수 있다. 추상화 하려는 클래스 이름과 메서드 앞에 
**타입스크립트의 abstract** 키워드를 추가한다.

```typescript
abstract class School {
    readonly name: string;

    constructor(name: string) {
        this.name = name;
    }
    
    abstract getStudentTypes(): string[];
}

class Preschool extends School {
    getStudentTypes() {
        return ["prescholler"];
    }
};

class Absence extends School {};
// Error Nonabstract class 'ABsence'
```

## 8.7 멤버 접근성

자바스크립트에서는 클래스 멤버 이름 앞에 # 을 추가해 private 클래스 멤버임을 나타낸다.

타입스크립트의 클래스 지원은 자바스크립트의 # 프라이버시보다 먼저 만들어졌다.

또한 타입스크립트는 private 클래스 멤버를 지원하지만 타입 시스템에만 존재하는 클래스 메서드와
속성에 대해 조금 더 미묘한 프라이버시 정의 집합을 허용한다.

- public (기본값): 모든 곳에서 누구나 접근 가능
- protected: 클래스 내부 또는 하위 클래스에서만 접근 가능
- private 클래스 내부에서만 접근 가능

이러한 키워드는 순수하게 타입 시스템 내에 존재하며 컴파일시 함께 키워드도 제거된다.

타입스크립트의 멤버 접근성은 타입 시스템에만 존재하는 반면 자바스크립트의 private 선언은
런타임에도 존재하는 점이 주요 차이점이다.

### 8.7.1 정적 필드 제한자

자바스크립트 static 키워드를 사용해 클래스 자체에서 멤버를 선언한다. 타입스크립트는 static 키워드를
단독으로 사용하거나 readonly 와 접근성 키워드를 함께 사용할 수 없도록 지원한다. 함께 사용할 경우 접근성 키워드를
먼저 작성하고 그다음 static, readonly 키워드가 온다.

```typescript
class Question {
    protected static readonly answer: "bash";
    protected static readonly prompt = 
        "What`s an ogre`s favorite programmming language?";
    
    guess(getAnswer: (propmt: string) => string) {
        const answer = getAnswer(Question.prompt);
        
        if (answer === Question.answer) {
            console.log("You got it");
        } else {
            console.log("Try Again");
        }
    }
}

Question.answer; // Error Property 'answer' is protected and only
```