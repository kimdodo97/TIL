## 1. 템플릿 문자열 (백틱 `)
백틱은 파이썬에서 사용하던 f"" 처럼 문자열 내부에서 문자를 {} 으로 감싸면 해당 구간을 선언된 변수로 사용하는 방식이다.
아래 코드 예시와 같이 사용가능

```javascript
const a = 'hello';
const b = true;
const c = 3;
//const d = a + ' ' + b + ' ' + c;
const d = `${a} ${b} ${c}`;
//백틱을 통해서 문제 해결 
//파이썬 f 나 C# $랑 같은 기능 {} 내부를 변수로 인식하는 방법
console.log(d)
```

## 2. 비구조화 할당
비구조화 할당이란 아래 코드의 주석과 같이 변수의 status의 객체에 직접 접근 하는 것이 아닌 
>**const {status,getCandy} = candyMachine;**

할당 하는 방식이다.
이렇게 객체 내부에 여러 개에 접근 하거나 변수에 할당 할때 유용하게 사용하며 자세한 활용은 추후 강의에서 보여진다.

📕비구조화 할당의 this
>**getCandy()**

비구조화 할당을 통해서 위와 같이 getCandy()를 실행하며 undefined 에러가 발생한다.
이는 비구조화 할당을 통하면 getCandy()가 candyMachine을 찾지 못한느 문제가 있다
this를 candyMachine을 찾아주는 다른 메서드를 사용해야 한다.
ex) getCany.call(candyMachine) or candyMachine.getCandy()

```javascript
var candyMachine = {
    status : {
        name : 'node',
        count : 5,
    } ,
    getCandy : function() {
        this.status.count--;
        return this.status.count;
    }
};
//function 생략 가능

// const status = candyMachine.status;
// const getCandy = candyMachine.getCandy;
//아래 처럼 변경 가능
const {status,getCandy} = candyMachine;
//const {Router} = require('express');
//console.log(getCandy());
console.log(candyMachine.getCandy());

```

#### 배열의 비구조화 할당
배열 또한 비구조화 할당이 가능하다.
const array 에서 할당된 4개의 인자값을 
const [?,?,?,?]를 통해서 할당 가능하다.
즉 위에서 3줄에 코드를 통해서 할당한 것을 한줄을 코드로 생성하였다. 

```javascript
var array = ['node.js', {}, 10, true];

var node = array[0];
var obj = array[1];
var bool = array[array.length-1];

const array = ['node.js', {}, 10, true];
const [node,obj,,bool] = array;
//두개의 식은 동일
```