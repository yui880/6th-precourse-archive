# 자주 사용되는 코드 모음 


[MDN 문서](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)

| 메서드                                        | 설명                                                |
|--------------------------------------------|---------------------------------------------------|
| `Array.prototype.some()`                   | 배열안의 어떤 요소가 주어진 판별 함수를 적어도 하나라도 통과하는지 테스트         |
| `Array.prototype.every()`                  | 배열의 모든 요소가 제공된 함수로 구현된 테스트를 통과하는지 테스트             |
| `Number.prototype.toLocaleString('ko-KR')` | 숫자를 한국식 숫자 세기(천단위 구분자) 형식 문자열로 만드는 메서드            |
| `Array.prototype.find()`                   | 배열에서 제공된 테스트 함수를 만족하는 첫 번째 요소를 반환, 없으면 undefined  |
| `Array.prototype.findIndex()`              | 배열에서 테스트 조건을 만족하는 첫 번쨰 요소의 인덱스 반환(indexOf는 값만 가능) |
| `Array.prototype.reduce()`                 | initialValue가 없는 경우 acc는 첫번째 값, cur는 두번째 값        |
| `Array.prototype.sort()`                   | a,b랑 연산했을 때 음수값이면 a가 먼저옴, a,b 연산시 양수값이면 b가 먼저옴    |
| `Object.assign()`                           | 객체들의 모든 열거 가능한 자체 속성을 복사해 대상 객체에 붙여 넣고 대상 객체를 반환함  |


--- 
### README.md
```markdown
# 기능 구현 목록 

---
## 📌 주제 
> 

---
## 📍 프로그램 흐름 

---
## 📚 기능 목록 

---
## 📒 예외 목록

---
## 💡주의할 요구사항

```

---
### 천 단위 구분자 
- toLocaleString()가 소수점을 제대로 표현할 수 없는 경우에 대신 사용
```js
export const REGEX = {
  ThousandSeparator: /\B(?=(\d{3})+(?!\d))/g,
};

formatNumber(number) {
    return String(number).replace(REGEX.ThousandSeparator, ',');
}
```

### 여러 번 자르기 
```js
#splitMenu(menus) {
    return menus
        .split(SEPARATOR.item)
        .map((menu) => menu.split(SEPARATOR.count))
        .map(([name, count]) => [name.trim(), Number(count)]);
}
```

### 2개씩 배열 자르기 
```js
#getSectionList(path) {
    return Array.from({ length: path.length - 1 }, (_, index) => [path[index], path[index + 1]]);
  }
```

---
## 예외 처리

### ValidationError
```js
class ValidationError extends Error {
  constructor(message) {
    super(`${ERROR.errorPrefix} ${message}`);
    this.name = 'ValidationError';
  }
}

export default ValidationError;

```

### ERROR 상수 
```js
export const ERROR = {
    errorPrefix: '[ERROR]',
};

```


### handleException (재입력 받기)
```js
async #handleException(callback) {
    while (true) {
        try {
            return await callback();
        } catch (error) {
            OutputView.printError(error.message);
        }
    }
}
```

### 오류 발생시 아예 처음부터 입력받는 방법 
```js
 async run() {
    await this.handleException(() => this.startProgram());
  }
```

---
## Test

```js
describe('클래스 테스트',()=>{
    describe('기능 테스트', ()=> {
        test.each()('', ()=> {})
        test.each()('', ()=> {})
        
    })
    describe('예외 테스트', () => {
        test.each()('', ()=> {})
        test.each()('', ()=> {})
        
    });
})

```


### mockQuestions
```js
const mockQuestions = (inputs) => {
    MissionUtils.Console.readLineAsync = jest.fn();

    MissionUtils.Console.readLineAsync.mockImplementation(() => {
        const input = inputs.shift();
        return Promise.resolve(input);
    });
};
```

### mockRandoms
```js
const mockRandoms = (numbers) => {
    MissionUtils.Random.pickNumberInRange = jest.fn();
    numbers.reduce((acc, number) => {
        return acc.mockReturnValueOnce(number);
    }, MissionUtils.Random.pickNumberInRange);
};
```

### getLogSpy
```js
const getLogSpy = () => {
    const logSpy = jest.spyOn(MissionUtils.Console, "print");
    logSpy.mockClear();
    return logSpy;
};

// 사용법
outputs.forEach((output) => {
    expect(logSpy).toHaveBeenCalledWith(expect.stringContaining(output));
});
```

### getOutput 
- 여러개의 출력 로그를 LINE_SEPARATOR로 묶어서 하나의 문자열로 만드는 함수
```js
import { EOL as LINE_SEPARATOR } from "os";

const getOutput = (logSpy) => {
    return [...logSpy.mock.calls].join(LINE_SEPARATOR);
};
```

### expectLogContains
- getOutput으로 만든 큰 출력 로그 안에 expectedLogs의 요소들을 포함하고 있는지 확인하는 함상
```js
const expectLogContains = (received, expectedLogs) => {
    expectedLogs.forEach((log) => {
        expect(received).toContain(log);
    });
};

// 사용법
expectLogContains(getOutput(logSpy), expected);
```

### 에러 메시의 개수 세기 
````js
const errorCount = logSpy.mock.calls.filter(([logMessage]) =>
    logMessage.includes(ERROR.invalidDate),
).length;

expect(errorCount).toBe();
````


### mock
```js
const getCustomProductMock = ({
  isPriceLessThan = false,
  dessertOrderCount = 0,
  mainOrderCount = 0,
}) => ({
  isPriceLessThan: jest.fn(() => isPriceLessThan),
  getCountByCategory: jest.fn((category) => {
    if (category === 'dessert') {
      return dessertOrderCount;
    }
    if (category === 'main') {
      return mainOrderCount;
    }
    return 0;
  }),
});
```

---
## Validator

```js
const Validator = {
    validateUserInput(input) {
        this.checkIsEmpty(input);
    },

    checkIsEmpty(input) {
        if (input.trim() === '') {
            throw new ValidationError(ERROR.isEmpty);
        }
    },

    checkIsInteger(input) {
        if (!this.isInteger(input)) {
            throw new ValidationError(ERROR.isNotNumber);
        }
    },

    checkIsPositive(input) {
        if (Number(input) <= 0) {
            throw new ValidationError(ERROR.isNotPositive);
        }
    },
    
    checkHasNonNumeric(inputs) {
        inputs.forEach((input) => {
            if (!this.isInteger(input)) {
                throw new ValidationError(ERROR.hasNonNumeric);
            }
        });
    },

    checkHasNotInRange(inputs) {
        inputs.forEach((input) => {
            if (!this.isInRange(input)) {
                throw new ValidationError(ERROR.hasNotInRange);
            }
        });
    },

    checkIsInRange(input) {
        if (!this.isInRange(input)) {
            throw new ValidationError(ERROR.isNotInRange);
        }
    },

    isInteger: (input) => !Number.isNaN(Number(input)) && Number.isInteger(Number(input)),
    isInRange: (input) => Number(input) >= LOTTO_NUMBERS.min && Number(input) <= LOTTO_NUMBERS.max,
};

export default Validator;

```