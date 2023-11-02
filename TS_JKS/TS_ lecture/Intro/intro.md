# 학습 내용

1. TS 기본 문법

2. 각종 라이브러리 Type 분해
    - Axios
    - React
    - Node
    - Express
    - JQuery
    - Redux

---

# 기본 지식

- 메인 룰: typescript는 최종적으로 javascript로 변환된다. 순전한 typescript 코드를 돌릴 수 있는 것은 deno이나 대중화되지가 않았음. 브라우저, 노드는 모두 js 파일을 실행한다.

- typescript는 언어이자 컴파일러(tsc)이다. 컴파일러는 ts 코드를 js로 바꿔준다.

- tsc는 tsconfig.json 에 따라 ts 코드를 js로 바꿔준다. 인풋인 ts 와 아웃풋인 js 모두에 영향을 끼치므로 tsconfig.json 설정을 반드시 봐야한다.

- 단순히 타입 검사만 하고싶다면 tsc --noEmit 하면 된다.

- tsconfig.json에서 그냥 esModuleInterop: true, strict: true 두 개만 주로 켜놓는 편. strict: true가 핵심임

- ts 파일을 실행하는 게 아니라 결과물인 js를 실행해야 한다.


## tsconfig.json

중요 포인트점.

- allowJS : JS 와 TS 코드를 같이 실행가능하도록 해준다.
- strict: 무조건 True 해야한다. 아니면 TS 의 의미가 없어진다.
- target: 컴파일을 할 ECMAScript 버전을 설정 가능하다. ex) ES5, ES6, ES2022
- forceConsistentCasingInFileNames: TS 에서는 다른 파일을 import 할 수 있는데 대소문자를 꼭 지켜서만 임포트를 가능하게 설정가능하다.
- skiplibCheck: 라이브러리 d.ts 를 스킵해서 검사할지 안할지 정할 수 있다. 보통은 true 로 설정하는데 이유는 컴파일 속도가 차이가 나기 떄문.

---