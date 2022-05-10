# 챕터1

## 아이템 1

타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 자바스크립트를 포함한다.
- 이 특징 덕분에 자바스크립트 프로젝트에서 타입스크립트로 변환하는 과정이 무척 편리하다.
- 타입스크립트가 정적 타입 언어처럼 모든 런타임 오류를 방지해주는 것은 아니다.

```typescript
const fruits = ["orange", "kiwi"];
console.log(fruits[2].toUpperCase());

// 위의 코드는 타입 체크를 통과하지만 런타임에 오류가 발생함.
// TypeError: Cannot read properties of undefined (reading 'toUpperCase')
```

- 타입스크립트는 배열을 범위 내에서 사용할 것이라고 전제하기 때문.
- 또한 타입스크립트는 자바스크립트의 런타임 동작을 모방함.

```typescript
console.log(2 + "3"); // 23
```

## 아이템 2

타입스크립트 설정 이해하기

- 타입스크립트 설정은 커멘트라인보다 `tsconfig.json` 파일을 사용하는 것이 좋음.
- 팀 프로젝트라면 타입스크립트 세팅을 공유할 수 있기 때문.
- `noImplicitAny`, `strictNullCheck` 이 두 설정은 반드시 켜놓는 게 좋음.
- `noImplicitAny`는 암시적인 any 타입을 허용하지 않음.

```typescript
const add = (a, b) => a + b;
// Parameter 'a' implicitly has an 'any' type. ❌

const add = (a: number, b: number): number => a + b; ✅
```

- `strictNullCheck`는 null과 undefined가 다른 타입에 들어가는 것을 허용하지 않는 옵션임.
- `strict` 설정을 켜놓으면 앞의 두 설정을 포함하여 엄격한 모드로 타입스크립트를 사용할 수 있음.

## 아이템 3

코드 생성과 타입이 관계없음을 이해하기

- 타입 오류가 있는 코드도 컴파일 할 수 있음.
- 타입스크립트는 타입 체크와 컴파일 과정이 별도로 이루어지기 때문.
- 런타임에는 타입 체크를 할 수 없음.
- 타입스크립트가 자바스크립트로 컴파일 될 때 타입 선언문은 모두 제거되기 때문.
- 런타임 타입과 선언된 타입은 다를 수 있음.

```typescript
const setLight = (value: boolean) => {
  if (value) print("ON");
  else print("OFF");
};

// value가 boolean이라는 것은 타입스크립트 안에서만 유효함.
// 자바스크립트로 컴파일되면 어떤 타입의 값이던 인자로 전달될 수 있음.
```

- 타입스크립트로는 함수 오버로딩을 구현할 수 없음(타입 수준에서만 가능.).
- 타입스크립트의 타입은 컴파일 시 제거되기 때문에 성능과는 무관함.

## 아이템 4

구조적 타이핑에 익숙해지기

- 자바스크립트는 덕 타이핑 기반 언어임(어떤 값이 타입의 요구사항을 충족하기만 하면 해당 타입으로 간주함.).
- 타입스크립트 역시 자바스크립트를 모델링한 구조적 타이핑 방식을 사용함.

```typescript
interface Person {
  name: string;
  gender: string;
}

interface Kim {
  name: string;
  gender: string;
  hobby: string;
}

const kim: Kim = { name: "Kim", gender: "man", hobby: "coding" };
const printName = (person: Person): void => console.log(person.name);
printName(kim);

// 오류 없음.
// Person과 Kim은 다른 인터페이스이고, 둘 간의 관계를 정의하지도 않았습니다.
// 하지만 Kim은 Person이 되기 위한 조건을 모두 갖추었으므로 Person으로 간주합니다.
```

- 이러한 타입스크립트의 특징을 '열린 타입'이라고 함. 다른 정적 타입 언어들은 '닫힌 타입' 언어임.

```typescript
interface Person {
  name: string;
  gender: string;
}

const printProperty = (person: Person): void => {
  Object.keys(person).forEach((key) => console.log(key, person[key]));
};

// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Person'.

// 구조적 타이핑에 익숙하지 않으면 이해하기 어려운 오류.
// 다른 속성과는 무관하게 name과 gender 속성만 가지고 있으면 Person이 될 수 있기 때문에 타입 체커는 person[key]의 타입을 추론할 수 없어 any로 판단했다.
```

## 아이템 5

any 타입 지양하기

- any 타입을 허용하고, 편하다는 이유로 사용할 경우 타입스크립트를 사용하는 의미가 퇴색된다.
- 귀찮더라도 타입을 지정하는 습관을 들이기.
