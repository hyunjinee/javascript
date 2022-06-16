# String

표준 빌트인 객체인 String은 원시 타입인 문자열을 다룰 때 유용한 프로퍼티와 메서드를 제공합니다.

## String 생성자 함수

String객체는 생성자 함수 객체다. 따라서 new 연산자와 함께 호출하여 String 인스턴스를 생성할 수 있다.

String생성자 함수에 인수를 전달하지 않고 new 연산자를 호출하면 [ [ StirngData ] ] 내부 슬롯에 빈 문자열을 할당한 String 래퍼 객체를 생성한다.

```tsx
const strObj = new String()
console.log(strObj) // String { length: 0 , [[PrimitiveValue]]: ""}
```

위 예제를 크롬 부라우저의 개발자 도구에서 실행하면 [ [ PrimitiveValue ] ]라는 접근할 수 없는 프로퍼티가 보인다. 이는 [ [ StringData ] ] 내부 슬롯을 가리킨다. ES5 에서는 [ [ StringData ] ]를 [ [ PrimitiveValue ] ]라 불렀다.

String 생성자 함수의 인수로 문자열을 전달하면서 new 연산자와 함께 호출하면 [ [ StringData ] ] 내부 슬롯에 인수로 전달받은 문자열을 할당한 String 래퍼 객체를 생성한다.

```tsx
const strObj = new String("lee")
console.log(strObj)
// String { 0 : "l" , 1 : "e" , 2 : "e" , length: 3 , [[PrimitiveValue]]: "lee" }
```

String 래퍼 객체는 배열과 마찬가지로 length 프로퍼티와 인덱스를 나타내는 숫자 형식의 문자열을 프로퍼티 키로, 각 문자를 프로퍼티 값으로 갖는 유사 배열 객체이면서 이터러블이다.

문자열은 원시 값이므로 변경할 수 없다. 이때 에러가 발생하지 않는다.

```tsx
strObj[0] = "s"
console.log(strObj) // 'lee'
```

String 생성자 함수의 인수로 문자열이 아닌 값을 전달하면 인수를 문자열로 강제 변환한 후 [ [ StringData ] ] 내부 슬롯에 변환된 문자열을 할당한 String 래퍼 객체를 생성한다.

```tsx
let strObj = new String(123)
console.log(strObj)
// String {0: "1", 1: "2", 2: "3", length: 3 , [[PrimitiveValue]]: "123"}
strObj = new String(null)
console.log(strObj)
// String {0: "n", 1:"u", 2: "l", 3: "l", length:4, [[PrimitiveValue]]:"null"}
```

new 연산자를 사용하지않고 String생성자 함수를 호출하면 String인스턴스가 아닌 문자열을 반환한다. 이를 이용하여 명시적으로 타입을 변환하기도 한다.

```tsx
String(1) // "1"
String(NaN) // "NaN"
String(Infinity) // "Infinity"

String(true) // "true"
String(false) // "false"
```

String래퍼 객체는 배열과 마찬가지로 length프로퍼티를 갖는다. 그리고 인덱스를 나타내는 숫자를 프로퍼티 키로, 각문자를 프로퍼티 값으로 가지므로 String 래퍼 객체는 유사 배열 객체이다.

배열에는 원본 배열 (배열 메서드를 호출한 배열)을 직접 변경하는 메서드와 원본 배열을 직접 변경하지 않고 새로운 배열을 생성하여 반환하는 메서드가 있다. 하지만 String객체에는 원본 String래퍼 객체를 직접 변경하는 메서드는 존재하지 않는다. 즉 String객체의 메서드는 언제나 새로운 문자열을 반환한다. 문자열은 변경 불가능한 원시값이기 때문에, String래퍼 객체도 읽기 전용 객체(read only)로 제공된다.

```tsx
const strObj = new String("LEE")

console.log(Object.getOwnPropertyDescriptors(strObj));
// String 래퍼 객체는 읽기 전용 객체다. 즉 writable프로퍼티 어트리뷰트 값이 false이다.
{
	"0" : {value: "L", writable: false, enumerable: true, configurable: false},
	"1" : {value: "E", writable: false, enumerable: true, configurable: false},
	"2" : {value: "E", writable: false, enumerable: true, configurable: false},
	length: {value: 3, writable: false, enumerable: false, configurable: false }
}
```

만약 String래퍼객체가 읽기전용 객체가 아니라면 변경된 String 래퍼 객체를 문자열로 되돌릴때 문자열이 변경된다. 따라서 String객체의 모든 메서드는 String래퍼 객체를 직접 변경할 수 없고 String 객체의 메서드는 언제나 새로운 문자열을 생성하여 반환한다.

### String.prototype.indexOf

indexOf 메서드는 대상 문자열(메서드를 호출한 문자열)에서 인수로 전달받은 문자열을 검색하여 첫번째 인덱스를 반환한다. 검색에 실패하면 -1을 반환한다.

(정확히 표현하면 String래퍼 객체가 맞지만 결국 String래퍼 객체는 문자열을 다루기 위한 임시 객체이므로 앞으로 문자열로 표현)

```tsx
const str = "Hello World"
str.indexOf("l") // 2
str.indexOf("or") // 7
str.indexOf("x") // -1
```

indexOf의 2번쨰 인수로 검색을 시작할 인덱스를 전달할 수 있다.

```tsx
str.indexOf("l", 3) // 3
```

indexOf메서드는 대상 문자열에 특정 문자열이 존재하는지 확인할 떄 유용합니다.

```tsx
if (str.indexOf("Hello") !== -1) {
}
```

ES6에서 도입된 String.prototype.includes 메서드를 사용하면 가독성이 더 좋다.

```tsx
if (str.includes("Hello") {}
```

### String.prototype.search

search메서드는 대상 문자열에서 인수로 전달받은 정규 표현식과 매치하는 문자열을 검색하여 일치하는 문자열의 인덱스를 반환한다. 검색에 실패하면 -1을 반환한다.

```tsx
const str = "Hello World"
str.search(/o/) // 4
str.search(/x/) // -1
```

### String.prototype.includes

ES6에서 도입된 includes 메서드는 대상 문자열에 인수로 전달받은 문자열이 포함되어 있는지 확인하여 그결과를 true 또는 fasle로 반환한다.

```tsx
const str = "Hello World"
str.includes("Hello") // true
str.includes("") // true
str.includes("x") // false
str.includes() // false
```

includes메서드의 2번째인수로 검색을 시작할 인덱스를 설정할 수 있습니다.

```tsx
const str = "Hello World"
str.includes("l", 3) // true
str.includes("H", 3) // false
```

### String.prototype.startsWith

ES6에서 도입된 startsWith 메서드는 대상 문자열이 인수로 전달받은 문자열로 시작하는지 확인하여 그 결과를 true 또는 false로 반환한다.

```tsx
const str = "Hello World"
str.startsWith("He") // true
str.startsWith("x") // false
```

startsWith 메서드의 2번째 인자로 검색을 시작할 인덱스를 전달할 수 있다.

```tsx
start.startsWith(" ", 5) // true
```

### String.prototype.endsWith

ES6에서 도입된 endsWith 메서드는 메서드 대상 문자열이 인수로 전달받은 문자열로 끝나는지 확인하여 그 결과를 true혹은 false로 반환한다.

```tsx
const str = "Hello World"
str.endsWith("ld") // true
str.endsWith("x") // false
```

endsWith메서드의 2번째 인수로 검색할 문자열의 길이를 전달할 수 있다.

```tsx
str.endsWith("lo", 5) // true
// 문자열 str의 처음부터 5자리까지 ('Hello') 가 'lo'로 끝나는지 확인
```

### String.prototype.charAt

charAt 메서든느 대상 문자열에서 인수로 전달받은 인덱스에 위치한 문자를 검색하여 반환한다.

```tsx
const str = "Hello"
for (let i = 0; i < str.length; i++) {
  console.log(str.charAt(i)) // H e l l o
}
```

인덱스는 문자열의 범위 즉 0~문자열의길이-1 사이의 정수이어야한다. 인덱스가 문자열의 범위를 벗어난 정수인 경우 빈 문자열을 반환한다.

```tsx
str.charAt(5) // ''
```

charAt 메서드와 유사한 문자열 메서드는 String.prototype.charCodeAt 과 String.prototype.codePointsAt이 있다.

### String.prototype.substring

substring메서드는 대상 문자열에서 첫번째 인수로 전달받은 인덱스에 위치하는 문자부터 두번째 인수로 전달받은 문자 바로 이전의 문자까지의 부분 문자열을 반환한다.

```tsx
const str = "Hello World"
str.substring(1, 4) // ell
```

substring 메서드의 두번째 인수는 생략 가능하다. 이때 첫번째 인수로 전달한 인덱스에 위치하는 문자부터 마지막 문자까지 부분 문자열을 반환한다.

```tsx
const str = "Hello World"
str.substring(1) // 'ello World'
```

substring 메서드의 첫번째 인수는 두번째 인수보다 큰 정수이어야 정상이다. 하지만 다음과 같이 인수를 전달하여도 동작한다.

- 첫번째 인수 > 두번째 인수인 경우 두 인수는 교환된다.
- 인수 < 0 또는 NaN인 경우 0으로 취급된다.
- 인수 > 문자열의 길이 (str.lenght)인 경우 인수는 문자열의 길이로 취급된다.

```tsx
const str = "Hello World"
str.substring(4, 1) // ell
str.substring(-2) // 'Hello World'
str.substring(1, 100) // 'ello World'
str.substring(20) // ''
```

String.prototype.indexOf 메서드와 함께 사용하면 특정 문자열을 기준으로 앞뒤에 위치한 부분 문자열을 취득할 수 있습니다.

```tsx
const str = "Hello World"
str.substring(0, str.indexOf(" ")) // 'Hello'
str.substring(str.indexOf(" ") + 1, str.length) // 'World'
```

### String.prototype.slice

slice메서드는 substring메서드와 동일하게 동작한다. 단 slice메서드에는 음수인 인수를 전달할 수 있다. 음수인 인수를 전달하면 대상 문자열의 가장 뒤에서부터 시작하여 문자열을 잘라내어 반환나다.

```tsx
const str = "Hello World"
str.substring(0, 5) // Hello
str.slice(0, 5) // Hello
str.substring(2) // llo World
str.slice(2) // llo World
// 인수 < 0 또는 NaN인 경우 0으로 취급된다.
str.substring(-5) // Hello World
// slice 메서드는 음수인 인수를 전달할 수 있다.
str.slice(-5) // 'World'
```

### String.prototype.toUppercase

```tsx
const str = "Hello World"
str.toUpperCase() // 'HELLO WORLD'
```

### String.prototype.toLowercase

```tsx
const str = "Hello World"
str.toLowercase() // hello world
```

### String.prototype.trim

trim메서드는 대상 문자열 앞 뒤에 공백문자가 있을 경우 이를 제거한 문자열을 반환한다.

```tsx
const str = "      foo "
str.trim() // 'foo'
```

String.prototype.replace 메서드에 정규 표현식을 인수로 전달하여 공백문자를 제거할 수도 있다.

```tsx
const str = "      foo "
str.replace(/\s/g, "") // 'foo'
str.replace(/^\s+/g, "") //'foo  '
str.replace(/\s+$/g, "") // '    foo'
```

### String.prototype.repeat

ES6에서 도입된 repeat메서드는 대상 문자열을 인수로 전달받은 정수만큼 반복해 연결한 새로운 문자열을 반환한다. 인수로 전달받은 정수가 0이면 빈 문자열을 반환하고 음수이면 RangeError 를 발생시킨다. 인수를 생략하면 기본값이 0이 설정된다.

```tsx
const str = "abc"
str.repeat() // ''
str.repeat(0) // ''
str.repeat(1) // 'abc'
str.repeat(2) // 'abcabc'
str.repeat(2.5) //'abcabc' (2.5 -> 2)
str.repeat(-1) // RangeError: Invalid count value
```

### String.prototype.replace

replace메서드는 대상 문자열에서 첫번째 인수로 전달받은 문자열 또는 정규표현식을 검색하여 두번째 인수로 전달한 문자열로 치환한 문자열을 반환한다.

```tsx
const str = "Hello World"
str.replace("World, "Lee") // "Hello Lee"
```

검색된 문자열이 여럿 존재할 경우 첫번째로 검색된 문자열만 치환한다.

```tsx
const str = "Hello World World"
str.reaplce("World", "Lee") // 'Hello Lee World'
```

특수한 교채 패턴을 사용할 수 있다. 예를 들어 $&는 검색된 문자열을 의미한다. 교체 패턴에 대한 자세한 내용은 MDN의 함수 설명을 참고.

```tsx
const str = "Hello world"
str.replace("world", "<strong>$&</strong>") //'hello <strong>world</strong>'
// 특수한 교체패턴을 사용할 수 있다. ($& -> 검색된 문자열)
```

replace의 첫번째 인수로 정규표현식을 전달할 수도 있다.

```tsx
const str = "Hello Hello"
str.replace(/hello/gi, "Lee") // 'Lee Lee'
```

replace메서드의 두번째 인수로 치환함수를 전달할 수 있다. replace메서드 첫번째 인수로 전달한 문자열 또는 정규표현식에 매치한 결과를 두번째 인수로 전달한 치환함수의 인수로 전달하면서 호출하고 치환함수가 반환한 결과와 매치 결과를 치환한다.

예를들어서 카멜케이스를 스네이크케이스로,스네이크 케이스를 카멜 케이스로 변경하는 함수는 다음과 같다.

```tsx
function camelToSnake(camelCase) {
  // /.[A-Z]/g는 임의의 한 문자와 대문자로 이루어진 문자열에 매치된다.
  // 치환함수의 인수로 매치 결과가 전달되고, 치환함수가 반환한 결과와 매치 결과를 치환한다.
  return camelCase.replace(/.[A-Z]/g, (match) => {
    console.log(match) // 'oW'
    return match[0] + "_" + match[1].toLowerCase()
  })
}

const camelCase = "helloWorld"
const snake = camelToSnake(camelCase)
console.log(snake) // 'hello_world'

function snakeToCamel(snakeCase) {
  // /_[a-z]/g 는 _와 소문자로 이루어진 문자열에 매치
  // 치환함수의 인수로 매치 결과가 전달되고 치환함수가 반환한 결과와 매치 결과를 치환한다.
  return snakeCase.replace(/_[a-z]/g, (match) => {
    console.log(match) // '_w'
    return match[1].toUpperCase()
  })
}
const snakeCase = "hello_world"
const camel = snakeToCamel(snakeCase)
console.log(camel) // 'helloWorld'
```

### String.prototype.split

split 메서드는 대상 문자열에서 첫 번째 인수로 전달한 문자열 또는 정규표현식을 검색하여 문자열을 구분한 후 분리된 각 문자열로 이루어진 배열을 반환한다. 인수로 빈 문자열을 전달하면 각 문자를 모두 분리하고 인수를 생략하면 대상 문자열 전체를 단일 요소로 하는 배열을 반환

```tsx
const str = "How are you doing"
str.split(" ") // ["How" ,"are", "you", "doing"]
// \s 는 여러가지 공백 문자(스페이스, 탭 등)을 의미한다. 즉 , [\t\r\n\v\f]와 같은 의미이다.
str.split(/\s/) // ["How", "are", "you", "doing"]
str.split("") // ["H", "o", "w", " ", "a", "r", "e", " ", "y", "o", "u", " ", "d", "o", "i", "n", "g"]
str.split() // ["How are you doing"]
// 인수를 생략하면 대상 문자열 전체를 단일 요소로 하는 배열을 반환한다.
```

두번째인수로 배열의 길이를 지정할 수 있다. 이거 기억해놓자.

```tsx
str.split(" ", 3) // ["How" , "are" , "you"]
```

split메서드는 배열을 반환한다. 따라서 Array.prototype.reverse, Array.prototype.join메서드와 함께 사용하면 문자열을 역순으로 뒤집을 수 있다.
