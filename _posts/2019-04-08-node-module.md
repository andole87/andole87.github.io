# Modules

노드의 모듈 시스템에서는, 각 파일은 분리된 모듈로 취급된다.  
예를 들면, foo.js라는 파일을 생각해 보자.  
```javascript
foo.js
const circle = require('./circle.js');
console.log('The area of a scircle of radius 4 is ${circle.area(4)}');
```

첫 라인에서, `foo.js`가 `circle.js` 모듈을 로드한다. `circle.js`는 `foo.js`와 같은 디렉토리에 있다.

```javascript
circle.js

const {PI} = Math;

exports.area = (r) = > PI * r ** 2;
exports.circumference = (r) => 2 * PI * r;
```

circle.js 모듈은 함수 `area()`, `circumference()`를 export하고 있다.  
함수와 객체는 exports 객체에 추가 명세한 대로, root of modules에 추가된다.

각 모듈에 있는 변수들은 private이다. 왜냐하면 모듈은 wrapped되어 있기 때문이다. 여기서는 PI 변수가 private이다.  

`module.exports` 속성은 새 값을 할당할 수 있다. (함수나 객체처럼)  
다음 `bar.js`는 Square class를 export하는 square 모듈에 대한 것이다.  

```javascript
bar.js

const Squre = require('./square.js');
cons mySquare = new Square(2);
console.log('The area of mySquare is ${mySquare.are()}');
```

```javascript
square.js

module.exports = class Square {
    constructor(width){
        this.width = width;
    }

    area() {
        return this.width ** 2;
    }
};
```
한 파일이 node.js로부터 직접 실행되면, `requrie.main` 은 그것의 module로 정해진다. 즉, `require.main === module`으로 어떤 파일이 직접 실행되는지 테스트할수 있다.  

`node foo.js'`는 참이지만,  
`require('./foo')`는 거짓이다.

### 총평
비슷한 인터프리터 언어인 파이썬과 비교해보겠다.

### 파이썬 모듈 시스템
파이썬은 `import`를 실행하는 쪽에서 선택이 가능하다. `from foo import bar as guido`처럼. 자유도는 높지만 캡슐화가 어렵다. 큰 프로젝트인 경우 private, public을 미리 약속하고 사용해야 한다.

### 노드 모듈 시스템
노드에서는 파일 그 자체가 아니라, `module.exports`라는 전역객체를 통해 모듈시스템이 동작한다. 모듈이 되는 파일에서 어떤 정보를 `exports`에 등록해두지 않으면, 모듈 호출자는 접근할 수 없다. (위 예시에서 `PI`처럼)   
모듈을 만들때마다 `exports`를 명세해줘야 하는 귀찮음이 있지만, 큰 시스템을 설계하고 개발하는데에는 이점이 분명하다.  
