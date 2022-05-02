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

## 🧪 에러 테스트 하기

---

### 현재 test의 문제점

> **calculator.js에서 add 메서드를 보면, 합이 100을 초과할 때 에러를 던지는 구문이 있는데, calculator.test.js에서는 이를 따로 표현해주는 테스트 코드가 없다.
이 상황에서 jest --coverage를 터미널에 입력하면 다음과 같이 나온다.**
> 

<img src="https://user-images.githubusercontent.com/73332608/166111027-7f8ee373-6b16-4f51-96fb-161e0e768db7.png" width="480" height="250">

✔︎ `Uncovered line`을 보면 17~18번째 줄은 커버할 수 있는 테스트 코드가 없다는 것을 알 수 있다. (해당 line에는 위에서 말한 합이 100을 초과할 때에 대한 코드가 작성돼있다.) → 94.28퍼센트만 테스트 완료됐다.

```jsx
// calculator.test.js 추가 코드
it('add should throw an error if value is greater than 100', () => {
    // error 예상 코드 작성하기
    // expect안에 또다른 콜백함수를 전달하면서 해당 함수 안에서 에러를 던져지길 예상하는 코드를 넣으면 된다.
    expect(() => {
        cal.add(101);
    }).toThrow('Value can not be greater than 100')
});
```

✔︎ 위와 같이 코드를 추가해주면 Uncovered Line부분이 없어지고 100퍼센트 테스트 코드가 완료된다.

<aside>
💡 **✔︎ 이처럼, 테스트코드에서 예외 처리는 expect를 통해 표현이 가능하다.
→ expect안에 콜백함수를 전달하고 해당 콜백함수 안에서 에러가 던져지는 코드를 수행하면 된다. toThrow를 통해 에러메시지를 확인할 수 있다.**

</aside>

[Using Matchers · Jest](https://jestjs.io/docs/using-matchers#exceptions)

공식 문서 참고

## ⚙️ 비동기 테스트 하기

---

<aside>
💡 **비동기 코드를 테스트할 때는, done보다는 return을 이용하거나, async-await을 이용하는 것이 좋다.**

</aside>

```jsx
// async.js
function fetchProduct(error) {
    if (error === 'error') {
        return Promise.reject('network error');
    }
    return Promise.resolve({ item: 'Milk', price: 200 });
}

module.exports = fetchProduct;
```

> **✔︎ 비동기 테스트 방식에는 done방식, return방식, async-await방식, resolve-reject방식 이 있다.

✔︎ 이 중에서 done방식은 속도가 느리기 때문에 거의 사용하지 않는다.**
> 

```jsx
// async.test.js
const fetchProduct = require('../async.js');

describe('Async', () => {
    // return에 비해 done방식이 더 느림.
		// done 방식
    it('async-done', (done) => {
        fetchProduct().then(item => {
            expect(item).toEqual({ item: 'Milk', price: 200 });
            done();
        });
    });
		
		// return 방식
    it('async-return', () => {
        return fetchProduct().then(item => {
            expect(item).toEqual({ item: 'Milk', price: 200 });
        });
    });

    // async-await 방식
    it('async-await', async () => {
        const product = await fetchProduct();
        expect(product).toEqual({ item: 'Milk', price: 200 });
    });

		// resolve, reject 방식
    it('async-resolves', () => {
        return expect(fetchProduct()).resolves.toEqual({
            item: 'Milk',
            price: 200,
        });
    });

    it('async-reject', () => {
        return expect(fetchProduct('error')).rejects.toBe('network error');
    });
});
```

[Testing Asynchronous Code · Jest](https://jestjs.io/docs/asynchronous)

## ⭐️ Mock에 대해 (Basic)

---

> **✔︎ 함수를 구현하지 않아도, Mock Function의 간편한 API를 이용해서 좀 더 간단하게 테스트 코드를 작성할 수 있다.
→ 어떤 함수가 몇 번 호출되었는지, 어떤 인자와 함께 호출되었는지 확인할 수 있다.**
> 

```jsx
// check.js
function check(predicate, onSuccess, onFail) {
    if (predicate()) {
        onSuccess('yes');
    } else {
        onFail('no');
    }
}

module.exports = check;
```

> ✔︎ mock 함수 관련 변수를 미리 선언해서 이를 활용해볼 수 있다.

✔︎ check.js 코드를 따르면 “predicate()는 true를 리턴하는 간단한 콜백함수”이다. 
→ `check(() ⇒ true, onSuccess, onFail);`

✔︎ onSuccess라는 mock 함수의 오브젝트 안에 있는 데이터가 호출된 횟수 관련 테스트 코드를 수동적으로 표현하면 
→ `expect(onSuccess.mock.calls.length).toBe(1)` 이지만,
이는 직관적이지 않으므로 좀 더 간단하고 명시적으로 표현하면
→ `expect(onSuccess).toHaveBeenCalledTimes(1)` 이다.
다른 코드들도 같은 방식이다.
> 

```jsx
// check.test.js
const check = require('../check.js');

describe('check', () => {
    // 1. mock 함수 변수 선언
    let onSuccess;
    let onFail;

    // 2. 테스트 코드 시작전 초기화 - 각각 mock fn 선언
    beforeEach(() => {
        onSuccess = jest.fn();
        onFail = jest.fn();
    });

    it('should call onSuccess when predicate is true', () => {
        // 준비해둔 mock fn 전달
        check(() => true, onSuccess, onFail);

        // true -> onSuccess 호출, onFail 호출x 검증
        // onSuccess라는 mock fn object에 있는 데이터의 calls(호출)의 횟수(length)가 1번이다.
        // expect(onSuccess.mock.calls.length).toBe(1);
        // 위 코드는 직관적이지 않으므로, 간편한 API를 활용해보자.
        expect(onSuccess).toHaveBeenCalledTimes(1);
        
        // yes가 전달된다. (검증)
        // 첫번째 호출의 첫번째 인자 === 'yes
        // expect(onSuccess.mock.calls[0][0]).toBe('yes');
        expect(onSuccess).toHaveBeenCalledWith('yes');

        // onFail은 호출되면 안된다.
        // expect(onFail.mock.calls.length).toBe(0);
        expect(onFail).toHaveBeenCalledTimes(0);
    });

    it('should call onFail when predicate is false', () => {
        check(() => false, onSuccess, onFail);

        expect(onFail).toHaveBeenCalledTimes(1);
        expect(onFail).toHaveBeenCalledWith('no');
        expect(onSuccess).toHaveBeenCalledTimes(0);
    });
})
```

[Mock Functions · Jest](https://jestjs.io/docs/mock-functions)

<aside>
💡 **이 부분은 Jest 공식문서에서도 볼 수 있는 간단한 방식이고 이어서 mock 함수 관련 심화된 내용에 대해 더 알아보자!**

</aside>

## Mock에 대해 (Advanced)

---

> **코드의 아키텍처를 개선하지 않고 Mock을 남용하는 예제를 한번 보자**
> 

### 👀 Mock을 이용한 테스트 - 제품 정보 가지고 오기

<aside>
💡 **✔︎ 우리가 원하는 것은 `ProductService`를 테스트하는 것인데, 여기서 다른 모듈 클래스를 사용하든 영향을 받지 않도록 모든 의존성에 대해 mock을 사용할 것이다.

→ `ProductClient`가 어떤 것을 리턴하는지에 대해 Mock 함수와 Mock Implementation을 이용해서 어떤 함수를 호출했을 때 어떤 데이터를 받아오는지 쉽게 컨트롤 할 수 있게 설정해둘 것이다.
→ 이에 따라, 우리가 테스트하고자 하는 ProductService 안에 있는 함수(fetchAvailable())는 가능한 한, available: true인 값들만 필터링하는지에 대한 부분만 집중해서 테스트 코드를 작성할 수 있는 것이다.**

</aside>

<aside>
💡 **✔︎ 이 방법을 썼을 때, 큰 장점이 있다.

→ ProductClient가 실패하거나 네트워크 상태가 불안정해서 네트워크 자체에서 실패하든, 환경적인 영향을 받지 않고 우리가 원하는 로직만 검증할 수 있다.
→ 이를 바로 모듈 간 의존성이 없는 “*단위 테스트*”라고 한다.**

</aside>

```jsx
// product_client.js
class ProductClient {
    fetchItems() {
        return fetch('http://example.com/login/id+password').then(response => {
            response.json()
        });
    }
}

module.exports = ProductClient;
```

```jsx
// product_service_no_di.js
const ProductClient = require('./product_client.js');
class ProductService {
    constructor() {
        this.ProductClient = new ProductClient();
    }

    // 테스트할 코드 - 사용자가 구매가능한 아이템만 보여주도록 하는 부분.
    // Mock Function을 통해 테스트해보자
    fetchAvailableItems() {
        return this.ProductClient
            .fetchItems()
            .then(items => items.filter(item => item.available));
    }
}

module.exports = ProductService;
```

```jsx
// product_service_no_di.test.js
const ProductService = require('../product_service_no_di.js');
const ProductClient = require('../product_client.js');
jest.mock('../product_client');

describe('Product Service', () => {
    const fetchItems = jest.fn(async () => [
        { item: '🥛', available: true },
        { item: '🍌', available: false },
    ]);
    // 함수와 mock하고 있는 product_client를 연결하자
    ProductClient.mockImplementation(() => {
        return {
            fetchItems: fetchItems
        }
    })
    let productService;

    beforeEach(() => {
        productService = new ProductService();
        fetchItems.mockClear();
        ProductClient.mockClear();
    });
    // ProductService와 ProductClient는 독립적으로 테스트 코드를 작성하는 것이 좋으므로 ProductClient를 Mock으로 만든다.

    it('should filter out only available items', async () => {
        const items = await productService.fetchAvailableItems();
        expect(items.length).toBe(1);
        expect(items).toEqual([{ item: '🥛', available: true }]);
    });
    it('test', async () => {
        const items = await productService.fetchAvailableItems();
        expect(fetchItems).toHaveBeenCalledTimes(1);
    });
})
```

<aside>
💡 `**beforeEach()`에서 `fetchItems`와 `ProductClient`를 `mockClear`해준 이유는, mock한 것을 초기화해야 이를 반복해서 사용한 것에 대한 오류가 생기지 않기 때문이다. 해당 방법은 수동적인 방법이며 자동적으로 설정하기 위해서는 `jest.config.js`에서 `clearMocks: true`로 설정해주면 된다.**

</aside>

## 🎉 Mock과 Stub의 차이

---

### 제품 정보 가지고 오기 Refactoring

> **👿 위 예제가 왜 Mock을 남용하는 사례일까??? 
위 코드를 좀 더 간단하게 작성해보자!**
> 

✔︎ **Mock은 내가 원하는 것만 부분적으로 흉내낸다면, Stub은 기존에 쓰이는 interface를 충족하는 실제로 구현된 코드인데, 실제 값과 대체 가능한 것을 말한다.**

즉, 위에서는 `const fetchItems = jest.fn(async () => [
        { item: '🥛', available: true },
        { item: '🍌', available: false },
    ]);` 와 같이 쓴 부분을 `stub_product_client.js`에 아래 코드와 같이 서서 대체할 수 있다는 뜻이다.

👀 클래스 내부에서 스스로의 의존을 결정하고 정의하고 만들어 사용하는 것은 의존성 주입의 원칙(**Dependency Injection**)에 어긋난다.

1. 필요한 것은 외부에서 받아와야 한다.
2. 테스트할 때는 테스트용 `productClient`를 주입하고(→ `stub_product_client.js`), 실제 production에서는 실제 `ProductClient`를 주입하면된다.

```jsx
// product_service.js
class ProductService {
    constructor(productClient) {
        this.productClient = productClient;
    }

    fetchAvailableItems() {
        return this.productClient
            .fetchItems()
            .then(items => items.filter(item => item.available));
    }
}

module.exports = ProductService;
```

```jsx
// stub_product_client.js
// 테스트용으로 사용 
// 네트워크 사용 없이 데이터를 바로 리턴하는 함수 (네트워크 의존 x)
class StubProductClient {
    async fetchItems() {
        return [
            { item: '🥛', available: true },
            { item: '🍌', available: false },
        ]
    }
}

module.exports = StubProductClient;
```

```jsx
// product_service.test.js
const ProductService = require('../product_service.js');
const StubProductClient = require('./stub_product_client.js');

describe('ProductService - Stub', () => {
    let productService;

    beforeEach(() => {
        productService = new ProductService(new StubProductClient());
    });

    it('should filter out only available items', async () => {
        const items = await productService.fetchAvailableItems();
        expect(items.length).toBe(1);
        expect(items).toEqual([{ item: '🥛', available: true }]);
    });
})
```

<aside>
💡 **mock과 stub 둘 중 어떤 것이 더 낫다 얘기할 수 없다. 
상황에 맞게 어떤 것을 쓸지 유연하게 결정해야 한다.**

</aside>
<br>

## Mock을 이용한 테스트

---

### 👀 사용자 로그인

> **행동에 대한 테스트(호출했는지 안했는지와 같은...)는 stub으로 구현하기에는 무리가 있으므로 mock을 이용해보자.**
> 

```jsx
// user_client.js
class UserClient {
    login(id, password) {
        return fetch('http://example.com/login/id+password')
            .then(response => response.json());
    }
}

module.exports = UserClient;
```

```jsx
// user_service.js
class UserService {
    constructor(userClient) {
        this.userClient = userClient;
        this.isLogedIn = false;
    }

    login(id, password) {
        if (!this.isLogedIn) {
            return this.userClient
                .login(id, password)
                .then(data => this.isLogedIn = true);
        }
    }
}

module.exports = UserService;

```

```jsx
// user_service.test.js
const UserService = require('../user_service.js');
const UserClient = require('../user_client.js');
jest.mock('../user_client')

describe('UserService', () => {
    const login = jest.fn(async () => 'success');
    UserClient.mockImplementation(() => {
        return {
            login
        };
    });
    let userService;

    beforeEach(() => {
        userService = new UserService(new UserClient());
    });

    it('calls login() on UserClient when trying to login', async () => {
        await userService.login('abc', 'abc');
        expect(login.mock.calls.length).toBe(1);
    });

    it('should not call login() on UserClient again if already logged in', async () => {
        await userService.login('abc', 'abc');
        await userService.login('abc', 'abc');

        expect(login.mock.calls.length).toBe(1);
    })
})
```