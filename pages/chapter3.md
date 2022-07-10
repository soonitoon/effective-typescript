# 챕터 3

## 아이템 19

추론 가능한 타입을 사용해 장황한 코드 방지하기.

- 타입스크립트가 충분히 추론할 수 있고, 타입을 명시할 이유가 없는 변수에 대해 타입을 명시하는 것은 가독성을 해친다.
- 타입스크립트는 복잡한 객체, 함수의 반환값 등에 대해 뛰어나게 추론함.
- 타입 추론은 리펙터링 시에도 유용하다.

```typescript
interface Person {
  name: string;
  age: number;
}

function printPerson(person: Person) {
  const name: string = person.name;
  const age: number = person.age;

  console.log(name, age);
}
```

- 위의 코드에서 갑자기 `Person`의 `age`에 `~세`를 붙여 `string` 형식으로 변경하고자 할 경우, `Person` 타입과 `printPerson` 함수의 지역 변수 타입을 모두 바꿔야 한다.
- 이럴 때 구조 분해 할당을 사용하면 지역 변수에 타입 추론이 적용된다.

```typescript
function printPerson(person: Person) {
  const { name, age } = person;

  console.log(name, age);
}
```

- 이렇게 코드를 작성하면 추후 `Person` 타입이 변경됨에 따라 `printPerson` 내 지역 변수의 타입도 저절로 변경된다.

- 함수의 매개변수에 기본값이 할당되어 있을 경우, 타입이 추론된다.
- 타입 정보가 포함된 라이브러리에서는 콜백 함수의 매개변수 타입을 명시할 필요가 없다.
- 객체 리터럴 선언에서 타입 추론은 **잉여 속성 체크** 기능을 원한다면 필요하다.
- (타입 체커가 추론할 수 있음에도) 함수의 반환 타입을 작성하는 것은 좋다.
  - 구현상의 오류가 사용상의 오류로 표시되지 않는다.
  - 함수의 구조를 명확하게 나타낼 수 있다.
  - 반환 타입으로 명명된 타입을 사용함으로써 함수 사용자가 반환 타입을 이해하기 쉬워진다.

## 아이템 20

다른 타입에는 다른 변수 사용하기

- 자바스크립트에서는 하나의 변수에 여러 타입을 담아 사용할 수 있지만, 타입스크립트에서 변수의 타입을 일반적으로 바뀌지 않는다.
- 다른 타입을 사용해야 하는 상황에서는 하나의 변수를 유니온 타입으로 묶기보다는 그냥 다른 변수 하나를 더 만드는 것이 좋다.
- 변수 스코프를 이용한 가려지는(shadowed) 변수도 혼란을 줄 수 있으며, 대부분의 린터는 사용을 금지한다.

## 아이템 21

타입 넓히기

- 타입스크립트는 타입 선언이 없는 변수의 타입을 어떻게 추론할까?

```typescript
let a = "a"; // string
const b = "b"; // "b"
```

- `let` 키워드로 선언된 변수는 재할당이 가능하므로 상대적으로 넓은 범위의 타입이 적용됨.
- `const` 키워드로 선언된 변수는 재할당 할 수 없으므로 상대적으로 좁은 범위의 타입이 적용됨.
- 객체의 경우 각 속성을 `let`으로 선언된 것처럼 다룸.
- 또한 객체에 새로운 속성을 추가할 수는 없다.
- 타입 체커가 판단하는 타입의 범위를 어떻게 조정할 수 있을까?
  - 타입 선언문 제공하기
  - 문맥 제공하기(뒤에서 다룸)
  - `const` 단언문 사용하기(선언 키워드 `const`와는 다른, 순수한 타입 공간에서의 의미임)

```typescript
const obj = {
  a: 1,
  b: "b",
};

// type obj = {a: numnber, b: string}

const obj: { a: 1; b: "b" } = {
  a: 1,
  b: "b",
};

// type obj = {a: 1, b: "b"}

const obj = {
  a: 1 as const,
  b: "b",
};

// type obj = {a: 1, b: string}

const obj = {
  a: 1,
  b: "b",
} as const;

// type obj = {readonly a: 1, readonly b: "b"}
```

## 아이템 22

타입 좁히기

- 타입스크립트의 타입 추론 결과의 범위를 좁히는 과정.
- `null` 체크가 대표적인 예시임.

```typescript
const button = document.getElementById("submit-button");
// type button = HTMLElement | null

if (button) {
  button.textContent = "submit";
  // type button = HTMLElement
} else {
  console.log("No Button!");
  // type button = null
}
```

- 위와 같은 처리는 얼리-리턴문에도 똑같이 적용됨.
- 타입 좁히기를 속성 체크로 구현할 수 있음.

```typescript
interface A {
  a: "a";
}

interface B {
  b: "b";
}

const pick = (ab: A | B) => {
  if ("a" in ab) {
    // type ab = A
    return;
  }
  if ("b" in ab) {
    // type ab = B
    return;
  }
  ab;
  // type ab = A | B
};
```

- 이러한 타입 좁히기는 내장 함수 `isArray` 등을 통해 구현할 수도 있음.
- 또한 타입 좁히기는 타입 태그를 통해 구현할 수도 있음

```typescript
interface A {
  type: "typeA";
  a: "a";
}

interface B {
  type: "typeB";
  b: "b";
}

const pick = (ab: A | B) => {
  if (ab.type === "typeA") {
    // type ab = A
    return;
  }
  if (ab.type === "typeB") {
    // type ab = B
    return;
  }
  ab;
  // type ab = A | B
};
```

- 마지막으로, **'사용자 정의 타입 가드'** 로 타입을 좁힐 수 있음.

```typescript
const isA = (ab: A | B): ab is A => {
  return "a" in ab;
};

const pick = (ab: A | B) => {
  if (isA(ab)) {
    // type ab = A
    return;
  }
  ab
  // type ab = B
  }
};
```

## 아이템 23

한꺼번에 객체 생성하기

- 타입스크립트에서 변수의 값은 변할 수 있지만 타입은 바뀌지 않는다.
- 타입스크립트는 객체가 만들어질 때(타입을 명시하지 않으면) 객체의 타입을 함께 추론하므로 한 번 생성한 객체에 새로운 속성을 추가할 수는 없다.
- 이미 처음 객체에 맞게 타입 정보가 세팅되었기 때문이다.
- 객체를 반드시 나눠서 만들어야 한다면 **타입 단언문**을 사용할 수 있다.

```typescript
interface Person {
  name: string;
  age: number;
  gender: "male" | "female";
}

const choi = {} as Person;

choi.name = "Hyuno Choi";
choi.age = 22;
choi.gender = "male";

// 타입 오류 없음
```

- 작은 객체를 합쳐서 큰 객체를 만들 때는 **전개 연산자**를 사용해서 작은 객체를 분해한 다음, 새로운 변수에 큰 객체를 할당하면 된다.

```typescript
const nameOfChoi = { name: "choi" };
const ageAndGenderOfChoi = { age: 22, gender: "male" };

const choi: Person = { ...nameOfChoi, ...ageAndGenderOfChoi };
// 타입 오류 없음
```

- 전개연산자와 삼항연산자를 응용해서 객체에 조건부 속성을 추가할 수 있음.

```typescript
declare let hasCat: boolean;

const choiWithCat = {
  ...choi,
  ...(hasCat ? { catName: "shiro", catColor: "white" } : {}),
};

// type choiWithCat = {
//    name: string;
//    age: number;
//    gender: "male" | "female";
//    catName?: string | undefined;
//    catColor?: string | undefined;
// }
```

## 아이템 24

일관성 있는 별칭 사용하기

- 객체의 특정 속성을 반복하여 참조하는 것을 방지하고자 변수에 담아 사용할 수 있음.
- 이때 변수명이 원래의 속성명과 계속해서 달라지면 디버깅에 혼란을 일으킴.
- 따라서 별칭은 원래 속성명과 같게 사용하는 것이 좋고, 최선의 선택은 비구조화 할당임.

```typescript
const Person = {
  name: "Choi",
  age: 22,
};

const printName = (person: Person) => {
  const nameOfPerson = person.name;
  console.log(nameOfPerson);
};

const printName2 = (person: Person) => {
  const { name } = person;
  console.log(name);
};

// printName보다 PrintName2 방식이 더 좋다. 👍
```

## 아이템 25

비동기 코드에는 콜백 대신 `async` 함수 사용하기

- 콜백 지옥을 2022년에 재현하지 말기...
- (특히 타입스크립트애서는) `Promise` 보다 `async` 함수가 더 나은 선택임.
  - 동기 코드와 같은 형식으로 작성할 수 있어 가독성 면에서 유리하다.
  - 마찬가지로 동기 코드에서 사용하는 `try/catch` 구문으로 오류 처리가 가능하다.
  - `async` 함수는 `Promise`를 반환하도록 강제되기 때문에 실수로 동기와 비동기 처리를 섞어 사용하는 것을 예방할 수 있다.

```typescript
const fetchWithCache = (url, callback) => {
  // 동기
  if (url in cache) {
    callback(cache);
    return;
  }
  // 비동기
  fetchURL(url, (text) => {
    cache[url] = text;
    callback(text);
  });
};
```

## 아이템 26

타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입스크립트는 값과 함께 문맥을 사용해 타입을 추론하기 때문에 문맥에서 값으 분리하면 타입 추론이 부정확해질 수 있음.

```typescript
type PrintLanguage = (language: "JavaScript" | "TypeScript" | "Python") => void;

const printLanguage: PrintLanguage = (language) => console.log(language);

let language = "JavaScript";

printLanguage("JavaScript");
// ok

printLanguage(language);
// 에러: string 타입을 매개변수에 할당할 수 없음!
```

- 위의 예제에서 `language` 변수의 추론된 타입은 `string`임.
  - `language` 변수는 문맥과 분리된 값이므로, 타입스크립트는 임의로 타입을 추론했음.
  - 더불어 별도의 타입 선언도 없었음.
  - 따라서 타입스크립트는 스스로 판단하기에 적절한 범위를 가지는 타입으로 추론한다.
- 따라서 위의 오류를 잡기 위해서는 `const`를 사용해 해당값이 상수라고 타입 체커에게 알려주면 된다.
- 혹은 타입 선언을 붙일수도 있음.

```typescript
const language = "JavaScript";
printLanguage(language);
// ok

type Language = "JavaScript" | "TypeScript" | "Python";
let language: Language = "JavaScript";
printLanguage(language);
// ok
```

- 튜플 사용시 문맥을 유의해야 함.

```typescript
const printCoordinate = (coordinate: [number, number]) =>
  console.log(coordinate);

const coordinate = [1, 2];
printCoordinate(coordinate);
// 에러: number[]를 [number, number] 매개변수에 할당할 수 없음!
```

- 변수 `coordinate`가 문맥과 분리되어 있으므로 튜플이라고 추론하지 못함.
- `const`로 선언해도 배열에 요소를 추가하고 삭제할 수 있으므로 `number[]` 추론은 타당함.
- `as const`를 생각해볼 수 있음.

```typescript
const coordinate = [1, 2] as const;
printCoordinate(coordinate);
// 에러: readonly [1, 2]는 [number, number] 매개변수에 할당할 수 없음!
```

- 이번에는 타입 추론의 범위가 과하게 좁아졌음.
- 이 상황에서 최선의 해결책은 `printCoordinate` 함수의 매개변수 타입도 `readonly`로 만드는 것.

```typescript
const printCoordinate = (coordinate: readonly [number, number]) =>
  console.log(coordinate);

const coordinate = [1, 2] as const;
printCoordinate(coordinate);
// ok
```

## 아이템 27

함수형 기법과 라이브러리로 타입 흐름 유지하기

- 자바스크립트의 루프를 대체할 수 있는 라이브러리는 타입스크립트와 함께 사용하면 더욱 좋다.
- 타입 정보를 그대로 유지하면서 흐름을 이어갈 수 있기 때문.
- csv 파일의 데이터를 추출하는 예제에서 순수 자바스크립트로 루프를 구현하면 다음과 같다.

```typescript
const csvData = "id,name,age,gender\n0,choi,22,M\n1,kim,22,F";

const rawRows = csvData.split("\n");
const headers = rawRows[0].split(",");

const rows = rawRows.slice(1).map((rowStr) => {
  const row = {};
  rowStr.split(",").forEach((val, j) => {
    row[headers[j]] = val;
  });
  return row;
});
```

- `reduce` 메소드를 사용하면 코드 길이를 줄일 수 있음.

```typescript
const rows = rawRows
  .slice(1)
  .map((rowStr) =>
    rowStr
      .split(",")
      .reduce((row, val, i) => ((row[headers[i]] = val), row), {})
  );
```

- 하지만 직관적으로 이해하기 어렵다.
- 무엇보다 두 코드 모두 빈 객체 `{}`에 새로우 속성을 추가할 때 타입 에러가 발생하므로, 잉여 타입이 들어갈 여지를 만들어줘야 한다.
- 여기서 **로대시**의 `zipObject` 함수를 사용하면 직관성과 타입 흐름을 모두 잡을 수 있다.

```typescript
const rows = rawRows
  .slice(1)
  .map((rowStr) => _.zipObject(headers, rowStr.split(",")));

console.log(rows);

// type rows = _.Dictionary<string>[]
```

- `Dictionary<string>[]` 타입은 `{[Key: string]: string}[]`과 동일하다.
- 즉, 별도의 타입 선언을 하지 않아도 라이브러리를 사용함으로써 타입 흐름을 유지할 수 있게 되었다.
- 이러한 라이브러리의 타입 흐름 유지는 데이터를 가공해야 하는 상황에서 더 유리하다.

```typescript
interface BasketballPlayer {
  name: string;
  team: string;
  salary: number;
}

declare const rosters: { [team: string]: BasketballPlayer[] };

let allPlayers = [];
// 에러! allPlayers는 any가 될 수도 있음.

for (const players of Object.values(rosters)) {
  allPlayers = allPlayers.concat(players);
}
```

- `BasketballPlayer` 전체 리스트는 만들어야 하는 상황.
- 물론 `allPlayers`를 선언할 때 타입 선언을 붙일 수는 있지만 `flat` 메소드를 사용하면 타입 흐름을 사용할 수 있다.

```typescript
const allPlayers = Object.values(rosters).flat();
// type allPlayers = BasketballPlayer[]
```

- 타입 선언을 쓰지 않고 간단하게 1차원 배열로 변환할 수 있다.
- 위의 `allPlayers`에서 팀별 최고 연봉을 받는 선수를 오름차순으로 정렬한 배열을 얻고 싶다면?
- 순수 자바스크립트로 구현하면 다음과 같다.

```typescript
const teamToPlayers: { [team: string]: BasketballPlayer[] } = {};

for (const player of allPlayers) {
  const { team } = player;
  teamToPlayers[team] = teamToPlayers[team] || [];
  teamToPlayers[team].push(player);
}

for (const players of Object.values(teamToPlayers)) {
  players.sort((a, b) => b.salary - a.salary);
}

const bestPaid = Object.values(teamToPlayers).map((players) => players[0]);
bestPaid.sort((playerA, playerB) => playerB.salary - playerA.salary);
console.log(bestPaid);
```

- 코드가 지나치게 길고, 타입 흐름을 벗어난 부분에 타입 선언도 달아줘야 한다.
- 같은 기능을 **lodash**의 `maxBy` 함수로 구현하면 다음과 같다.

```typescript
const bestPaid = _(allPlayers)
  .groupBy((player) => player.team)
  .mapValues((players) => _.maxBy(players, (p) => p.salary)!)
  .values()
  .sortBy((p) => -p.salary)
  .value();
console.log(bestPaid);
```

- `null` 아님 단언문을 한 번만 사용해서 완벽한 체인으로 구현함.
- `_()`로 로대쉬 메소드를 쓸 수 있게 래핑한 다음, 마지막에 `.value()` 메소드로 언래핑된 값을 반환받는다.
- 결론적으로, 이러한 라이브러리들의 타입 유지 기능을 잘 활용할수록 타입스크립트의 의도와 일치하게 된다.
