# 챕터 4

타입 설계

## 아이템 28

유효한 상태만 표현하는 타입을 지향하기

- 타입은 언제나 실제로 가질 수 있는 상태만을 나타내야 함.

```typescript
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}

function renderPage(state: State) {
  if (state.error) {
    // 에러 처리...
    return;
  } else if (state.isLoading) {
    // 로딩중...
    return;
  }
  // 페이지 로딩...
  return;
}
```

- 위의 `renderPage` 함수는 `error` 값이 존재하고, `isLoading`이 `true`일 때 기능이 불명확해짐.
- `State` 상태 자체가 `error`의 존재와 `isLoading: true`를 허용하기 때문.
- 따라서 `State` 타입은 가능한 상태만을 가질 수 있도록 재설계되어야 한다.

```typescript
interface Pending {
  state: "pending";
}

interface Error {
  state: "error";
  error: string;
}

interface Success {
  state: "ok";
  pageText: string;
}

type RequestState = Pending | Error | Success;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}
```

- `State` 설계가 요청 페이지당 하나의 상태만을 가질 수 있도록 변경되었다.
- 로딩 중에 에러나 페이지 텍스트를 가질 수 없으며, 에러 메시지는 오직 에러 상태일 때만 가질 수 있다.
- 이제 페이지 렌더링과 관련된 함수를 작성할 때 `State.requests[page].state` 값을 기준으로 분기하면 각 상태에 대한 작동이 명확해진다.

## 아이템 29

사용할 때는 너그럽게, 생성할 때는 엄격하게

- 매개변수의 타입은 넓은 것이 사용에 편리하다.
- 하지만 반환 타입은 좁은 것이 사용에 편리하다.
- 따라서 넓은 타입의 매개변수와 좁은 타입의 반환값을 도입해야 한다.

```typescript
type LngLat =
  | [number, number]
  | [string, string]
  | { lng: number; lat: number }
  | { lng: string; lat: string };

type Weather = {
  temp?: number | string;
  humidity?: number | string;
  clound?: number | string;
  dustLevel?: number | string;
  UVLevel?: number | string;
};

declare function getWeather(location: LngLat): Weather;
```

- 위도와 경도를 다양한 타입의 매개변수로 받아 해당 지역의 날씨 정보를 반환하는 `getWeather` 함수.
- 위도 경도 정보가 저장된 방식에 구애받지 않고 자유롭게 사용할 수 있다.
- 하지만 반환된 날씨 정보를 사용하기는 굉장히 어렵다.
- 특정 날씨 속성을 필수로 요구하거나, 매개변수 타입이 고정되어 있는 함수에 전달하고자 하는 경우에는 일일히 분기문을 만들거나 아예 사용할 수 없게 된다.

## 아이템 30

문서에 타입 정보를 쓰지 않기

- 함수의 반환값, 매개변수 타입 등은 주석보다 타입 선언으로 작성하는 것이 훨씬 좋다.
- 주석과 구현체는 다를 가능성이 존재하지만 타입 선언과 구현체는 다를 수 없도록 타입 체커가 강제하기 때문.

```typescript
/**
 * 매개변수 a는 string, b는 number 타입입니다.
 * 반환값은 없습니다.
 * 이 함수는 a 문자열을 b만큼 출력해줍니다.
 */

// 위처럼 작성하지 말 것.

type Print = (a: number, b: number) => void;

const print: Print = (a, b) => {
  for (let i = 0; i < b; i++) {
    console.log(a);
  }
};

// 타입 선언으로 주석 대신하기
```

- 변수명에도 역시 타입 정보가 들어갈 필요가 없음.
- 하지만 특정 단위와 관련된 변수라면 단위 정보가 들어갈 수는 있다.

```typescript
const printAge = (ageNum) => ...; // X

const printAge = (age: number) => ...; // O

const temperatureC = 22;
const temperatureF = 71.6;

const timeMS = 1000;
const timeS = 1;
```

## 아이템 31

타입 주변에 `null` 값 배치하기

- 여러 값을 다룰 때, 어떤 값은 `null`이고 다른 값은 `null`이 아닌 경우를 만들기보다는 전부 `null`이거나 아닌 경우를 만드는 것이 낫다.

```typescript
function extend(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num); // number | undefined는 number에 할당할 수 없음.
      // nums가 빈 배열일 수 있으며, max는 별도의 null 체크를 하지 않았기 때문.
    }
  }
  return [min, max];
}
```

- `max`에도 `null` 체크를 적용하면 문제가 해결될까?
- 오히려 더 복잡해진다.
- `max`와 `min`이 제각각 `null`이 될 수 있기 때문에.
- 해결책은 `min`과 `max`를 하나의 변수로 만들고, 둘 모두 `null`이거나 아닌 상태를 만드는 것.

```typescript
function extend(nums: number[]) {
  let result: [number, number] | null = null;
  // min과 max는 전부 값이 존재하거나 null임.
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}
```

- 이제 `extend` 함수의 반환값은 `[number, number] | null`로 훨씬 타입을 예상하기 쉬워짐.
- 사용하는 쪽에서 `null` 체크를 수행하거나 `null` 아님 단언문(`!`)을 사용하면 된다.
- `null`과 `null`이 아닌 값을 섞어 쓰는 것은 클래스에서도 위험하다.

```typescript
class UserPosts {
  user: UserInfo | null;
  posts: Post[];

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    return Promise.all([
      async () => (this.user = await fetchUser(userId)),
      async () => (this.posts = await fetchPostsForUser(userId)),
    ]);
  }

  getUserName() {
    // ...
  }
}
```

- `init` 메소드를 호출하면 시점에 따라서 `user`와 `posts` 값이 각각 `null`일 수도, `null`이 아닐 수도 있다.
- 따라서 해당 속성과 관련된 메소드를 사용하기 어려워진다.
- 여기서는 `user`와 `posts`가 모두 준비된 후에 객체를 반환하는 방식을 사용할 수 있다.

```typescript
class UserPosts {
  user: UserInfo | null;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUsers(userId),
      fetchPostsForUser(uesrId),
    ]);
    return new UserPosts(user, posts);
  }

  getUserName() {
    // ...
    return this.user.name;
  }
}
```

- 이제 `user`와 `posts` 둘 다 `null`이 아닐 때에만 두 값이 준비된 인스턴스를 반환한다.
- 결론적으로 API를 설계할 때 여러 반환값을 하나의 객체로 담아 모두 `null`이거나 그렇지 않게 만들어야 한다.

## 아이템 32

유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 인터페이스가 유니온 속성을 가진다면 이는 인터페이스가 여러 상태를 가질 수 있다는 것을 의미함.
- 따라서 해당 인터페이스를 사용하기 까다로워짐.
- 만약 인터페이스의 속성이 여러 상태를 가져야 한다면 인터페이스를 분리하여 유니온으로 만드는 방식이 더 낫다.

```typescript
interface Layer {
  type: "fill" | "line" | "point";
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}

/*
나쁜 방법.
각각의 상태들이 가질 수 있는 경우의 수를 합하면 27가지나 된다.
*/

interface FillLayer {
  type: "fill";
  layout: FillLayout;
  paint: FillPaint;
}

interface LineLayer {
  type: "line";
  layout: LineLayout;
  paint: LinePaint;
}

interface PointLayer {
  type: "paint";
  layout: PointLayout;
}

type Layer = FillLayer | LineLayer | PointLayer;

/*
좋은 방법.
Layer 타입이 가질 수 있는 상태는 오직 3가지 경우 뿐이며, 3가지 상태 모두 유효하다.
*/
```

## 아이템 33

`string` 타입보다 더 구체적인 타입 사용하기.

- `string` 타입은 문자열로 이루어진 **모든** 값을 담을 수 있다.
- 타입을 더 좁히는 것이 가능할 때는 `string`을 남발하면 안 된다.

```typescript
interface pet {
  type: string; // dog이거나 cat입니다.
  name: string;
}

// 나쁜 설계.

interface pet {
  type: "cat" | "dog";
  name: string;
}

// 올바른 설계. 타입의 범위를 실제 사용 범위와 맞췄다.
```

- 또한 함수에서 매개변수들의 타입을 모두 `string`으로 선언할 경우, 매개변수의 순서가 바뀌어 들어가도 에러가 발생하지 않는 문제가 생긴다.
- 객체로 이루어진 배열에서 한 필드값만 추출하는 `pluck` 함수를 통해 `string` 타입을 좁혀 사용해야 하는 이유를 알 수 있다.

```typescript
type Pluck = (records: any[], key: string) => any[];
// 나쁜 타입 설계.
// 절대로 any 타입을 반환하도록 설계해서는 안 된다.
// key는 records의 키 중 하나여야 하지만 문자열 전체가 타입의 범위로 설정되어 있다.

type Pluck2<T> = (records: T[], key: keyof T) => T[keyof T][];
// 제너릭을 사용하여 any를 쓰지 않았고, key의 범위도 T의 키가 됨.
// 하지만 key는 T의 키 전체가 아니라 키 중 '하나'여야 한다.

type Pluck3<T, K extends keyof T> = (records: T[], key: K) => T[K][];
// 이제 모든 타입이 최대한으로 좁혀졌다.

const pluck: Pluck3<{ text: string }, "text"> = (records, key) =>
  records.map((r) => r[key]);
```

- 또한 `string` 타입을 리터럴의 유니온으로 좁혔을 경우 코드 편집기의 자동완성 기능을 사용할 수 있게 된다.

## 아이템 34

부정확한 타입보다는 미완성 타입을 사용하기.

- 타입 선언은 구체적일수록 좋을까?
- 성급한 구체화는 타입 선언이 없는 것보다 못하다.

```typescript
type Coordinates: number[];
// 숫자 배열 타입은 구체적이지 못하다.

type Coordinates: [number, number];
// 그래서 x, y 축을 표현할 수 있는 튜플 타입으로 수정했다.
// 하지만 위치 정보에 고도가 존재했다면? 혹은 다른 정보가 존재했다면?
```

- 결국 다른 정보가 포함된 위치 데이터를 사용하기 위해서는 타입 단언을 사용해야 한다.
- 타입 구체화가 너무 과했던 예.
- 결론적으로 추상적인 타입은 피해야 한다. 하지만 부정확한 구체화보다는 추상적인 것이 나을 수 있다.

## 아이템 35

데이터가 아닌, API와 명세를 보고 타입 만들기

- 현재 시점에서 가지고 있는 데이터를 기반으로 타입을 만드는 것보다는 API에서 제공하는 타입 명세를 사용하는 것이 훨씬 낫다.
- 그렇게 함으로써 가능한 모든 데이터 범위에 적용될 수 있는 코드를 작성할 수 있다.
- 이벤트 핸들러 등을 작성하는 경우, 혹은 HTML 엘리먼트 타입이 필요한 경우, 직접 타입을 작성하기보다 DOM API 타입 선언을 사용하는 것이 좋다.

## 아이템 36

해당 분야의 용어로 타입 이름 짓기

- 모호한 타입 이름을 붙이면 안 된다.
- 그보다는 수십 년 전부터 그 분야에서 사용되던 용어를 사용하는 것이 훨씬 좋다.
- 해당 타입이 어떤 데이터를 표현하는 것인지 명확해지기 떄문이다.

```typescript
// 동물 DB를 만든다고 가정했을 때,

interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

/**
 * 좋지 못한 방법.
 * name이 의미하는 바는 무엇인가? 일반적인 이름? 학명?
 * endangered는? 무슨 위험에 처한 것인가?
 * habitat은 어떻게 분류해야 하는가?
 *
 * -> 이러한 질문에 대해 해당 인터페이스를 설계한 사람에게 직접 물어봐야 한다.
 **/

interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimate[];
}

/**
 * 좋은 방법
 * 아무 이름이 아니라 해당 분야의 용어를 사용했다.
 * 어떤 뜻인지 모를 때는 인터넷에 검색하면 됨.
 * 타입 정보가 의미하는 바가 타입 안에 모두 들어있다.
 **/
```

- 또한 같은 의미의 타입은 항상 같은 이름으로 써야 한다.

## 아이템 37 공식 명칭에는 상표를 붙이기

- 타입스크립트의 덕 타이핑은 의도치 않은 결과를 기져올 수 있음.

```typescript
interface Person {
  name: string;
  age: number;
}

function printName(person: Person) {
  console.log(person.name);
}

printName({ name: "choi", age: 23 });
// 정상적인 사용

const programmer = { name: "choi", age: 23, language: "JavaScript" };
printName(programmer);
// 작동은 하지만 받고자 하는 매개변수 타입이 아니다.
```

- 이런 덕 타이핑의 단점을 해결할 수 있는 방법으로 **상표 붙이기**가 있다.

```typescript
interface Person {
  _brand: "person";
  name: string;
  age: number;
}

function makePerson(name: string, age: number): Person {
  return { name, age, _brand: "person" };
}

printName(makePerson("choi", 23));
// 정상

const programmer = { name: "choi", age: 23, language: "JavaScript" };
printName(programmer);
// 에러! '_brand' 속성이 없음!
```

- 이러한 상표 붙이기를 활용하면 `number` 자료형에 단위 정보를 더할 수도 있다.

```typescript
type Meters = number & { _brand: "meters" };
// 타입 수준에서만 가능한 선언.
// 실제로 number타입과 {_brand: "meters"} 타입을 동시에 만족하는 값은 존재하지 않는다.
type Seconds = number & { _brand: "seconds" };

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);
// 1000이라는 값과 그것이 meter 단위라는 타입을 동시에 전달할 수 있음.
// 그에 따라 oneKm은 1000m라는 것이 보장됨.
```

- 하지만 이렇게 붙인 단위는 산술 연산 이후 사라지기 때문에 이에 주의해서 사용해야 함.
