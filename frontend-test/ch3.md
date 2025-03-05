# Chapter 3. 처음 시작하는 단위 테스트

## 1. 환경설정

- Jest를 사용
    - 목 객체와 코드 커버리지 수집 기능까지 갖춘 메타의 오픈소스

## 2. 테스트 구성요소

### 2.1 가장 단순한 테스트

- 테스트는 구현 파일에 작성하지 않고 별도의 파일을 만들어 테스트 대상을 import로 불러와서 테스트한다.

### 2.2 테스트 구성 요소

```jsx
test(테스트명, 테스트 함수);

// 테스트 명에는 테스트 내용
// 테스트 함수에는 단언문
// 단언문은 함수 + 매처

test("1 + 2는 3", () => {
	expect(검증값).toBe(기댓값);
}
```

### 2.3 테스트 그룹 작성

- 연관성 있는 테스트들을 그룹화하고 싶을 때는 `describe` 함수를 사용
- `test` 함수는 중첩시킬 수 없지만 `describe` 함수는 중첩이 가능

```jsx
describe("사칙연산", () => {
  describe("add", () => {
    test("1 + 1은 2", () => {
      expect(add(1, 1)).toBe(2);
    });
    test("1 + 2는 3", () => {
      expect(add(1, 2)).toBe(3);
    });
  });
  describe("sub", () => {
    test("1 - 1은 0", () => {
      expect(sub(1, 1)).toBe(0);
    });
    test("2 - 1은 1", () => {
      expect(sub(2, 1)).toBe(1);
    });
  });
});
```

## 3. 테스트 실행 방법

크게 두 가지

- 명령줄 인터페이스로 실행
- 제스트 러너로 실행

### …

## 4. 조건 분기

- 사양이 복잡할수록 조건 분기에서 버그가 많이 생긴다. 특별히 주의하면서 작성해야!

### 4.1 상한이 있는 덧셈 함수 테스트

### 4.2 하한이 있는 뺄셈 함수 테스트

```jsx
// index.ts
export function sub(a: number, b: number) {
  const sum = a - b;
  if (sum < 0) {
    return 0;
  }
  return sum;
}

// index.test.ts
test("반환값은 첫 번째 매개변수에서 두 번째 매개변수를 뺀 값이다", () => {
  expect(sub(51, 50)).toBe(1);
});
test("반환값의 하한은 '0'이다", () => {
  expect(sub(70, 80)).toBe(0);
});
```

## 5. 에지 케이스와 예외 처리

- 모듈에 예외 처리를 했다면 예상하지 못한 입력값을 받았을 때 실행 중인 디버거로 문제를 빨리 해결 할 수있다

### 5.1 타입스크립트로 입력값 제약 설정

### 5.2 예외 발생시키기

### 5.3 예외 발생 검증 테스트

```jsx
expect(예외가 발생하는 함수).toThrow();
```

### 5.4 오류 메세지를 활용한 세부 사항 검증

- 의도한 대로 예외가 발생하고 있는가라는 관점으로 접근하자

### 5.5 instanceof 연산자를 활용한 세부 사항 검증

- `Error` 클래스를 더욱 구체적인 상황에 맞춰 작성해보자

```jsx
export class HttpError extends Error {}
export class RangeError extends Error {}

if (err instanceof HttpError) {
  // 발생한 오류가 HttpError인 경우
}
if (err instanceof RangeError) {
  // 발생한 오류가 RangeError인 경우
}
```

```jsx
function checkRange(value: number) {
  if (value < 0 || value > 100) {
    throw new RangeError("0〜100 사이의 값을 입력해주세요");
  }
}

export function add(a: number, b: number) {
  checkRange(a);
  checkRange(b);
  const sum = a + b;
  if (sum > 100) {
    return 100;
  }
  return sum;
}

export function sub(a: number, b: number) {
  checkRange(a);
  checkRange(b);
  const sum = a - b;
  if (sum < 0) {
    return 0;
  }
  return sum;
}
```

```jsx
describe("사칙연산", () => {
  describe("add", () => {
    test("반환값은 첫 번째 매개변수와 두 번째 매개변수를 더한 값이다", () => {
      expect(add(50, 50)).toBe(100);
    });
    test("반환값의 상한은 '100'이다", () => {
      expect(add(70, 80)).toBe(100);
    });
    test("인수가 '0~100'의 범위밖이면 예외가 발생한다", () => {
      const message = "0〜100 사이의 값을 입력해주세요";
      expect(() => add(-10, 10)).toThrow(message);
      expect(() => add(10, -10)).toThrow(message);
      expect(() => add(-10, 110)).toThrow(message);
    });
  });
  describe("sub", () => {
    test("반환값은 첫 번째 매개변수에서 두 번째 매개변수를 뺀 값이다", () => {
      expect(sub(51, 50)).toBe(1);
    });
    test("반환값의 하한은 '0'이다", () => {
      expect(sub(70, 80)).toBe(0);
    });
    test("인수가 '0~100'의 범위밖이면 예외가 발생한다", () => {
      expect(() => sub(-10, 10)).toThrow(RangeError);
      expect(() => sub(10, -10)).toThrow(RangeError);
      expect(() => sub(-10, 110)).toThrow(Error);
    });
  });
});
```

## 6. 용도별 매처

- 단언문은 테스트 대상이 기댓값과 일치하는지 매처로 검증한다

### 6.1 진릿값 검증

- `toBeTruthy`, `toBeFalsy`
- `not` 으로 반전 가능
- `null` 이나 `undefined` 도 `toBeFalsy`와 일치
    - 하지만 정확히 검증하고 싶다면 `toBeNull`, `toBeUndefined`

### 6.2 수치 검증

- 등가 비교나 대소 비교 관련 매처 사용
- toBe, toEqual, toBeGreaterThan, toBeGreaterThanOrEqual, toBeLessThan, toBeLessThanOrEqual
- 자바스크립트는 소수 계산에 오차가 있다, 10진수인 소수를 2진수로 변환할 때 발생하는 문제다.
    - 라이브러리를 사용하지 않고 계산한 소숫값을 검증할 때는 toBeCloseTo 매처를 사용

### 6.3 문자열 검증

- toContain, toMatch. toHaveLength
- stringContaining, stringMatching 은 객체에 포함된 문자열을 검증할 때 사용

```jsx
describe("문자열 검증", () => {
  const str = "Hello World";
  const obj = { status: 200, message: str };
  test("검증값이 기댓값과 일치한다", () => {
    expect(str).toBe("Hello World");
    expect(str).toEqual("Hello World");
  });
  test("toContain", () => {
    expect(str).toContain("World");
    expect(str).not.toContain("Bye");
  });
  test("toMatch", () => {
    expect(str).toMatch(/World/);
    expect(str).not.toMatch(/Bye/);
  });
  test("toHaveLength", () => {
    expect(str).toHaveLength(11);
    expect(str).not.toHaveLength(12);
  });
  test("stringContaining", () => {
    expect(obj).toEqual({
      status: 200,
      message: expect.stringContaining("World"),
    });
  });
  test("stringMatching", () => {
    expect(obj).toEqual({
      status: 200,
      message: expect.stringMatching(/World/),
    });
  });
});
```

### 6.4 배열 검증

- 배열에 원시형(primitive type) 확인 - toContain
- 배열 길이 - toHaveLength
- 배열에 특정 객체 - toContainEqual
- arrayContaining - 인수로 넘겨준 배열의 요소들이 전부 포함돼 있어야

```jsx
  describe("객체들로 구성된 배열", () => {
    const article1 = { author: "taro", title: "Testing Next.js" };
    const article2 = { author: "jiro", title: "Storybook play function" };
    const article3 = { author: "hanako", title: "Visual Regression Testing" };
    const articles = [article1, article2, article3];
    test("toContainEqual", () => {
      expect(articles).toContainEqual(article1);
    });
    test("arrayContaining", () => {
      expect(articles).toEqual(expect.arrayContaining([article1, article3]));
    });
  });
```

### 6.5 객체 검증

- toBeMatchObject, toHaveProperty
- objectContaining

```jsx
describe("객체 검증", () => {
  const author = { name: "taroyamada", age: 38 };
  const article = {
    title: "Testing with Jest",
    author,
  };
  test("toMatchObject", () => {
    expect(author).toMatchObject({ name: "taroyamada", age: 38 });
    expect(author).toMatchObject({ name: "taroyamada" });
    expect(author).not.toMatchObject({ gender: "man" });
  });
  test("toHaveProperty", () => {
    expect(author).toHaveProperty("name");
    expect(author).toHaveProperty("age");
  });
  test("objectContaining", () => {
    expect(article).toEqual({
      title: "Testing with Jest",
      author: expect.objectContaining({ name: "taroyamada" }),
    });
    expect(article).toEqual({
      title: "Testing with Jest",
      author: expect.not.objectContaining({ gender: "man" }),
    });
  });
});
```

## 7. 비동기 처리 테스트

### 7.1 테스트할 함수

```jsx
export function wait(duration: number) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(duration);
    }, duration);
  });
}
```

### 7.2 Promise를 반환하는 작성법

- Promise를 반환하면서 then에 전달할 함수에 단언문을 작성하는 방법
- resolves 매처를 사용하는 단언문을 리턴하는 방법
    - resolve 됐을 때의 값을 검증하고 싶다면 첫번째보다 간편

### 7.3 async/await를 활용한 작성법

- 테스트 함수를 async 함수로 만들고 함수 내에서 Promise가 완료될 때까지 기다리는 방법
    - resolves 매처를 사용하는 단언문도 await로 대기시킬 수 있다
- 검증값인 Promise가 완료되는 것을 기다린 후 단언문을 실행하는 방법(가장 간단)

```jsx
  describe("wait", () => {
    test("지정 시간을 기다린 뒤 경과 시간과 함께 resolve된다", () => {
      return wait(50).then((duration) => {
        expect(duration).toBe(50);
      });
    });
    test("지정 시간을 기다린 뒤 경과 시간과 함께 resolve된다", () => {
      return expect(wait(50)).resolves.toBe(50);
    });
    test("지정 시간을 기다린 뒤 경과 시간과 함께 resolve된다", async () => {
      await expect(wait(50)).resolves.toBe(50);
    });
    test("지정 시간을 기다린 뒤 경과 시간과 함께 resolve된다", async () => {
      expect(await wait(50)).toBe(50);
    });
  });
```

### 7.4 Reject 검증 테스트

- 반드시 reject 되는 경우를 검증

```jsx
export function timeout(duration: number) {
  return new Promise((_, reject) => {
    setTimeout(() => {
      reject(duration);
    }, duration);
  });
}
```

```jsx
  describe("timeout", () => {
	  **// 1.Promise를 return 하는 방법, catch 메서드에 전달할 함수에 단언문 작성**
    test("지정 시간을 기다린 뒤 경과 시간과 함께 reject된다", () => {
      return timeout(50).catch((duration) => {
        expect(duration).toBe(50);
      });
    });
    **// 2.rejects 매처를 사용하는 단언문을 활용하는 방법, 단언문을 리턴하거나 async/await를 사용한다**
    test("지정 시간을 기다린 뒤 경과 시간과 함께 reject된다", () => {
      return expect(timeout(50)).rejects.toBe(50);
    });
    test("지정 시간을 기다린 뒤 경과 시간과 함께 reject된다", async () => {
      await expect(timeout(50)).rejects.toBe(50);
    });
  });
  
  // 3.try-catch 문을 사용하는 방법
  test("지정 시간을 기다린 뒤 경과 시간과 함께 reject된다", async () => {
  expect.assertions(1);
  try {
    await timeout(50);
  } catch (err) {
    expect(err).toBe(50);
  }
});
```

### 7.5 테스트 결과가 기댓값과 일치하는지 확인하기

- 비동기 처리를 테스트할 때 테스트 함수가 동기 함수라면 반드시 return해야 한다.

```jsx
test("return하고 있지 않으므로 Promise가 완료되기 전에 테스트가 종료된다", () => {
  // 실패할 것을 기대하고 작성한 단언문
  expect(wait(2000)).resolves.toBe(3000);
  // 올바르게 고치려면 다음 주석처럼 단언문을 return해야 한다
  // return expect(wait(2000)).resolves.toBe(3000);
});
```

- 비동기 처리 테스트 작성 시 주의할 것
    1. 비동기 처리가 포함된 부분을 테스트할 때는 테스트 함수를 async 함수로 만든다.
    2. .resolves나 .rejects가 포함된 단언문은 await한다.
    3. try-catch 문의 예외 발생을 검증할 때는 expect.assertions를 사용한다.