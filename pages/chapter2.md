# 챕터 2

## 아이템 6

편집기를 사용하여 타입 시스템 탐색하기

- 타입스크립트는 `tsserver`를 통해 언어 서비스를 제공함.
- 편집기(vscode)를 통해 해당 서비스를 사용하면 매우 편리함.
  - 타입 추론: 변수나 함수명 위에 커서를 올리면 타입스크립트가 추론한 타입을 볼 수 있음.
  - 개발자가 의도한 타입과 추론된 타입이 같은지의 여부를 알 수 있음.
  - `Go to definition`: 누르면 해당 변수나 함수의 타입 선언을 볼 수 있음.

## 아이템 7

타입이 값들의 집합이라고 생각하기

- 타입 = 할당 가능한 값들의 집합
  - 모든 숫자의 집합: `number`
  - 모든 문자열의 집합: `string`
- 타입스크립트에서 타입에 따른 집합의 크기는 다음과 같음.
  1. `never`
     - 공집합.
     - 어떤 타입도 들어갈 수 없음.
  2. `literal`(`unit`)
     - 하나의 값만 들어갈 수 있는 집합.
     - `type A = 'A';`
  3. `union`
     - 멤머 타입들의 합집합.
     - `type NumOrStr = number | string;`
  4. `number`, `string`, ...
     - 크기가 무한대인 집합.
     - 모든 숫자(number)와 모든 문자열(string)을 포함하는 집합이다.
- 타입스크립트의 인터섹션(교집합)
  - 두 타입을 `&` 연산자로 묶어 두 타입의 교집합에 해당하는 타입을 만들 수 있음.
  - 타입스크립트는 덕 타이핑 구조라는 사실을 잊으면 안 됨.

```typescript
interface Person {
  name: string;
  age: number;
}

interface Programmer {
  language: string;
}

type Intersection = Person & Programmer;
// type Intersectin = {
//    name: string;
//    age: number;
//    language: string;
// }
```

- 위의 예시에서 두 타입 간 겹치는 속성이 없기 때문에 교집합 역시 존재하지 않는다고 생각할 수 있음.
- 하지만 타입스크립트의 덕 타이핑 원리에 따르면, 두 타입의 교집합은 두 타입에 명시된 속성을 가지고 있는 모든 타입임.
- 따라서 교집합은 name, age, language 속성을 가진 타입이 된다.
- 두 인터페이스의 유니온에 대해서는 적용될 수 없음.

```typescript
type K = keyof (Person | Programmer);
// type K = never
```

- `A extends B`는 A가 B의 부분집합임을 의미함.
- 따라서 A는 B가 가지는 모든 속성을 가지고 있어야 함.

## 아이템 8

타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트에서 값과 타입은 같은 이름을 가질수도, 여러 의미로 쓰일수도 있기 때문에 둘을 잘 구분해야 함.
  - `const`, `let` 뒤에 나오면 값
  - `type`, `interface` 뒤에 나오면 타입
  - `class`와 `enum`은 값과 타입 둘 다 쓰일 수 있음.
- `typeof` 연산자
  - 타입의 의미로 쓰일 때는 값의 타입을 반환함.
    - `type T = typeof P`
  - 값의 의미로 쓰일 때는 해당 값의 런타임 타입을 나타내는 문자열을 반환함.
    - `const V = typeof P`

```typescript
const p: Person = { name: "Kim", age: 1 };

type T = typeof p; // type T = Person
const V = typeof p; // "object"
```

- 속성 접근자 `[]`는 타입에 접근할 때도 사용할 수 있음.
  - `.`으로 접근하지 말고 타입에서는 언제나 `[]` 사용할 것.
  - 브라켓 안에는 모든 타입을 사용할 수 있음.

```typescript
type Name = Person["name"]; // type Name = string
type All = Person["name" | "age"]; // type Name = string | number
```

- 자바스크립트의 비구조화 할당에 타입을 직접 명시할 수 없음.
  - 비구조화 할당 문법과 겹치기 떄문.
  - 할당문과 타입 선언문을 분리해야 한다.

```typescript
interface Props {
    address: string;
    title: string;
    name: string;
}

function sendEmail({address, title, name}: Props) {
    ...
}
```

## 아이템 9

타입 단언보다는 타입 선언 사용하기

```typescript
const kim: Person = { name: "kim" }; // 선언
const choi = { name: "choi" } as Person; // 단언
```

- 타입 선언은 값이 타입을 만족하는지를 체크하지만 단언은 그렇지 않음.
  - 따라서 잘못된 값이 들어와도 오류가 발생하지 않는다.
- `<Person>{name: "choi}`와 같은 단언문도 사용할 수 있음.
  - TSX 파일에서 컴포넌트와 혼동되기 때문에 잘 사용하지 않는다.
- 타입 단언은 해당 타입을 타입 체커보다 개발자가 잘 알 때 사용한다.
  - DOM Element 같이 Typescript가 접근할 수 없는 값들.
- `!`을 변수 뒤에 붙이는 것 역시 타입 단언문으로, 해당 값이 `null` 타입이 될 수 없음을 단언한다.
- 타입 단언으로 모든 타입을 바꿀 수는 없다.
  - 단언하려는 타입은 기존 타입의 서브 타입이어야 한다.
  - 모든 타입은 `unknown` 타입의 서브 타입이므로 `unknown`으로 변환 후 단언하려는 타입으로 바꿀 수 있다(권장하지 않음.).

## 아이템 10

객체 래퍼 타입 피하기

- 자바스크립트에서는 기본형 타입임에도 객체처럼 속성과 메소드에 접근할 수 있음.
  - 속성에 접근하는 순간 내부적으로 래퍼 객체를 만들고, 동작이 끝나면 만든 객체를 버리기 때문.
  - 따라서 기본형 타입의 속성을 수정해도 원래대로 되돌아가게 됨.
- 타입스크립트에도 래퍼 객체의 타입이 존재함.
  - 기본형 타입에서 첫 글자를 대문자로 쓰면 됨.
  - 예) `string`과 `String`
- 래퍼 타입 대신 기본형 타입을 사용해야 함.

## 아이템 11

잉여 속성 체크의 한계 인지하기

- **잉여 속성 체크**: 타입이 명시된 변수에 객체 리터럴을 할당할 때 해당 타입의 속성이 있는지와 없는 속성이 있는지를 검사하는 과정.
- 타입스크립트는 덕 타이핑을 따르는데, 왜 잉여 속성 체크 과정에서 에러가 발생할까?
  - 할당 가능 검사와 잉여 속성 체크는 별도의 과정이기 때문.
  - 덕 타이핑에 의해 어떤 변수 타입의 부분 집합에 해당하는 값은 모두 할당 가능하다.
  - 하지만 객체 리터럴 할당 시 개발자의 실수를 방지하기 위해 잉여 속성 체크가 작동한다.
- 잉여 속성 체크가 일어날 때
  - 객체 리터럴을 변수에 할당할 때
  - 객체 리터럴을 함수에 매개변수로 전달할 때
- 잉여 속성 체크가 일어나지 않을 때
  - 리터럴이 아닌 객체 변수를 할당할 때
  - 타입 단언문을 사용할 때
- 잉여 속성 체크를 원하지 않는다면 추가될 속성을 인덱스 시그니처로 만들어놓을 수 있음.

```typescript
interface Options = {
  a: string;
  [other: string]: unknown;
}

const Obj: Options = {a: "a", b: "b"} // ok
```

## 아이템 12

함수 표현식에 타입 적용하기

- 자바스크립트 함수는 표현식과 문장 두 방식으로 선언할 수 있음.

```javascript
function fn1(a) {
  return a;
} // 문장
const fn2 = (a) => a; // 표현식
const fn3 = function (a) {
  return a;
}; // 표현식
```

- 타입스크립트에서는 함수 표현식 사용을 권장함.
  - 함수의 매개변수부터 반환값까지를 하나의 타입으로 묶어 전달할 수 있기 때문.
  - 이렇게 타입을 선언하면 가독성이 훨씬 높아진다.
  - 함수 전체의 타입을 사용하면 여러 함수에 재사용이 가능해진다.

## 아이템 13

타입과 인터페이스의 차이점 알기

- 둘 간의 차이를 알기 위해서는 하나의 타입을 두 방식 모두로 작성할 수 있어야 함.
- 공통점
  - 인터페이스와 타입은 상호 간에 확장(`extends`)이 가능하다.
  - 둘 모두 클래스로 구현(`implements`)할 수 있다.
- 차이점
  - 유니온 인터페이스는 존재하지 않음.
  - 튜플의 경우 타입으로 구현하는 것이 좋다.
    - 더 간단할 뿐더러, 튜플 메소드를 사용할 수 있기 때문.

```typescript
interface Tuple {
  0: number;
  1: number;
  length: 2;
}

type Tuple2 = [number, number];
```

- 인터페이스는 보강이 가능하다(선언 병합)

```typescript
interface Person {
  name: string;
}

interface Person {
  age: number;
}

const person: Person = { name: "choi", age: 12 };
```

- 결론
  - 복잡한 타입이 필요하다 => 타입
  - 프로젝트에서 일관되게 사용하는 방식이 있다 => 그 방식
  - 프로젝트 내부적으로만 사용됨 => 타입
  - API 등과 같이 변경될 가능성이 있음 => 인터페이스

## 아이템 14

타입 연산과 제너릭 사용으로 반복 줄이기

- DRY 원칙은 타입에도 적용될 수 있다.
  - 타입에 이름 붙이기
  - 타입 확장(`extends`)하여 사용하기.
  - 인터섹션 연산자(`&`) 사용하기.
  - 매핑된 타입 사용하기

```typescript
type Person = {
  firstName: string;
  lastName: string;
  age: number;
};

type PersonName = {
  [k in "firstName" | "lastName"]: Person[k];
};
// type PersonName = {firstName: string; lastName: string};

type PersonNameWithPick = Pick<Person, "firstName" | "lastName">;
// 유틸 타입 Pick의 역할과 동일함.
```

- 유니온 인덱싱을 통해 유니온의 공통 타입을 유니온으로 가져올 수 있음.
- 이는 공통 타입의 유니온을 바로 타입으로 가져온다는 뜻이며, K에 해당하는 속성만 뽑아오는 `Pick`과는 다른 결과를 가져옴.

```typescript
type Choi = {
  name: "Choi";
};

type Kim = {
  name: "Kim";
};

type Persons = Choi | Kim;
type Names = Persons["name"];
// type Persons = Choi | Kim

type PersonNames = Pick<Persons, "name">;
// type PersonNames = {
//    name: "Choi" | "Kim";
// }
```

- 특정 타입의 변수가 부분적으로 업데이트 되는 함수를 작성할 때, 매핑된 타입을 사용해 타입을 재사용할 수 있음.

```typescript
type Person = {
  firstName: string;
  lastName: string;
  age: number;
  gender: "male" | "female" | "none";
  hobbies: string[];
};

type UpdateOptions = {
  [k in keyof Person]?: Person[k];
};

type UpdateOptions2 = Partial<Person>;
// Partial<T>과 동일한 기능을 수행함.

type update = (person: UpdateOptions) => void;
```

- `keyof`가 타입의 키를 유니온 형태로 반환한다면, `typeof`는 값의 타입을 유니온 형태(혹은 단일 타입)로 반환한다.
- `ReturnType<T>`으로 `T` 함수의 반환 타입을 가져올 수 있음.

```typescript
type Person = {
  firstName: string;
  lastName: string;
};

type isPerson = (person: Person) => boolean;
type user = ReturnType<isPerson>;
// type user = boolean
```

- 제너릭 타입에서 매개변수 제한하기
  - `extends` 키워드를 통해 제너릭 매개변수가 특정 타입의 키임을 선언할 수 있음.

```typescript
type Pick2<T, K> = {
  [k in K]: T[k];
};
// Type 'k' cannot be used to index type 'T'.

type Pick3<T, K extends keyof T> = {
  [k in K]: T[k];
};
```

## 아이템 15

동적 데이터에 인덱스 시그니쳐 사용하기

- **인덱스 시그니쳐**를 사용하여 '유연한' 타입을 정의할 수 있음.

```typescript
type Person = {
  [property: string]: string;
}

const choi: Person = {
  name: "choi";
}

// property: 키의 위치를 표시하기 위함으로, 사용하지 않는 값임.
// 키의 타입: string | number | symbol이어야 한다. 보통은 string을 사용.
// 값의 타입: any
```

- 인덱스 시그니쳐를 사용해서 타입을 정의하면 네 가지 단점이 드러난다.
  1. 모든 키를 허용하게 됨.
  2. 필수로 포함되어야 하는 키를 지정할 수 없음.
  3. 키마다 다른 타입을 적용할 수 없음.
  4. 1번에 따라 타입스크립트의 자동 완성 서비스가 동작하지 않음.
- 따라서 인덱스 시그니쳐는 동적 데이터에 대한 타입을 정의할 때 쓰도록 하고, 어떤 키가 필요한지 알고 있다면 인터페이스로 작성하는 것이 더 좋다.
- 만약 알고 있는 키 정보가 불확실하다면 인덱스 시그니쳐 외에도 다른 정의 방법이 존재한다.

```typescript
interface Person {
  [property: string]: string;
}

// 가장 광범위한 타입

interface Person {
  name: string;
  age?: number;
  hobby?: string;
}

// 최선

type Person =
  | { name: string }
  | { name: string; age: number }
  | { name: string; age: number; hobby: string };

// 가장 정확하지만 사용하기 번거로움.
```

```typescript
interface Person {
  name: string;
  age: number;
}
```

## 아이템 16

`number` 인덱스 시그니처보다는 `Array`, 튜플, `ArrayLike`를 사용하기

- 자바스크립트 객체는 키/값 쌍을 가짐.
- 오직 문자열만 객체의 키로 쓸 수 있음(**ES2015** 이후로 심볼도 가능).
- 문자열이 아닌 다른 자료형을 키로 쓰려고 하면 자바스크립트는 자동으로 `toString` 메소드를 호출하여 문자열로 강제 변환함.

```javascript
const obj = {};
obj[0] = "a";

obj; // { '0': 'a'}
```

- 심지어 `Array`의 경우도 숫자 인덱스를 문자열로 변환하여 사용함.

```javascript
const arr = [1, 2, 3];

Object.keys(arr); // ['0', '1', '2']
```

- 타입스크립트에서는 객체의 키로 `number` 자료형을 허용함.
- 하지만 `Object.keys()` 메소드로 키를 추출한 반환값은 여전히 `string[]`임.
- 실용적인 목적을 위해서임.
- 일반적으로 `number` 키를 사용할 일은 거의 없으며, 차라리 배열, 튜플, `ArrayLike` 등을 사용하는 게 나음.

## 아이템 17

변경 관련 오류를 방지하기 위해 `readonly` 사용하기

- 자바스크립트에서는 `const` 키워드로 상수를 선언할 수 있음.
- `const`로 선언한 변수는 재할당이 불가능하지만, 변경은 가능함.
- `const`로 선언한 배열에 새로운 요소를 추가하는 것이 가능한 이유.
- 따라서 변경되면 안 되는 객체나 배열이 있다면 `readonly` 키워드를 붙일 수 있음.

```typescript
const arr: readonly number[] = [1, 2, 3];
```

- `readonly` 타입은 속성을 읽을수만 있으며, 속성을 변경하는 메소드가 모두 제거되어 있음.
- 원래 타입의 서브타입임.
- 특히 함수의 매개변수가 변경되지 않는다는 제한을 걸 때 사용하면 편하다.

## 아이템 18

매핑된 타입을 사용하여 값을 동기화하기

- 산점도를 그리는 컴포넌트에서의 리렌더링 예시.
- 화면상에 표시되는 점들의 데이터는 다시 그려야 하지만, 이벤트 핸들러 함수는 다시 그릴 필요가 없다.

```typescript
interface ScatterProps {
  // data
  xs: number[];
  ys: number[];

  // display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // event handler
  onClick: (x: number, y: number, index: number) => void;
}
```

- 첫 번째 방법은 실패에 닫힌(fail close) 방법 혹은 보수적 접근법이다.

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

- 실패에 닫힌 접근법을 사용하면 `ScatterProps`에 새로운 속성이 모두 업데이트 대상이 된다.
- 하지만 컴포넌트가 너무 자주 업데이트 될 수 있다.
- 두 번째 방법은 실패에 열린(fail open) 방법이다.

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // onClick 속성은 체크하지 않는다.
  );
}
```

- 실패에 열린 접근법을 쓰면 새로운 속성에 대해 업데이트가 수행되지 않는다.
- 하지만 새로운 속성이 변할 때마다 업데이트를 필요로 하는 속성이라면 문제가 발생할 수 있다.
- 혹은, 새로운 속성을 추가할 때마다 `shouldUpdate` 함수를 고치라고 주석을 작성해놓을 수도 있다.

```typescript
interface ScatterProps {
  // data
  xs: number[];
  ys: number[];

  // display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // event handler
  onClick: (x: number, y: number, index: number) => void;

  // 새로운 속성을 추가할 때는 shouldUpdate 함수도 같이 고치세요!
```

- 가장 나은 방법은 타입 체커가 이 역할을 대신하게 하는 것이다.

```typescript
const REQUIRES_UPDATE: {[k in keyof ScatterProps: boolean]} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE) {
      return true;
    }
  }
  return false;
}
```

- 여기에서 `REQUIRES_UPDATE`가 `ScatterProps`의 키를 그대로 사용한다는 사실을 명시해주었기 때문에 만약 `ScatterProps`에 새로운 속성이 추가된다면 `REQUIRES_UPDATE`에서 타입 체커가 오류를 발생시켜 개발자에게 `REQUIRES_UPDATE`에도 새로운 속성을 추가해야한다는 사실을 알려주게 된다.
- 이러한 방법은 바뀌는 속성에 대해 선택적 업데이트를 적용함과 동시에, 새로운 속성이 추가되었을 때 그 속성이 업데이트를 필요로 하는지를 개발자가 명시해주어야 한다는 사실을 타입 체커를 통해 알려줄 수 있다.
