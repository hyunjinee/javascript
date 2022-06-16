# OLOO 패턴에 대한 이해

## 프로토타입 기반 언어

자바스크립트를 한번쯤 공부해봤으면 알 수 있는 것처럼, 자바스크립트는 프로토타입 기반으로 동작하는 언어이다. 즉, 생성자 호출을 통해서 객체를 생성할 때 프로토타입 체인으로 연결되고 프로퍼티와 메서드를 프로토타입을 거슬러 올라가며 찾는 원리를 갖는다. 이러한 언어의 특성으로 볼 때 기존의 객체 지향 언어의 기본 개념인 클래스와 인스턴스는 자바스크립트에 없다는 것을 알 수 있다. 그러나 예전부터 프로토 타입 기반 상속 이라는 말처럼 프로토타입을 기반으로 객체지향을 구현하려는 많은 시도들이 있어 왔고 ES6에서 그러한 노력을 보상하는 것처럼 클래스도 등장하였다. 하지만 그럼에도 불구하고 객체 지향의 상속 및 다형성 등의 핵심 개념들을 완벽하게 구현하는 것은 불가능하다. You don't know JS의 저자인 카일 심슨은 이러한 한계쩜으로 인해 OLOO(Objects Linked to Other Objects)라는 새로운 패턴을 제시하였고 여기서 이를 정리해보고자 한다.

## ES6 이전의 클래스 구현

ES6에서 클래스가 나오기 전에, 프로토타입 기반으로 객체지향 원리를 구현하려는 노력을 알아야 왜 OLOO를 제시했으며 클래스가 나왔는지를 알 수 있다. 먼저 코드를 보자

```js
function Person(name) {
  this.name = name
}

function Student(name, level) {
  Person.call(this, name)
  this.level = level
}

Student.prototype = Object.create(Person.prototype)
Student.prototype.constructor = Student
Student.prototype.showLevel = function () {
  console.log(this.level)
}
Person.prototype.showName = function () {
  console.log(this.name)
}

const stu = new Student("이현진", 1)
stu.showName() // 이현진
stu.showLevel() // 1
```

여기서 Person은 부모 클래스, Student는 자식 클래스가 되도록 만든 것이며 프로토타입 링크를 연결시킴으로서 Student Student로 생성자 호출을 하여도 Person.prototype의 메서드에 접근할 수 있게 되었다. 자세히 살펴보면 아래와 같다.

- Object.create는 새로운 객체를 생성하여 프로토타입 링크인 [[Prototype]]을 인자로 전달된 객체에 연결시킨다. 즉, 본래의 프로토타입 프로퍼티를 버리고 새로운 객체를 생성하기 때문에 원래있던 constructor 프로퍼티가 사라진다.
- 이를 위해서 Student.prototype.constructor = Student를 복구해준다.
- 이후 Student.prototype과 Person.prototype의 메서드를 만들고 Student를 생성자 호출로 하여 새로운 객체 stu를 생성한다.
- stu는 프로토타입 체인을 통해서 Student.prototype의 showLevel과 Person.prototype의 showName을 호출할 수 있게 된다.

여기서 ES6의 Object.setPrototypeOf 메서드로 좀 더 개선할 수 있다.

```js
// Student.prototype = Object.create(Person.prototype);
// Student.prototype.constructor = Student;
Object.setPrototypeOf(Student.prototype, Person.prototype)
```

constructor를 버리지 않고 프로토타입 프로퍼티를 수정하는 메서드이기 때문에 안전하고 직관적이다.

## ES6 이후의 클래스 구현

ES6의 클래스를 사용하면 보다 읽기 쉽고 편하게 구현할 수 있다.

```js
class Person {
  constructor(name) {
    this.name = name
  }

  showName() {
    console.log(this.name)
  }
}

class Student extends Person {
  constructor(name, level) {
    super(name)
    this.level = level
  }

  showLevel() {
    console.log(this.level)
  }
}

const stu = new Student("이름", 1)
stu.showName() // "이름"
stu.showLevel() // 1
```

prototype과 같은 프로퍼티라던가 Object.create나 Object.setPrototypeOf와 같은 메서드를 하나도 사용하지 않고 기존의 객체 지향 언어들과 비슷하게 프로토타입 기반의 상속을 구현하였다. 또한 super 키워드를 통해서 상위 객체의 메서드를 참조할 수 있도록 하였다.

상당히 편리해졌고 가독성도 훨씬 좋아졌다. 하지만, 그 내부에는 여전히 프로토타입 기반으로 동작하고 있으며 실제의 클래스를 구현한 것이 아니라는 것을 반드시 기억해야한다.

## 자바스크립트에서
