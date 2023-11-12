# 05. 함수

## 5.1 함수 매개변수

```typescript
function sing(song) {
    console.log(`Singing ${song}`);
}
```

sing 함수를 작성한 개발자가 song 매개변수를 제공하기 위해 의도한 값의 타입은 무엇일까?
명시적 타입 정보가 선언되지 않으면 절대 타입을 알 수 없다.

뱐수와 마찬가지로 타입스크립트를 사용하면 타입 에너테이션으로 함수 매개변수의 타입을 선언 사능하다.

```typescript
function sing(song: string) {
    console.log(`Singing ${song}`);
}
```

### 5.1.2 선택적 매개변수

자바스크립트에서 함수 매개변수가 제공되지 않으면 함수 내부의 인숫값은 undefined 로 기본값이 서정된다.

떄로는 함수 매개변수를 제공할 필요가 없을 때도 있다. 또는 의도적으로 undefined 를 사용할 때도 있다.

에너테이션 : 앞에 ? 를 붙이면 방지 가능하다.

### 5.1.3 기본 매개변수

자바스크립트에서 선택적 매개변수를 선언할 때 = 와 값이 포함된 기본값을 제공할 수 있다.

타입스크립트 타입에는 암묵적으로 함수 내부에 | undefined 유니언 타입이 추가된다. 타입스클비트는 함수의 매개변수에 대해인수를 누락하거나
undefined 인수를 사용해서 호출하는 것을 여전히 허용한다.

타입스크립트의 타입 추론은 초기 변숫값과 마찬가지로 기본 함수 매개변수에 대해서도 유사하게 작동한다.

매개변수에 기본값이 있고 타입 에너테이션이 없는 경우 타입스크립트는 해당 기본값을 기반으로 매개변수 타입을 유추한다.

```typescript
function rateSong(song: string, rating = 0) {
    console.log(`${song} get ${rating}/5 stars!` );
}

rateSong("Photograph"); // OK
rateSong("Set Fire to the Rain"); // OK
rateSong("Set Fire to the Rain", undefined); // OK

rateSong("Set Fire to the Rain", "100"); // Error
// parameter of type number | undefined
```

### 5.1.4 나머지 매개변수

자바스크립트의 일부 함수는 임의의 수의 인수로 호출할 수 있도록 만들어진다. ... 스프레드 연산자는 함수 선언의 마지막 매개변수에 위치하고
해당 매개변수에서 시작해 함수에 전달된 나머지 인수가 모두 단일 배열에 저장되어야 함을 나타낸다.

타입스크립트는 이러한 나머지 매개변수의 타입을 일반 매개변수와 유사하게 선언할 수 있다.

인수 배열을 나타내기 위해 끝에 [] 구문이 추가된다는 점만 다르다.

```typescript
function singAllTheSongs(singer: string, ...songs: string[]) {
    for (const song of songs) {
        console.log(`${song}, by ${singer}`);
    }
};

singAllTheSongs("a"); // OK
singAllTheSongs("a", "a song1", "a song2"); // OK
singAllTheSongs("a", 2000); // Error number is not assignable toparameter of type string
```

## 5.2 반환 타입

타입스크립트는 지각적이다. 함수가 반환할 수 있는 가능한 모든 값을 이해하면 함수가 반환하는 타입을 알 수 있다.

singSongs 는 타입스크립트에서 number 를 반환하는 것으로 파악된다.

```typescript
function singSongs(songs: string[]) {
    for (const song of songs) {
        console.log(`${song}`)
    };
    
    return songs.length;
}
```

함수에 다른 값을 가진 여러 개의 반환문을 포함하고 있다면 타입스크립트는 반환 타입을 가능한 모든 반환 타입의 조합으로 유추한다.

### 5.2.1 명시적 반환 타입

변수와 마찬가지로 타입 에너테이션을 사용해 함수의 반환 타입을 명시적으로 선언하지 않는 것이 좋다. 그러나 특히 함수에서
반환 타입을 명시적으로 선언하는 방식이 매우 유용할 때가 종종 있다.

1. 가능한 반환값이 많은 함수가 항상 동일한 타입의 값을 반환하도록 강제한다.
2. 타입스크립트는 재귀 함수의 반환 타입을 통해 타입을 유추하는 것을 거부한다.
3. **수백 개 이상의 타입스크립트 파일이 있는 매우 큰 프로젝트에서 타입스크립트 타입 검사 속도를 늘릴 수 있다.**

함수 선언 반환 타입 애너테이션은 매개변수 목록이 끝나는 ) 다음에 배치된다. 함수 선언의 경우 { 앞에 배치 된다.

화살표 함수의 경우 => 앞에 배치된다.

## 5.3 함수 타입

자바스크립트에서 함수를 값으로 전달할 수 있다. 함수를 가지기 위한 매개변수 또는 변수의 타입을 선언하는 방법이 필요하다.

함수 타입 구문은 화살표 함수와 유사하지만 함수 본문 대신 타입이 있다.

```typescript
let nothingInGivesString: () => string;
```

매개변수가 없고 string 을 반환하는 함수다.

```typescript
let inputAndOutput: (songs: string[], count?: number) => number;
```

매개변수가 string [], count는 undefined | number 에 number 를 반환하는 함수이다.

### 5.3.1 함수 타입 괄호

함수 타입은 다른 타입이 사용되는 모든 곳에 배치할 수 있다. 여기에는 유니언 타입도 포함된다.

유니언 타입의 에너테이션에서 함수 반환 위치를 나타내거나 유니언 타입을 감싸는 부분을 표시할 때 괄호를 사용한다.

### 5.3.2 매개변수 타입 추론

매개변수로 사용되는 인라인 함수를 포함하여 작성한 모든 함수에 대해 매개변수를 선언해야 한다면 번거로울 것이다. 다행이도 타입스크립트는
선언된 타입의 위치에 제공된 함수의 매개변수 타입을 유추할 수 있다.

```typescript
let singer: (song: string) => string;

singer = function (song) {
    return `Singing ${song.toUpperCase()}`; // OK
};
```

## 5.4 그 외 반환 타입

void, never 에 대해서 알아보자

### 5.4.1 void 반환 타입

<a href="https://github.com/YDP-SPLOUNGE-CLUB/typescript-study/wiki/InflearnTS-Chapter-7~9">**올인원 타입스크립트 void 와 동일한 내용.**</a>

### 5.4.2 never 반환 타입

<a href="https://github.com/YDP-SPLOUNGE-CLUB/typescript-study/wiki/TypeScript-Never">**Never 정리 위키**</a>

## 5.5 함수 오버로드

일부 자바스크립트 함수는 선택적 매개변수와 나머지 매개변수만으로 표현할 수 없는 매우 다른 매개변수들로 호출될 수 있다.

이러한 함수는 오버로드 시그니쳐? 라고 불리는 타입스크립트 구문으로 설명할 수 있다.

즉 최종 구현 시그니쳐와 그 함수의 본문 앞에 서로 다른 비전의 함수 이름 매개변수 반환 타입을 여러 번 선언한다.

오버로드된 함수 호출에 대해 구문 오류를 생성할지 여부를 결정할 때 타입스크립트는 함수의 오버로드 시그니쳐만 확인한다. 구현 시그니쳐는 함수의 내부 로직에서만 사용된다.

```typescript
// 오버로드 시그니쳐
function createDate(timestamp: number): Date;
function createDate(month: number, day: numbet, year: number): Date;

// 구현 시그니쳐
function createDate(monthOrTimestamp: number, day?: number, year?: number) {
    return day === undefined || year === undefined
        ? new Date(monthOrTimestamp)
        : new Date(year, monthOrTimestamp, day);
};

createDate(554356800); // OK
createDate(7, 21 , 1111); // OK

createDate(1,1); // Error
```

타입스크립트를 컴파일해 자바스크립트로 출력하면 다른 타입 시스템 구문처럼 오버로드 시그니쳐도 삭제된다.

### 5.5.1 호출 시그니처 호환성

오버로드된 함수의 구현에서 사용되는 구현 시그니처는 매개변수 타입과 반환 타입에 사용하는 것과 동일하다. 따라서 함수의 오버로드 시그니처에 있는 반환 타입과
각 매개변수는 구현 시그니처에 있는 동일한 인덱스의 매개변수에 할당할 수 있어야 한다.

구현 시그니처는 모든 오버로드 시그니처와 호환되어야 한다.

```typescript
function format(data: string): string; // OK
function format(data: string, needle: string, haystack: string): string // OK

// Error Function implementation is missing or not immediately following the declaration
function format(get: () => string); string;

function format(data: string, needle?: string, haystack?: string) {
    return needle && haystack ? data.replace(needle, haystack): data;
}
```