# VariableEnvironment & LexicalEnvironment

VariableEnvironmnet에 담기는 내용은 LexicalEnvironment에 담기는 내용과 같지만 **최초 실행시의 스냅샷**을 유지한다는 점이 다릅니다.

**실행컨텍스트를 만들 때 VariableEnvironment에 정보를 먼저 담은 다음 이를 그대로 복새해서 LexicalEnvironment를 만들고, 이후에는 Lexicalenvironment를 사용합니다**.

LexicalEnvironment 내부는 enviromentRecord와 outer-EnvironmentReference로 구성돼 있습니다.

enviromentRecord에는 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장됩니다. 컨텍스트를 구성하는 함수에 지정된 매개변수 식별자, 선언한 함수가 있을 경우 그함수자체,변수식별자등이 식별자에 해당합니다. 컨텍스트 내부전체를 처음부터 끝까지 쭉 훑어나가면서 순서대로 수집합니다.

전역 실행 컨텍스트는 변수 객체를 생성하는 대신 자바스크립트 구동환경이 별도로 제공하는 객체, 즉 전역 객체를 활용합니다. 전역 객체에는 브라우저의 window, node의 global 객체 등이 있습니다. 이들은 자바스크립트 내장 객체가 아닌 호스트 객체로 분류됩니다.

호이스팅은 자바스크립트 엔진이 실제로 변수를 끌어올리진 않지만 편의상 끌어올린다고 간주하는 것입니다. 호이스팅에서 함수는 선언 전체 즉 function() {}이 부분 전체가 끌어올려지는 것으로 생각하자.

environmentRecord에는 매개변수의 이름, 함수선언, 변수명이 담깁니다.

함수선언문과 표현식. 이둘 모두 함수를 새롭게 정의할 때 쓰이는 방식인데, 선언문은 function 정의부만 존재하고 별도의 할당 명령이 없는 것을 의미하고, 반대로 함수 표현식은 정의한 function을 별도의 변수에 할당하는 것을 말합니다.

함수 선언문의 경우 반드시 함수명이 정의돼 있어야하는 반면, 함수 표현식은 없어도 됩니다. 함수명을 정의한 함수 표현식을 기명 함수 표현식, 정의하지 않은 것을 익명함수 표현식이라고 부르기도 하는데 일반적으로 함수 표현식은 익명 함수 표현식을 말합니다.

```jsx
function a() {} // 함수 선언문 . 함수명 a가 곧 변수명

let b = function () {}; // 익명 함수 표현식 변수명 b가 곧 함수명
b();

let c = function d() {}; // 기명 함수 표현식. 변수명은 c, 함수명은 d
c(); // 실행 ok
d(); // 실행되지 않음 에러.
```

기명 함수 표현식은 주의해야 할 점이 존재하는데 바로 외부에서 함수명으로 함수를 호출할 수 없다는 점 입니다. 함수명은 오직 함수 내부에서만 접근할 수 있습니다. 그렇다면 기명 함수 표기식에서함수명은 어떤 용도로 쓰일까요? 기명함수 표현식은 디버깅시 어떤 함수인지 추적하기에 익명함수 표현식보다 유리한 측면이 존재합니다. 그러나 이제는 모든 브라우저들이 익명함수 표현식의 변수명을 함수의 name 프로퍼티에 할당해서 이제 의미가 딱히 없습니다. 함수 c 내부에서는 c()든 d()든 호출이 잘 됩니다. 따라서 함수 내부에서 재귀함수를 호출하는 용도로 함수명을 쓸 수 있습니다.

함수 선언문은 전체를 호이스팅하고 함수 표현식은 변수 선언부만 호이스팅합니다. -> function a () { } 이건 다 호이스팅 된다는 말.
함수도 하나의 값으로 취급할 수 있다는 것이 바로 이런 것 입니다. 함수를 다른 변수에 값으로써 할당한것이 곧 함수 표현식입니다. 여기서 함수 선언문과 함수 표현식의 차이가 발생합니다.

스코프란 식별자에 대한 유효 범위입니다. 어떤 경계 A의 외부에서 선언한 변수는 A의 외부 뿐 아니라 A의 내부에서도 접근이 가능하지만 A의 내부에서 선언한 변수는 오직 A의 내부에서 접근할 수 있습니다. es5까지 JavaScript에서는 오직 함수에 의해서만 스코프가 생성되는 문제가 있었습니다.

식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것을 스코프 체인이라고 합니다. 그리고 이를 가능하게 하는 것이 바로 LexicalEnvironment의 두번째 수집 자료인 outerEnvironmentReference 입니다.

outerEnvironmentReference는 현재 호출된 함수가 선언될 당시의 LexicalEnvironment를 참조합니다. 과거 시점인 선언될 당시에 주목해야합니다.선언하다라는 행위가 실제로 일어날 수 있는 시점이란 콜 스택상에서 어떤 실행 컨텍스트가 활성화된 상태일 뿐 입니다. 어떤 함수를 선언하는 행위 자체도 하나의 코드에 지나지 않으며 모든 코드는 실행 컨텍스트가 활성화 상태일 때 실행되기 때문입니다.

예를 들어 A함수 내부에 B함수를 선언하고 다시 B 함수 내부에 C함수를 선언한 경우, 함수 C의 outerEnvironmentReference는 함수 B의 LexicalEnviromnet를 참조합니다. 함수 B의 LexicalEnvironment에 있는 outerEnvironmentReference는 다시 함수 B가 선언되면 A의 LexicalEnviroment를 참조합니다. 이처럼 outerEnviromentReference는 연결리스트형태를 띱니다. 선언시점의 LexicalEnviroment를 계속 찾아 올라가면 마지막에는 전역 컨텍스트의 LexicalEnvironment가 존재합니다. 또한 각 OuterEnviromentReference는 오직 자신이 선언된 시점의 LexicalEnviroment만 참조하고 있으므로 가장 가까운 요소부터 차례대로만 접근할 수 있고 다른순서로 접근하는 것은 불가능합니다. 이런 구조적 특성 때문에 여러 스코프에서 동일한 식별자를 선언한 경우에는 무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능하게 됩니다.

inner 스코프의 LexicalEnviroment에 a가 존재한다고 합시다. 이렇게되면 스코프 체인 검색을 더 진행하지 않고 즉시 a를 반환합니다. 즉 함수 내부에서는 a를 선언했기 때문에 전역 공간에서 선언한 동일한 이름의 a변수에는 접근할 수 없는 셈입니다. 이를 변수 은닉화 (variable shadowing) 이라고 합니다. 실행 컨텍스트의 this binding 에는 this로 저장된 객체가 저장됩니다. 실행 컨텍스트 활성화 당시에 this가 지정되지 않은 경우 this에는 전역 객체가 저장됩니다. 그밖에는 함수를 호출하는 방법에 따라 this에 저장되는 대상이 다릅니다. 그 밖에는 함수를 호출하는 방법에 따라 this에 저장되는 대상이 다릅니다.

실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아놓은 객체입니다. 실행 컨텍스트는 전역 공간에서 자동으로 생성되는 전역 컨텍스트와 eval 및 함수 실행에 의한 컨텍스트 등이 있습니다. 실행 컨텍스트 객체는 활성화 되는 시점에 VariableEnviroment, LexicalEnvironment, ThisBinding의 세가지 정보를 수집합니다.

전역 변수의 사용을 최소화하는데 도움을 주는 다양한 방법이 존재합니다. 몇가지만 소개하자면, 즉시실행함수활용, 네임스페이스, 모듈패턴, 샌드박스 패턴등이 대표적입니다. 그밖에도 모듈관리도구인 AMD나 CommonjS, ES6의 모듈등도 모두 이와 관련한 역할을 합니다.

실행 컨텍스트를 생성할 때는 VariableEnvironment와 LexicalEnviroment가 동일한 내용으로 구성되지만 LexicalEnvironment는 함수 실행 도중에 변경되는 사항이 즉시 반영되는 반면 VariableEnviroment는 초기 상태를 유지합니다. 둘다 매개변수명, 변수의 식별자, 선언한 함수명등을 수집하는 environmentRecord와 바로 직전 컨텍스트의 LexicalEnviroment 정보를 참조하는 outerEnviromentReference로 구성돼 있습니다.

호이스팅은 코드 해석을 좀더 수월하게 하기 위해서 enviromentRecord의 수집 과정을 추상화한 개념으로 실행 컨텍스트가 관여하는 코드집단의 최상단으로 이들을 끌어올린다고 해석하는 것입니다. 변수 선언과 값할당이 동시에 이뤄진 문장은 선언부만을 끌어올린다고 해석하는 것입니다. 할당과정은 원래 자리에 남아있게되는데, 여기서 함수 선언문과 함수 표현식의 차이가 발생합니다.

스코프는 변수의 유효범위를 말합니다. outerEnviromentReference는 해당 함수가 선언된 위치의 LexicalEnviroment를 참조합니다. 코드상에서 어떤 변수에 접근하려고 하면 현재 컨텍스트의 LexicalEnviroment를 탐색해서 발견되게 되면 그 값을 바로 반환하고 (변수의 은닉 variable shadowing) 찾지 못할 경우 다시 outerenvironmentReference에 담긴 LexicalEnvironment를 탐색하는 과정을 거칩니다. 전역 컨텍스트의 LexicalEnvironment까지 탐색해도 해당 변수를 찾지 못하면 undefined를 반환합니다.

전역 컨텍스트의 LexicalEnviromnet에 담긴 변수를 전역변수라고 하고 그밖의 함수에 의해 생성된 실행 컨텍스트의 변수들은 모두 지역변수입니다. 안전한 코드 구성을 위해 가급적 지역 변수의 사용은 최소화하는 것이 좋습니다.

this에는 실행컨텍스트를 활성화하는 당시에 지정된 this가저장됩니다. 함수를 호출하는 방법에 따라 그 값이 달라지는데 지정되지 않은 경우에는 전역 객체가됩니다.
