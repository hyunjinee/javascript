# Promise

자바스크립트는 비동기 처리를 위한 하나의 패턴으로 콜백함수를 사용한다. 하지만 전통적인 콜백 패턴은 콜백헬로 인해 가독성이 나쁘고 비동기 처리 중 발생한 에러의 처리가 곤란하며 여러개의 비동기 처리를 한번에 처리하는데도 한계가 있다.

ES6에서는 비동기 처리를 위한 또다른 패턴으로 프로미스를 도입했다. 프로미스는 전통적인 콜백 패턴이 가진 단점을 보오나하며 비동기 처리 시점을 명확하게 표현할 수 있다는 장점이있다.

## 45.1 비동기 처리를 위한 콜백 패턴의 단점

```tsx
const get = (url) => {
  const xhr = XMLHttpRequest()
  xhr.open("GET", url)
  xhr.send()

  xhr.onload = () => {
    if (xhr.status === 200) {
      console.log(JSON.parse(xhr.response))
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`)
    }
  }
}
```

비동기 함수란 함수내부에 비동기로 동작하는 코드를 포함한 함수를 말한다. 비동기 함수를 호출하면 함수 내부의 비동기로 동작하는 코드가 완료되지 않았다고 해도 기다리지 않고 즉시 종료된다. 즉, 비동기 함수 내부의 비동기로 동작하는 코드는 비동기 함수가 종료된 이후에 완료된다. 따라서 비동기 함수 내부의 비동기로 동작하는 코드에서 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하면 기대한대로 동작하지 않는다. 예를들어 setTimeout함수는 비동기 함수이다. setTimeout함수가 비동기 함수인 이유는 콜백함수의 호출이 비동기로 동작하기 때문이다. setTimeout 함수를 호출하면 콜백함수를 호출 스케줄링 한다음, 타이머 id를 반환하고 즉시 종료된다. 즉 비동기 함수인 setTimeout함수의 콜백함수는 setTimeout 함수가 종료된 이후에 호출된다. 따라서 setTimeout 함수 내부의 콜백함수에서 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하면 기대한 대로 동작하지 않는다. setTimeout 함수의 콜백 함수에서 상위 스코프의 변수에 값을 할당해보자. setTimeout 함수는 생성된 타이머를 식별할 수 있는 고유한 타이머 id를 반환하므로 콜백함수에서 값을 반환하는 것은 무의미하다.

setTimeout함수의 콜백함수에서 상위 스코프의 변수에 값을 할당해보자. setTimeout 함수는 생성된 타이머를 식별할 수 있는 고유한 타이머 id를 반환하므로 콜백함수에서 값을 반환하는 것은 무의미하다.

```tsx
let g = 0
// 비동기 함수인 setTimeout 함수는 콜백함수의 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하지 못한다.
setTimeout(() => {
  g = 100
}, 0)
console.log(g) // 0
```

```tsx
const get = (url) => {
  const xhr = XMLHttpRequest()
  xhr.open("GET", url)
  xhr.send()

  xhr.onload = () => {
    if (xhr.status === 200) {
      return JSON.parse(xhr.response)
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`)
    }
  }
}
```

get 함수가 호출되면 XMLHttpRequest 객체를 생성하고, HTTP 요청을 초기화한 후, HTTP 요청을 전송합니다. 그리고 xhr.onload 이벤트 핸들러 프로퍼티에 이벤트 핸들러를 바인딩하고 종료한다. 이때 get 함수에 명시적인 반환문이 없으므로 get함수는 undefined를 반환한다.

xhr.onload 이벤트 핸들러 프로퍼티에 바인딩한 이벤트 핸들러의 반환문은 get함수의 반환문이 아니다.get함수의 반환문이 생략되었으므로 암묵적으로 undefined를 반환한다. 함수의 반환값은 명시적으로 호출한 다음에 캐치할 수 있으므로 onload이벤트 핸들러를 get함수가 호출할 수 있다면 이벤트 핸들러의 반환값을 get함수가 캐치하여 다시 반환할 수도 있겠지만 onload 이벤트 핸들러는 get함수가 호출하지 않기 때문에 그럴 수도 없다. 따라서 onload이벤트 핸들러의 반환값은 캐치할 수 없다.

```tsx
let todos

const get = (url) => {
  const xhr = XMLHttpRequest()
  xhr.open("GET", url)
  xhr.send()

  xhr.onload = () => {
    if (xhr.status === 200) {
      todos = JSON.parse(xhr.response)
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`)
    }
  }
}

get("https://jsonplaceholder.typicode.com/posts/1")
console.log(todos) // undefined
```

xhr.onload이벤트 핸들러에서 서버의 응답을 상위 스코프의 변수에 할당하면 처리 순서가 보장되지 않는다. 그 이유에 대해서 알아보자.

비동기 함수 get이 호출되면 함수 코드를 평가하는 과정에서 get함수의 실행 컨텍스트가 생성되고 실행 컨텍스트 스택(콜 스택)에 푸시된다. 이후 함수 실행과정에서 xhr.onload이벤트 핸들러 프로퍼티에 이벤트 핸들러가 바인딩된다.

get 함수가 종료하면 get함수의 실행 컨텍스트가 콜 스택에서 팝되고, 곧바로 console.log가 호출된다. 이때 console.log의 실행 컨텍스트가 생성되어 실행 컨텍스트 스택에 푸시된다. 만약 console.log가 호출되기 직전에 load이벤트가 발생했더라도 xhr.onload이벤트 핸들러 프로퍼티에 바인딩한 이벤트 핸들러는 결코 console.log보다 먼저 실행되지 않는다.

서버로부터 응답이 도착하면 xhr객체에서 load이벤트가 발생한다. 이때 xhr.onload이벤트 핸들러 프로퍼티에 바인딩한 이벤트 핸들러가 즉시 실행되는 것이 아니다. xhr.onload이벤트 핸들러는 load이벤트가 발생하면 일단 태스크 큐에 저장되어 대기하다가, 콜스택이 비면 이벤트 루프에 의해 콜스택으로 푸시되어 실행된다. 이벤트 핸들러도 함수이므로 이벤트 핸들러의 평가 → 이벤트 핸들러의 실행컨텍스트 생성 → 콜스택에 푸시 → 이벤트 핸들러 실행과정을 거친다.

따라서 xhr.onload이벤트 핸들러가 실행되는 시점에는 콜 스택이 빈 상태여야 하므로 console.log()는 이미 종료된 이후이다. 만약 get 함수 이후에 console.log()가 100번 호출된다 해도 xhr.onload 이벤트 핸들러는 모든 console.log가 종료된 이후에 실행된다.

이처럼 비동기 함수는 비동기 처리 결과를 외부에 반환할 수 없고, 상위 스코프의 변수에 할당도 할 수 없다. 따라서 비동기 함수의 처리 결과(서버의 응답 등)에 대한 후속 처리는 비동기 함수 내부에서 수행해야한다. 이때 비동기 함수를 범용적으로 사용하기 위해 비동기 함수에 비동기 처리 결과에 대한 후속 처리를 수행하는 콜백함수를 전달하는 것이 일반적이다. 필요에 따라 비동기 처리가 성공하면 호출될 콜백함수와 비동기 처리가 실패하면 호출될 콜백 함수를 전달할 수 있다.

```tsx
const get = (url, successCallback, failureCallback) => {
  const xhr = new XMLHttpRequest()
  xhr.open("GET", url)
  xhr.send()

  xhr.onload = () => {
    if (xhr.status === 200) {
      successCallback(JSON.parse(xhr.response))
    } else {
      failureCallback(xhr.status)
    }
  }
}
```

이처럼 콜백함수를 통해 비동기 처리 결과에 대한 후속 처리를 수행하는 비동기 함수가 비동기 처리 결과를 가지고 또다시 비동기 함수를 호출해야한다면 콜백함수 호출이 중첩되어 복잡도가 높아지는 현상이 발생하는데, 이를 콜백 헬이라고 한다. 다음 예시를 보자.

```tsx
const get = (url, callback) => {
  const xhr = new XMLHttpRequest()
  xhr.open("GET", url)
  xhr.send()

  xhr.onload = () => {
    if (xhr.status === 200) {
      callback(JSON.parse(xhr.response))
    } else {
      console.error(`${xhr.status} ${xhr.statusText}`)
    }
  }
}

get(`${url}/posts/1}`, ({ userId }) => {
  console.log(userId)
  get(`${url}/users/${userId}`, (userInfo) => {
    console.log(userInfo)
  })
})
```

위 예제를 보면 GET요청을 통해 서버로부터 응답을 취득하고 이 데이터를 사용하여 또다시 GET요청을 한다. 콜백 헬은 가독성을 나쁘게 하여 실수를 유발하는 원인이 된다.

비동기 처리를 위한 콜백 패턴의 문제점 중에서 가장 심각한 것은 에러처리가 곤란하다는 것이다.

```tsx
try {
  setTimeout(() => {
    throw new Error("Error!")
  }, 1000)
} catch (e) {
  console.error("캐치한 에러", e)
}
```

> try...catch...finally문
> try...catch...finally문은 에러 처리를 구현하는 방법이다. try...catch...finally문을 실행하면 먼저 try코드 블록이 실행된다. 이때 try 코드 블록에 포함된 문 중에서 에러가 발생하면 해당 에러는 catch문의 err변수에 전달되고 catch 코드 블록이 실행된다. finally 코드 블록은 에러 발생과 상관없이 반드시 한번 실행된다. try...catch...finally 문으로 에러를 처리하면 프로그램이 강제 종룓되지 않는다.

try 코드 블록 내에서 호출한 setTimeout 함수는 1초 후에 콜백 함수가 실행되도록 타이머를 설정하고, 이후 콜백함수는 에러를 발생시킨다. 하지만 이 에러는 catch 코드 블록에서 캐치되지 않는다. 비동기 함수인 setTimeout이 호출되면 setTimeout함수의 실행 컨텍스트가 생성되어 콜 스택에 푸시되어 실행된다. setTimeout은 비동기 함수이므로 콜백 함수가 호출되는 것을 기다리지 않고 즉시 종료되어 콜 스택에서 제거된다. 이후 타이머가 만료되면 setTimeout함수의 콜백함수는 태스크 큐로 푸시되고 콜스택이 비어져있을 때 이벤트 루프에 의해 콜스택으로 푸시되어 실행된다.

setTimeout함수의 콜백함수가 실행될 때 setTimeout함수는 이미 콜스택에서 제거된 상태이다. 이것은 setTimeout 함수의 콜백 함수를 호출한 것이 setTimeout함수가 아니라는 것을 의미한다.

setTimeout 함수의 콜백 함수의 호출자가 setTimeout함수라면 콜 스택의 현재 실행중인 실행 컨텍스트가 콜백함수의 실행 컨텍스트일 때 현재 실행중인 실행컨텍스트의 하위 실행컨텍스트가 setTimeout함수여야한다.

에러는 호출자 방향으로 전파된다. 즉 콜스택의 아래방향 (실행중인 실행컨텍스트가 푸시되기 직전에 푸시된 실행 컨텍스트 방향)으로 전파된다. 하지만 앞에서 살펴본 바와 같이 setTimeout 함수의 콜백함수를 호출한 것은 setTimeout 함수가 아니다. 따라서 setTimeout 함수의 콜백 함수가 발생시킨 에러는 catch블록에의해 캐치되지 않는다.

지금까지 살펴본 비동기 처리를 위한 콜백 패턴은 콜백 헬이나 에러 처리가 곤란하다는 문제가 있다. 이를 극복하기 위해 ES6에서 프로미스가 도입되었다.

## 45.2 프로미스의 생성

Promise 생성자 함수를 new 연산자와 함께 호출하면 Promise객체를 생성한다. ES6에서 도입된 Promise는 호스트 객체가 아닌 ECMAScript 사양에 정의된 표준 빌트인 객체이다.

Promise 생성자 함수는 비동기 처리를 수행할 콜백함수를 인수로 전달받는데, 이 콜백함수는 resolve와 reject함수를 인수로 전달받는다.

```tsx
const promise = new Promise((resolve, reject) => {
  if (비동기 처리 성공)  {
    resolve('result')
  } else {
    reject('failure reason')
  }
})
```

Promise 생성자 함수가 인수로 전달받은 콜백 함수 내부에서 비동기 처리를 수행한다. 이때 비동기 처리가 성공하면 콜백함수의 인자로 전달받은 resolve 함수를 호출하고 비동기 처리가 실패하면 reject 함수를 호출한다.

```tsx
const promiseGet = (url) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open("GET", url)
    xhr.send()

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.response))
      } else {
        reject(new Error(xhr.status))
      }
    }
  })
}

promiseGet("https://jsonplaceholder.typicode.com/users")
```

비동기 함수인 promiseGet은 함수 내부에서 프로미스를 생성하고 반환한다. 비동기 처리는 Promise 생성자 함수가 인수로 전달받은 콜백함수 내부에서 수행한다. 만약 비동기 처리가 성공하면 비동기 처리 결과를 resolve 함수에 인수로 전달하면서 호출하고, 비동기 처리가 실패하면 에러를 reject함수에 인수로 전달하면서 호출한다.

프로미스는 다음과 같이 현재 비동기 처리가 어떻게 진행되고 있는지를 나타내는 상태 정보를 갖는다.

pending : 비동기 처리가 아직 수행되지 않은 상태

fulfilled : 비동기 처리가 수행된 상태(성공)

rejected: 비동기 처리가 수행된 상태(실패)

생성된 직후의 프로미스는 기본적으로 pending 상태가된다. 이후 비동기 처리가 수행되면 비동기 처리 결과에 다라 다음과 같이 프로미스의 상태가 변경된다.

- 비동기 처리 성공: resolve 함수를 호출해 프로미스를 fullfilled 상태로 변경한다.
- 비동기 처리 실패: reject함수를 호출해 프로미스를 rejected 상태로 변경한다.

fulfilled 또는 rejected 상태를 settled 상태라고 한다. settled 상태는 fullfilled 또는 rejected 상태와 상관없이 pending이 아닌 상태로 비동기 처리가 수행된 상태를 말한다.

프로미스는 pending 상태에서는 fulfilled 또는 rejected 상태, 즉 settled 상태로 변화할 수 있다. 하지만 settled 상태가 되면 더는 다른 상태로 변화할 수 없다.

프로미스는 비동기 처리 상태와 더불어 비동기 처리 결과도 상태로 갖는다. 비동기 처리가 성공하면 프로미스는 pending 상태에서 fulfilled 상태로 변화한다. 비동기 처리가 실패하면 프로미스는 pending 상태에서 rejected상태로 변화한다. 그리고 비동기 처리 결과인 Error 객체를 값으로 갖는다. 즉, 프로미스는 비동기 처리 상태와 처리 결과를 관리하는 객체이다.

## 45.3 프로미스의 후속 처리 메서드

프로미스의 비동기 처리 상태가 변화하면 이에 따른 후속 처리를 해야한다. 예를 들어, 프로미스가 fulfilled 상태가 되면 프로미스의 처리 결과를 가지고 무언가를 해야하고, 프로미스가 rejected 상태가 되면 프로미스의 처리 결과를 가지고 에러 처리를 해야한다. 이를 위해 프로미스는 후속 메서드 then, catch, finally를 제공한다. 이를 위해 프로미스는 후속 메서드 then,catch,finally를 제공한다. 프로미스의 비동기 처리 상태가 변화하면 후속 처리 메서드에 인수로 전달한 콜백 함수가 선택적으로 호출된다. 이때 후속 처리 메서드의 콜백 함수에 프로미스의 처리 결과가 인수로 전달된다.

모든 후속 처리 메서드는 프로미스를 반환하며, 비동기로 동작한다. 프로미스의 후속 처리 메서드는 다음과 같다.

### 45.3.1 Promsie.prototype.then

then 메서드는 두개의 콜백 함수를 인수로 전달받는다.

- 첫번째 콜백함수는 프로미스가 fulfilled 상태 (resolve 함수가 호출된 상태)가 되면 호출된다. 이때 콜백 함수는 프로미스의 비동기 처리 결과를 인수로 전달받는다.
- 두 번째 콜백함수는 프로미스가 rejected 상태(reject 함수가 호출된 상태)가 되면 호출된다. 이때 콜백함수는 프로미스의 에러를 인수로 전달받는다

즉, 첫번째 콜백함수는 비동기 처리가 성공했을 때 호출되는 성공 처리 콜백함수이며, 두번째 콜백함수는 비동기 처리가 실패했을 때 호출되는 실패 처리 콜백함수이다.

```tsx
new Promise((resolve) => resolve("fulfilled")).then(
  (v) => console.log(v),
  (e) => console.error(e)
) // fulfilled

new Promise((_, reject) => reject(new Error("rejected"))).then(
  (v) => console.log(v),
  (e) => console.error(e)
) // Error: rejected
```

then 메서드는 언제나 프로미스를 반환한다. 만약 then메서드의 콜백함수가 프로미스를 반환한다면 그 프로미스를 그대로 반환하고, 콜백함수가 프로미스가 아닌 값을 반환하면 그 값을 암묵적으로 resolve 또는 reject하여 프로미스를 생성해 반환한다.

### 45.3.2 Promise.prototype.catch

catch 메서드는 한개의 콜백 함수를 인수로 전달받는다. catch메서드의 콜백함수는 프로미스가 rejected 상태인 경우만 호출된다.

```tsx
new Promise((), reject) => reject(new Error('rejected'))
.catch(e => console.log(e));
```

catch 메서드는 then(undefined, onRejected) 와 동일하게 동작한다. 따라서 then 메서드와 마찬가지로 언제나 프로미스를 반환한다.

```tsx
new Promise((_, reject) => reject(new Error('rejected'))
.then(undefined, e => console.log(e)); // Error: rejected
```

### 45.3.3 Promise.prototype.finally

finally 메서드는 한개의 콜백 함수를 인수로 전달받는다. finally 메서드의 콜백함수는 프로미스의 성공 또는 실패 유무와 상관없이 무조건 한번 호출된다. finally 메서드는 프로미스의 상태와 상관없이 공통적으로 수행해야 할 처리 내용이 있을 때 유용하다. finally 메서드도 then/catch 메서드와 마찬가지로 언제나 프로지스를 반환한다.

```tsx
new Promise(() => {}).finally(() => console.log("finally"))
```

프로미스로 구현한 비동기 함수 get을 사용해 후속처리를 구현해보자.

```tsx
const promiseGet = (url) => {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest()
    xhr.open("GET", url)
    xhr.send()

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(xhr.response)
      } else {
        reject(new Error(xhr.status))
      }
    }
  })
}

promiseGet("https://jsonplaceholder.typicode.com/posts/1")
  .then((res) => console.log(res))
  .catch((err) => console.error(err))
  .finally(() => console.log("finally"))
```

## 45.3 프로미스의 에러 처리

에러 처리의 한계에서 살펴보았듯이 비동기 처리를 위한 콜백 패턴은 에러 처리가 곤란하다는 문제가 있다. 프로미스는 에러를 문제없이 처리할 수 있다.

위 예제의 비동기 함수 get은 프로미스를 반환한다. 비동기 처리 결과에 대한 후속처리는 프로미스가 제공하는 후속 처리 메서드 then, catch, finally를 사용하여 수행한다. 비동기 처리에서 발생한 에러는 then메서드의 두번째 콜백 함수로 처리할 수 있다.

```tsx
const wrongUrl = "https://jsonplaceholder.typicode.com/XXX/1"

promiseGet(wrongUrl).then(
  (res) => console.log(res),
  (err) => console.log(err)
)
```

비동기 처리에서 발생한 에러는 프로미스의 후속 처리 메서드의 catch를 사용해 처리할 수도 있다.

```tsx
const wrongUrl = "https://jsonplaceholder.typicode.com/XXX/1"

promiseGet(wrongUrl)
  .then((res) => console.log(res))
  .catch((err) => console.error(err))
```

catch 메서드를 호출하면 내부적으로 then(undefined, onRejected)를 호출한다. 따라서 위 예제는 다음과 같이 처리된다.

```tsx
const wrongUrl = "https://jsonplaceholder.typicode.com/XXX/1"

promiseGet(wrongUrl)
  .then((res) => console.log(res))
  .then(undefined, (err) => console.log(err))
```

단 , then메서드의 두번째 콜백 함수는 첫 번째 콜백 함수에서 발생한 에러를 캐치하지 못하고 코드가 복잡해져서 가독성이 좋지 않다.

```tsx
promiseGet("https://jsonplaceholder.typicode.com/todos/1").then(
  (res) => console.xxx(res),
  (err) => console.error(err)
)
// 두번째 콜백 함수는 첫번 째 콜백 함수에서 발생한 에러를 캐치하지 못한다.
```

catch 메서드는 모든 then메서드를 호출한 이후에 호출하면 비동기 처리에서 발생한 에러(rejected 상태)뿐만 아니라 then메서드 내부에서 발생한 에러까지 모두 캐치할 수 있다.

```tsx
const wrongUrl = "https://jsonplaceholder.typicode.com/XXX/1"

promiseGet(wrongUrl)
  .then((res) => console.xxx(res))
  .catch((err) => console.error(err))
```

또한 then메서드에 두번째 콜백 함수를 전달하는 것보다 catch메서드를 사용하는 것이 가독성이 좋고 명확하다. 따라서 에러 처리는 then메서드에서 하지 말고 catch 메서드에서 하는 것을 권장한다.

## 45.5 프로미스 체이닝

‘콜백 헬'에서 살펴보았듯이 비동기 처리를 위한 콜백 패턴은 콜백 헬이 발생하는 문제가 있다. 프로미스는 then,catch,finally후속 처리 메서드를 통해 콜백 헬을 해결한다.

```tsx
const url = 'https~~'
promiseGet(`${url}/posts/1`)
.then(({userId}) => promiseGet(`${url}/users/${userId}`)
.then(userInfo => console.log(userInfo))
.catch(err => console.error(err))
```

위 예제에서 then → then → catch순서로 후속 처리 메서드를 호출했다. then, catch, finally 후속 처리 메서드는 언제나 프로미스를 반환하므로 연속적으로 호출할 수 있다. 이를 프로미스 체이닝이라고 한다.
