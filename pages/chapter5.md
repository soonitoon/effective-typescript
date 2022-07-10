# 5장

any 다루기

## item 38

`any` 타입은 가능한 한 좁은 범위에서만 사용하기

- 타입스크립트는 적정임과 동시에 동적인 언어임.
- 때문에 다른 정적 타입 언어와는 다르게 **점진적인 타입 시스템 도입**이 가능함.
- `any`는 이런 점진적 타입 시스템에서 중요한 역할을 하는 타입임.
- `any` 타입을 통해 타입 시스템을 무력화시킬 수 있기 때문에 매우 신중하게 사용해야 함.

```typescript
const x = returnValue();

someFunction(x);
// 오류: x는 매개변수에 할당될 수 없음!

// 이 가상의 오류 상황에서 any를 쓰는 방법은 두 가지임.
const x: any = returnValue();
someFunction(x);
// 타입 체크 통과

const x = returnValue();
someFunction(x as any);
// 타입 체크 통과

/**
 * 첫 번째 방법보다 두 번째 방법이 권장됨.
 * x 자체를 any로 만듦으로서 코드의 다른 부분에서 문제가 발생할 수 있기 때문.
 * 예: x에 대한 자동 완성 서비스가 꺼짐.
 * x는 어떤 함수에도 매개변수로 들어갈 수 있음.
 **/
```

- 이러한 `any`의 좁은 사용은 객체에서도 마찬가지로 적용됨.

```typescript
const person: Person = {
  name: "choi",
  age: 23,
  x: {
    y: "Y",
    // 타입 오류!
  },
};

// 객체 전체를 any로 만들면 오류가 해결됨.

const person: Person = {
  name: "choi",
  age: 23,
  x: {
    y: "Y",
  },
} as any;

/**
 * 하지만 위에서와 동일한 문제가 발생함.
 * name, age에 대해 자동 완성 기능이 적용되지 않음.
 * 따라서 객체 전체를 any로 선언하기보다, 문제가 발생한 부분만 any로 두는 것이 낫다.
 **/

const person: Person = {
  name: "choi",
  age: 23,
  x: {
    y: "Y" as any,
  },
};
```

- 그 어떤 경우에도 함수가 `any` 타입을 반환해서는 안 됨.
- 강제로 타입 오류를 제거할 필요가 있을 땐 `any`를 양산하는 것보다 `@ts-ignore` 주석을 사용하는 것이 낫다.
- 그렇게 함으로써 프로젝트 전체에 `any` 타입의 영향이 미치는 것을 막을 수 있기 때문.

## item 39

`any`를 구체적으로 변형해서 사용하기

- `any`보다 구체적인 타입을 사용할 수 있다면 구체적인 타입을 사용하는 것이 타입 안정성에 좋다.

```typescript
function getLengthBad(array: any) {
  return array.length;
}
// 나쁜 예시.

function getLengthGood(array: any[]) {
  return array.length;
}
// any를 구체화시킨 좋은 예시
```

- 이렇게 `any`를 구체화시키면 세 가지 면에서 더 좋다.
  1. 함수 내에서 `array.length` 타입을 사용할 수 있음.
  2. 반환 타입이 `any`에서 `number`로 좁혀짐.
  3. 함수를 호출할 때 매개변수가 배열인지의 여부를 체크할 수 있게 됨.
- 매개변수가 객체인데 어떤 객체인지 알 수 없다면 `{[key: string]}: any` 타입을 쓸 수 있음.
- `object` 타입의 경우 키 열거는 가능하지만 키를 통해 속성을 참조할 수는 없다.

```typescript
function hasTwolveLetterKey(o: { [key: string]: any }) {
  for (const key in o) {
    if (key.length === 12) {
      return true;
    }
  }
  return false;
}
// {[key: string]: any} 타입을 사용한 매개변수.
// 키 열거가 가능하다.

function hasTwolveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
      // {} 형삭에 인덱스 시그니쳐가 없으므로 암시적으로 any임!
      return true;
    }
  }
  return false;
}
// object 타입을 사용한 매개변수.
// 키를 통해 값에 참조할 수 없다.
```

- 함수 타입에서도 `any`를 가능하다면 최대한 좁혀 사용해야 함.

```typescript
type Fn0 = () => any; // 매개변수가 없는 모든 함수
type Fn1 = (arg: any) => any; //매개변수가 1개인 모든 함수
type FnN = (...args: any[]) => any; // n개의 매개변수를 가지는 모든 함수, Function 타입과 동일.
```

## item 40

함수 안으로 타입 단언문 감추기

- 함수를 작성하다보면 타입 단언이 필요한 상황이 있음.
- 이때 단언문을 함수 외부에서 사용하는 것보다는, 내부에서 사용하고 함수 자체의 타입은 명확하게 해주는 것이 좋다.

```typescript
declare function shallowEqual(a: any, b: any): boolean;

function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    // 에러! (...args: any[]) => any 형식은 T에 할당할 수 없음!
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
// cacheLast 함수의 반환 타입(T)과 반환 함수 간 관련성을 추론할 수 없기 때문에 오류가 발생함.
// cacheLast 함수의 반환값을 T라고 단언해주면 됨.

function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[] | null = null;
  let lastResult: any;
  return function (...args: any[]) {
    // 에러! (...args: any[]) => any 형식은 T에 할당할 수 없음!
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
  // 반환값에 타입 단언 사용.
}
// 또한, 함수 내부에서는 any를 자주 사용했지만 사용하는 쪽에서는 신경쓸 필요가 없음. 함수 시그니쳐만 중요하기 때문.
```

- 위의 예제에서는 배열이 똑같은지 확인하는 `shallowEqual`을 사용했지만, 객체의 각 키와 값들이 동일한지를 검사한다면 조금 더 복잡한 처리가 필요함.

```typescript
declare function shallowEqual(a: any, b: any): boolean;

function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k])
      // 에러! {} 형식에 인덱스 시그니처가 없으므로 암시적으로 any가 포함됨.
      return false;
  }
  return Object.keys(a).length === Object.keys(b).length;
}
// if문으로 k기 b 안에 있는 속성이라는 것을 확인했음에도 불구하고 타입스크립트의 한계로 b[k] 부분에서 오류가 발생함.
// 이런 부분은 타입 단언으로 해결할 수 있음.

declare function shallowEqual(a: any, b: any): boolean;

function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) return false;
  }
  return Object.keys(a).length === Object.keys(b).length;
}
// b의 타입을 any로 단언함으로써 k로 b의 속성에 접근할 수 있게 되었음.
```

- 결론: 일반적으로 `any` 타입은 지양해야 하지만, 상황에 따라 현실적인 해결책이 되기도 한다. `any`를 꼭 사용해야 한다면 명확한 타입을 가지는 함수 내부에서 사용해야 한다.

## item 41

`any`의 진화를 이해하기

- 다른 타입들과는 달리, `any` 타입은 결정된 이후에도 **진화**할 수 있음.

```typescript
function range(start: number, limit: number) {
  const out = []; // type out = any[];
  for (let i = start; i < limit; i++) out.push(i); // type out = any[];
  return out; // type out = number[];
}
// out은 any[] 타입으로 선언되었지만, 마지막에 가서는 number[] 타입으로 '진화'했다.
```

- `any` 혹은 `any[]` 타입이 진화하는 이유는 어떤 값이 할당될 수 있느냐에 따라 타입이 유동적으로 변하기 때문.
- 이는 '타입 좁히기' 개념과는 다르다.

```typescript
const result = []; // any[]
result.push("a");
result; // string[]
result.push(1);
result; // (string | number)[]

let val; // any
if (Math.random() < 0.5) {
  val = "hello";
} else {
  val = 12;
}
val; // number | string

// 타입 좁히기 개념과 any의 진화가 다른 이유.
// 타입 좁히기가 타입이 가질 수 있는 가능성을 제거해나가는 과정이라면, any 진화는 할당될 수 있는 타입의 가능성을 늘리는 과정이라고 할 수 있다.
```

- `any`의 진화는 **암시적 any** 타입에서만 일어난다.
- **명시적 any** 타입에서 진화는 일어나지 않으며, `any` 타입이 지속적으로 유지된다.

```typescript
let val: any; // any
if (Math.random() < 0.5) {
  val = "hello";
} else {
  val = 12;
}
val; // any
```

- `any`의 진화는 암시적 `any` 타입의 값에 어떤 타입의 값을 할당하려 할 때 발생한다.
- 따라서 함수 호출 등으로 진화가 일어나진 않는다.

```typescript
function makeSquares(start: number, limit: number) {
  const out = []; // 암시적 any[] 오류
  range(start, limit).forEach((i) => out.push(i * i));
  return out; // 암시적 any[] 오류
}
```

- `any`의 진화를 사용하는 것은 권장되지 않는다.
- 그보다는 명시적으로 올바른 타입을 선언해주는 것이 훨씬 나은 설계다.

## item 42

모르는 타입의 값에는 `any` 대신 `unknown`을 사용하기

- 왜 `any` 대신 `unknown`을 사용해야 할까?

```typescript
function parseYAML(yaml: string): any {
  // YAML을 파싱하는 코드
  // 파싱 결과가 어떤 형태일지 알 수 없기 때문에 반환 타입이 any다.
}

interface Book {
  name: string;
  author: string;
}

const book: Book = parseYAML(`
  name: 원미동 사람들
  author: 양귀자
`);

// parseYAML의 반환값이 암시적 any이기 때문에 book 변수에 타입 선언을 도입했다.
// 하지만 타입 선언은 강제가 아님.
// 만약 타입 선언을 빼먹었다면, book 변수는 암시적 any 타입인 상태로 코드를 돌아다니게 됨.

book.title; // 존재하지 않는 속성이지만 undefined로 정상이 됨.
```

- 이러한 문제점을 `unknown` 타입을 사용함으로써 명확히 할 수 있게 된다.

```typescript
function safeParseYAML(yaml: string): unknown {
  return parseYAML(yaml);
}

const book = safeParseYAML(`
  name: 원미동 사람들
  author: 양귀자
`);

book.title; // 오류! unknown 형식임!
// 이제 사용자가 타입 선언을 추가해야 한다는 것이 '강제된다'
```

- `any` 타입이 위험한 이유는 **할당 가능성** 때문.
  - 모든 타입은 `any` 타입에 할당 가능하다.
  - `any` 타입은 모든 타입에 할당 가능하다.
- 이 두 속성이 `any`가 타입 시스템을 망치는 원인이 된다.
- `unknown`은 `any`보다 안전한 속성을 가지고 있다.
  - 모든 타입은 `unknown`에 할당 가능하다.
  - **`unknown` 타입은 `unknown`과 `any` 타입에만 할당 가능하다.**
- 이 두 번째 속성 덕분에 개발자는 `unknown` 타입을 좁혀 사용하도록 강제된다.

```typescript
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}

// instanceof 연산을 통해 타입 좁히기
function processValue(val: unknown) {
  if (val instanceof Date):
    val // type val = Date
}

// 사용자 정의 타입 가드를 사용해 타입 좁히기(이전 예제)
function isBook(val: unknown): val is Book {
  return (
    typeof(val) === 'object' && val !== null && 'name' in val && 'author' in val
  );
  // in 연산자 사용을 위해 객체인지 먼저 체크
  // null도 객체이므로 null 여부 체크
  // Book 타입과 부합하는지 마지막으로 체크
}

function processValue(val: unknown) {
  if (isBook(val)) {
    val // type val = Book
  }
}
```

- `unknown` 대신 제너릭을 사용하여 반환 타입을 설정하게 하는 경우도 있는데, 일반적으로 추천하는 방식은 아님.
- 이중 단언문에서도 `any` 보다는 `unknown`을 사용하는 것이 안전함.

```typescript
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

- 이중 단언에서 두 타입의 기능은 동일함.
- 하지만 추후 이중 단언을 분리할 일이 생기면 `any` 단언의 경우 문제가 심각해짐.
- `unknown` 타입이 나오기 전에는 비슷한 역할을 `{}` 혹은 `object` 타입을 사용했음.
  - `{}` 타입은 `null`과 `undefined`를 제외한 모든 값을 포함함.
  - `object` 타입은 비기본형 타입을 의미하며, 따라서 기본형 타입(문자열, 숫자, 불리언 등)은 포함되지 않는다.
- `unknown` 타입이 나온 이후로는 거의 쓰이지 않음.

## item 43

몽키 패치보다는 안전한 타입 사용하기

- 자바스크립트의 타입 시스템에서는 객체 및 클래스에 임의의 속성을 추가할 수 있다.
- 내장 기능의 프로토타입에도 속성을 추가할 수 있음.

```typescript
RegExp.prototype.monkey = "monkey";
/123/.monkey = "monkey";
// 혹은
window.monkey = "monkey";
```

- 일반적으로 권장되는 방법은 아님.
- 몽키 패치의 범위가 전역이므로 사이드 이펙트가 발생할 확률이 높아진다.
- 또한 타입 체커는 임의로 추가한 속성에 대해 아무것도 모른다.

```typescript
document.monkey = "monkey"(
  // 오류! 'Document' 유형에 'monkey' 속성이 없음!

  // any 딘언을 사용하면 타입 오류를 없앨 수 있음.
  document as any
).monkey = "monkey"; // 정상

// 하지만 document 객체와 관련된 모든 언어 서비스를 사용할 수 없게 됨.
```

- 가장 최선의 방법은 `document`나 `DOM`에 굳이 데이터를 끼워넣으려고 하지 않는 것.
- 하지만 어쩔 수 없다면 두 가지 방법이 존재한다.
  - `interface`의 보강 기법을 사용하기
  - 차라리 `any` 단언보다 구체적인 단언을 사용하기

```typescript
// 보강 기법 사용하기
interface Document {
  monkey: string;
}

document.monkey; // 정상

// 보강을 사용하면 any 단언과 다르게 기존 언어 서비스를 유지할 수 있음(새로 추가한 속성에 대해서도).
// 또한, 내장 객체에 어떤 속성을 추가했다는 기록이 명시적으로 남음.

// 만약 import/export를 사용한다면 global 선언을 추가해야 함.
export {...} from ...;

declare global {
  interface Document {
    monkey: string;
  }
}

document.monkey // 정상

// any보다 구체적인 단언 사용하기
interface MonkeyDocument extends Document {
  monkey: string;
}

(document as MonkeyDocument).monkey = 'monkey'; // 정상

// 기존 Document의 언어 서비스를 그대로 사용할 수 있음.
// 기존 Document를 바꾸지 않고도 새로운 속성을 추가할 수 있음.
```

- 위의 방법들은 **어쩔 수 없는 경우**에 도입할 수 있다.
- 기본적으로 몽키 패치는 안 하는 게 나으며 더 나은 설계로 리펙토링해야 한다.

## item 44

타입 커버리지를 추적하여 타입 안정성 유지하기

- `noImplicitAny` 속성을 `true`로 설정하고, 에러를 모두 없애도 `any`가 완전히 사라진 것은 아니다.
  - 명시적 `any`는 에러를 발생시키지 않음.
  - 서드파티 타입 선언에서 `any` 타입이 유입될 수 있음.
- 타입 커버리지라는 개념을 사용해 프로젝트 내부의 `any` 타입 비율을 검사할 수 있다.
- `$ yarn add -D type-coverage`
- `$yarn type-coverage`

```sh
> yarn type-coverage
yarn run v1.22.19
$ /Users/hyunochoi/programming/join-research/node_modules/.bin/type-coverage
1557 / 1560 99.80%
type-coverage success.
```

- 프로젝트 내부에 `any`의 비율을 백분율로 보여준다.
- `--detail` 옵션을 추가해 구체적으로 어떤 파일에 `any` 타입이 숨어있는지 볼 수 있다.
