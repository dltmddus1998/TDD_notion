# 유닛테스트

## 용어 정리

---

**Test Runner**

> 테스트 실행 후 결과 생성
ex) Mocha
> 

**Assertion**

> 테스트 조건, 비교를 통한 테스트 로직 
ex) Chai, expect.js, better-assert
> 

⭐️ **Jest**

> 간편하게 테스트 코드를 작성할 수 있는 라이브러리
> 

✔︎ **Jest is a delightful JavaScript Testing Framework with a focus on simplicity.
: JavaScript 환경에서 간편하게 테스트할 수 있는 프레임 워크**

**특징**

1. **Fast and Safe**
2. **Code Coverage**
3. **Easy Mocking**
4. **Great Exceptions**

## ⭐️ 테스트 코드를 작성해보자!

---

```
> npm install @types/jest

: @types/jest 설치를 통해 개발할 때 더 편리하게 해주는 "타입에 대한 정보"를 볼 수 있다.  
```

<aside>
💡 오른쪽 사진과 같이 test의 타입을 확인할 수 있다.

</aside>

<img src="https://user-images.githubusercontent.com/73332608/165264711-21126b2b-0cc4-4ac7-8050-411d1dbc0d0c.png" width="350" height="150">

<img src="https://user-images.githubusercontent.com/73332608/165264529-91eba12f-fdfa-498c-b8e6-3472c038d38f.png" width="800" height="400">

npm run test 실행시

✚) **jest.config.js에 있는 collectCoverage를 false로 바꾸면 원할 때만 coverage를 확인할 수 있다.
`jest --coverage` 를 통해 coverage를 따로 확인할 수 있다.**

## 자동 환경 설정

---

**✔︎ 항상 모든 파일들을 지켜보면서 파일이 변경될 때마다 테스트 코드를 실행시킬 수 있는 option 설정**

**watchAll 옵션**

```json
"scripts": {
	"test": "jest --watchAll"
}
```

**✔︎ 이미 commit된 테스트 코드들에 대해서는 테스트하지 않고 현재 작업중인 부분에 대해서만 테스트를 수행하고 싶다면 다음 option을 설정**

**watch 옵션**

```json
"scripts": {
	"test": "jest --watch"
}
```

```
**## git init 을 통해 git 초기화**
> *git init* 

**## node_modules 안에 있는 것들은 commit하고 싶지 않으므로 .gitignore에 표현해준다.**
> *git add .*

> *git commit -am "Just setup & add"*

## **기존의 테스트 파일은** **이미 커밋을 했기 때문에 여기서 "npm run test"로 테스트를 돌리면, 
"jest --watch" 옵션에 의해 아무 것도 테스트 되지 않는다.**
```

## 📚 공식문서

---

```jsx
const add = require('../add.js');

test('add', () => {
    // 테스트 코드 작성!
    expect(add(1, 2)).toBe(3);
})
```

→ ‘add’를 추가해서 테스트명 작성 가능

### Using Matchers > Common Matchers

1. **정확하게 일치해야할 때는 `toBe()`, object가 동일해야 한다면 `toEqual()`을 사용한다.**
    1. toEqual은 객체 안에 있는 키와 값을 모두 확인한다.
    2. 객체 그리고 배열 모두에서 사용 가능하다.
1. **Truthiness**
    
    ```jsx
    test('null', () => {
      const n = null;
      expect(n).toBeNull();
      expect(n).toBeDefined();
      expect(n).not.toBeUndefined();
      expect(n).not.toBeTruthy();
      expect(n).toBeFalsy();
    });
    ```
    

✔︎ not의 경우 뒤에 오는 메서드의 반대의 경우를 말한다.

`expect(n).not.toBeTruthy() === expect(n).toBeFalsy()`

1. **Numbers**
    
    ```jsx
    test('two plus two', () => {
      const value = 2 + 2;
      expect(value).toBeGreaterThan(3);
      expect(value).toBeGreaterThanOrEqual(3.5);
      expect(value).toBeLessThan(5);
      expect(value).toBeLessThanOrEqual(4.5);
    
      // toBe and toEqual are equivalent for numbers
      expect(value).toBe(4);
      expect(value).toEqual(4);
    });
    ```
    
    ```jsx
    test('adding floating point numbers', () => {
      const value = 0.1 + 0.2;
      //expect(value).toBe(0.3);           This won't work because of rounding error
      expect(value).toBeCloseTo(0.3); // This works.
    });
    ```
    

✔︎ 숫자 비교시 `toBe()`, `toEqual()` 모두 의미가 동일하지만, 대체로 toBe()를 사용한다.

✔︎ 실수 비교의 경우 `toBeCloseTo()`로 사용할 수 있다.

1. **Strings**
    
    ```jsx
    test('there is no I in team', () => {
      expect('team').not.toMatch(/I/);
    });
    
    test('but there is a "stop" in Christoph', () => {
      expect('Christoph').toMatch(/stop/);
    });
    ```
    

✔︎ **정규표현식**을 통해 `toMatch()`를 사용해서 매칭이 되는지 알아볼 수 있다.

1. **Arrays and iterables**
    
    ```jsx
    const shoppingList = [
      'diapers',
      'kleenex',
      'trash bags',
      'paper towels',
      'milk',
    ];
    
    test('the shopping list has milk on it', () => {
      expect(shoppingList).toContain('milk');
      expect(new Set(shoppingList)).toContain('milk');
    });
    ```
    

✔︎ `toContain()`을 통해 특정 아이템이 들어있는지 확인할 수 있다.

1. **Exceptions**
    
    ```jsx
    function compileAndroidCode() {
      throw new Error('you are using the wrong JDK');
    }
    
    test('compiling android goes as expected', () => {
      expect(() => compileAndroidCode()).toThrow();
      expect(() => compileAndroidCode()).toThrow(Error);
    
      // You can also use the exact error message or a regexp
      expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
      expect(() => compileAndroidCode()).toThrow(/JDK/);
    });
    ```
    

✔︎ `toThrow()`를 통해 코드가 에러를 던지는지 확인할 수 있다. 

## 🧪 공식문서 바탕으로 테스트 코드 작성해보기

---

### Calculator.test.js

<aside>
💡 **✔︎ *describe()*
관련된 여러 테스트들을 묶어서 테스트할 수 있다.

👀 각각의 테스트는 서로 독립적으로 동작할 수 있어야 한다.
→ 그래서 새로운 오브젝트를 각 테스트에서 선언해줘야 한다. 그런데 이걸 그대로 냅두면 코드의 중복의 너무 심해서 효율성이 떨어진다.
이럴 때 사용하는 게, `beforeEach()`이다. 이처럼 테스트 전에 무언가가 실행돼야 한다면 `beforeEach()`, 끝난 후 실행돼야 한다면 `afterEach()`를 사용한다.**

</aside>

<aside>
💡 ✔︎ **모든 테스트가 시작되기 전에 실행하는 `beforeAll()`, 모든 테스트가 끝난 후 실행하는 `afterAll()`도 있다.**

</aside>

```jsx
// calculator.js
class Calculator {
    constructor() {
    this.value = 0;
    }

    set(num) {
    this.value = num;
    }

    clear() {
    this.value = 0;
    }

    add(num) {
    const sum = this.value + num;
    if (sum > 100) {
        throw new Error('Value can not be greater than 100');
    }
    this.value = sum;
    }

    subtract(num) {
    this.value = this.value - num;
    }

    multiply(num) {
    this.value = this.value * num;
    }

    divide(num) {
    this.value = this.value / num;
    }
}

module.exports = Calculator;
```

```jsx
// calculator.test.js
const Calculator = require('../calculator.js');

describe('Calculator', () => {
    // it(): Calculator를 가리키는 3인칭 주어
    let cal;
    beforeEach(() => {
        cal = new Calculator();
    })
    it('inits with 0', () => {
        expect(cal.value).toBe(0);
    });

    it('set', () => {
        cal.set(9);
        expect(cal.value).toBe(9);
    });

    it('clear', () => {
        cal.clear();
        expect(cal.value).toBe(0);
    });

    it('add', () => {
        cal.set(1);
        cal.add(2);

        expect(cal.value).toBe(3);
    });

    it('subtract', () => {
        cal.set(2);
        cal.subtract(1);

        expect(cal.value).toBe(1);
    });

    it('multiply', () => {
        cal.set(5);
        cal.multiply(4);

        expect(cal.value).toBe(20);
    });

    describe('divides', () => {
        it('0 / 0 === NaN', () => {
            cal.divide(0);
            expect(cal.value).toBe(NaN);
        });
        it('1 / 0 === Infinity', () => {
            cal.set(1);
            cal.divide(0);
            expect(cal.value).toBe(Infinity);
        });
        it('4/ 4 ===  1', () => {
            cal.set(4);
            cal.divide(4);
            expect(cal.value).toBe(1);
        })
    })
});
```

<img src="https://user-images.githubusercontent.com/73332608/165265218-ea278aa1-a02e-4e66-a763-df0fd86f2501.png" width="350" height="250">


**👀 여기서 잠깐!!**

✔︎ collectCoverage를 false로 했을 때, test, describe name이 안보일 수 있다. 이럴 땐, package.json을 다음과 같이 설정하면 된다.

`“test”: “jest --watchAll --verbose”`
