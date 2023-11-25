# 11. 선언 파일

타입스크립트는 구현과 별도로 타입 형태를 선언할 수 있다. 타입 선언은 파일 이름이 .d.ts 확장자로 끝나는 선언 파일에 작성된다. 선언 파일은 일반적으로 프로젝트 내에서 작성되고

프로젝트의 컴파일된 npm 패키지로 빌드 및 배포되거나 독립 실행형 typings 패키지로 공유된다.

## 11.1 선언 파일

.d.ts 선선 파일은 런타임 코드를 포함할 수 없다는 주목할 만한 제약 사항을 제외하고는 .ts 와 유사하게 작동한다.

.d.ts 파일에는 사용 가능한 런타임 값, 인터페이스, 모듈, 일반적인 타입의 설명만 포함된다. .d.ts 파일은 자바스크립트로 컴파일할 수 있는 모든 런타임 코드를
포함할 수 없다.

```typescript
// types.d.ts
export interface Character {
  catchphrase?: string;
  name: string;
}

// index.ts
import { Chapter } from "./types";

export const charcter: Character = {
  catchphrase: "1",
  name: "name"
}
```

## 11.2 런타임 값 선언

비록 선언 파일은 함수 또는 변수 값은 런타임 값을 생성하지 않을 수 있지만 declare 키워드를 사용해 이러한 구조체가 존재한다고 선언할 수 있다.

이렇게 하면 웹 페이지의 <script> 태그 같은 일부 외부 작업이 특정 타입의 이름을 사용해 값을 생성했음을 타입 시스템에 알린다.

declare 로 변수를 선언하면 초깃값이 허용되지 않는다는 점을 제외하고 일반적인 변수 선언과 동일한 구문을 사용한다.

다음 스니펫은 declared 변수를 성공적으로 선언하지만 intializer 변수에 값을 제공하려고 하면 타입 오류가 발생한다.

```typescript
// types.d.ts
declare let declare: string;

declate let initializer: string = "wanda"; // Error Initializers are not allowed in ambient context;
```

함수와 클래스도 일반적인 형식과 유사하게 선언되지만 함수 또는 메서드에 본문이 없다.

```typescript
// fairies.d.ts
declare function canGreatWish(wish: string): boolean; // OK

declare function greatWish(wish: string) { return true}; // Error ... are not allowed in ambient context;

class Fairy {
  canGrantWish2(wish: string): boolean; // OK
  
  grantWish(wish: string) {
    return true;
  } // Error
}
```

declare 키워드를 사용한 타입 선언은 .d.ts 선언 파일에서 사용하는게 가장 일반적이지만.
선언 파일 외부에서도 사용할 수 있다.

```typescript
// index.ts
declare const myGlobalValue: string;

console.log(myGlobalValue); // OK
```

### 11.2.1 전역 변수

import 또는 export 문이 없는 타입스클비트 파일은 모듈이 아닌 스크립트로 취급되기 때문에 여기에 선언된 타입을 포함한 구문은 전역으로 사용된다.

import 또는 export가 없는 선언 파일은 해당 동작의 이점을 사용해 타입을 전역으로 선언할 수 있다.

```typescript
// globlas.d.ts
declare const version: stringl;

// version.ts
export function logVersion() {
  console.log(`Version: ${version}`); // OK
}
```

### 11.2.2 전역 인터페이스 병합

변수는 타입스크립트의 타입 시스템에서 떠돌아다니는 유일한 전역은 아니다. 전역 API 와 값에 대한 많은 타입 선언이 전역으로 존재한다.

인터페이스는 동일한 이름의 다른 인터페이스와 병합되기 때문에 import 와 export 문이 없는 .d.ts 선언 파일 같은 전역 스크립트 컨텍스트에서
인터페이스를 선언하면 해당 인터페이스가 전역으로 확장된다.

### 11.2.3 전역 확장

다른 곳에 정의된 타입을 가져와서 전역정의를 크게 단순화할 때와 같이 전역 범위로 확장이 필요한 .d.ts 파일에 import 또는 export 문을 항상 금지할 수 있는 것은 아니다.

경우에 따라서 모듈 파일에 선언된 타입이 전역으로 사용되어야 한다.

```typescript
// types.d.ts
// (모듈 컨텍스트)
declare global {
  // (전역 컨텍스트)
}

// (모듈)
```

## 11.4 모듈 선언

선언 파일의 또 다른 중요한 기능은 모듈의 상태를 설명하는 기능이다. 모듈의 문자열 이름 앞에 declare 키워드를 사용하면 모듈의 내용을 타입 시스템에 알릴 수 있다.

```typescript
// modules.d.ts
declare module "my-example-lib" {
  export const value: string;
}

// index.ts
import { value } from "my-example-lib";

console.log(value); // OK
```

코드에서 declare module 을 자주 사용해서는 안된다. declare module 은 주로 다음 절에 나오는 와일드카드 모듈 선언과 이 장의 후반부에서 다루는 패키지 타입과
함께 사용된다. 또한 타입스크립트가 .json 파일의 가져오기를 기본으로 인식하도록 설정하는 컴파일러 옵션인 resolveJsonModule 이 있다.

### 11.4.1 와일드카드 모듈 선언

모듈 선언은 자바스크립트와 타입스크립트 파일 확장자가 아닌 특정 파일의 내용을 코드로 가져올 수 있음을 웹 어플리케이션에 알리기 위해 사용된다.
모듈 선언으로 하나의 * 와일드카드를 포함해 해당 패턴과 일치하는 모든 모듈을 나타낼 수 있다.

```typescript
// styles.d.ts

declare module "*.module.css" {
  const styles: { [i: string]: string };
  export default styles;
}

// component.ts
import styles from "./styles.module.css";

styles.anyClassName; // 타입 string;
```

## 11.5 패키지 타입

프로젝트 내에서 declare를 사용하는 방법을 살펴봤으므로 패키지 간에 타입을 사용하는 방법을 다룰 차례이다.

타입스크립트로 작성된 프로젝트는 여전히 .js 로 컴파일된 파일이 포함된 패키지를 배포한다.

일반적으로 .d.ts 파일을 사용해 이러한 자바스크립트 파일 뒤에 타입스크립트 타입 시스템 형태를 지원하도록 선언한다.

### 11.5.1 선언

타입스크립트는 입력된 파일에 대한 .d.ts 출력 파일과 자바스크립트 파일을 함께 선언하는 선언 옵션을 제공한다.

```typescript
// index.ts

export const greet = (text: string) => {
  console.log(`Hello ${text}`);
}
```

module 은 es2015, target es2015 인 선언 옵션을 사용해 다음 출력 파일을 생성한다.

```typescript
// index.d.ts
export declare const greet: (text: string) => void;
```

```javascript
// index.js
export const greet = (text) => {
  console.log(`Hello ${text}`)
}
```

### 11.5.2 패키지 타입 의존성 

타입스크립트는 프로젝트의 node_module 의존성 내부에서 번들로 제공되는 .d.ts 파일을 감지하고 활용할 수 있다. 이러한 파일은 해당 패키지에서
내보낸 타입 형태에 대해 마치 동일한 프로젝트에서 작성되었거나 선언 모듈 블록으로 선언된 것처럼 타입 시스템에 알린다.

자체 .d.ts 선언 파일과 함꼐 제공되는 npm 모듈은 대부분 다음과 같은 파일 구조를 갖느다.
lib/
    index.js
    index.d.ts
package.json

### 11.5.3 패키지 타입 노출

프로젝트가 npm 에 배포되고 사용자를 위한 타입을 제공하려면 패키지의 package.json 파일에 types 필드를 추가해 루트 선언 파일을 가리킨다.
types 필드는 main 필드와 유사하게 작동하고 종종 동일한 것처럼 보이지만 .js 확장자 대신에 .d.ts 확장자를 사용한다.

예를 들어 다음 가상의 패키지 파일에서 main 런타임 파일인 ./lib/index.js 는 types 파일인 ./lib/index.d.ts 와 병렬 처리된다.

```json
{
  "author": "pedant Publishing",
  "main": "./lib/index.js",
  "name": "coffetable",
  "types": "./lib/index.d.ts",
  "version": "0.5.22"
}
```

그런 다음 타입스크립트는 유틸리티 패키지에서 가져온 파일을 사용하기 위해 제공해야 하는 것으로 ./lib/index.d.ts 파일을 사용한다.

types 필드가 패키지의 package.json 에 존재하지 않으면 타입스크립트는 ./index.d.ts 를 가진 기본값으로 가정한다. 

types 필드가 지정되지 않은 경우 ./index.js 파일을 패키지의 기본 진입점으로 가정하는 npm 의 기본 동작을 반영한 것이다.

## 11.6 DefinitelyTyped

타입스크립트 프로젝트는 여전히 해당 패키지에서 모듈의 타입 형태를 알려줘야 한다.

타입스크립트 팀과 커뮤니티는 커뮤니티에서 작성된 패키지 정의를 수용하기 위해 DefinitelyTyped 라는 거대한 저장소를 만들었다.

깃허브에서 가장 활발한 저장소 중 하나이다. 저장소에는 변경 제안 검토 및 업데이트 게시와 관련된 자동화 부분과 수천 개의 .d.ts 정의 패키지가 포함되어 있다.

DT 패키지는 타입을 제공하는 패키지와 동일한 이름으로 npm에 @types 범위로 게시된다.