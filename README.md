# javascript-reverse-tracing

## Goal

 - 웹 브라우저의 GET, POST 요청의 특정 파라미터들이 어떻게 만들어지는가?

## Scope

 - 함수내에서 `var`로 선언된 변수는 함수 전체에서 유효하다.
 - 함수내에서 `let`로 선언되 변수는 블럭 안쪽에서만 유효하다.
 - 함수 밖에 선언된 변수는 모든 곳(전역)에서 유효하다.

다음 예제를 보자.

```js
function a() {
  b = 99;
}
a()
console.log(b)
```

```
99
```

`b`는 어디에도 존재하지 않기 때문에 `Implied Globals`로 선언된다.

다음 예제를 보자.

```js
var x = 1;

function foo() {
  var x = 10;
  bar();
}

function bar() {
  console.log(x);
}

foo();
bar();
```

```
1
1
```

`Lexical Scope`는 함수가 어디에서 호출되는지가 아닌 어디에 선언되어있는가에 따라 결정된다.

## Expression Analysis

한 줄 한 줄 분석하기

## Function Analysis

### IN / OUT

#### E1

 - 함수에는 입력과 출력이 있다.
 - 입력이나 출력에 의존하지 않고 Scope에 의존하기도 한다.
 - `IN` 집합은 가능한 모든 입력들이다.
 - `OUT` 집합은 가능한 모든 출력들이다.

```js
a=4
function b(c,d,e) {
c+=1
if (d>0) return e;
return d
}
b(1,2,3)
```

 - `b`함수의 `IN`은 `IN={a,c,d,e}`이다. 
 - `a`는 전역에 선언된 변수이며, `c,d,e`는 `b`의 인자들이다.
 - 따라서 `a,c,d,e`는 가능한 모든 입력들이다.

 - `b`함수의 `OUT`은 `OUT={e,d}`이다.
 - `b`함수의 terminator이 `e,d`를 반환하기 때문이다.
 - `c`가 `OUT`에 속하지 않는 이유는 이미 인자로 선언되어 있기 때문이다.

##### E2

```js
function a(x,y) {
 if (x>y)return x;
 return y;
}
function b(f,x,y) {
 if (x<y) reutrn f(y,x);
 return f(x,y);
}
b(a,0,1)
```

### 어떤 인자가 반환값에 영향을 미치는지?

```js
function foo(a,b,c,d,e) {
 if (a<b) return c;
 return d;
}
```

반환값에 영향을 미치는 변수는 `a,b,c,d`이다.

### 함수 안에서 인자의 값이 변할 수 있는지?

```js
function a(b) {
 b.a = 0;
}
c = {a:1}
a(c)
```

 - 함수 `a`는 `b`의 변수인 `a`를 바꿀 수 있다.
 - 따라서 `c::a`는 `a`에 의해 값이 변할 수 있다.
 - `c` 자체는 변하지 못한다.

### 어떤 인자가 다른 인자에 영향을 주는지?

### 람다 집합

```js
fuction a1(a) {a.a = 1}
fuction a2(a) {a.b = 2}
fuction a3(a) {a.c = 3}

function foo(f1,f2,f3,a,b,d,e,c) {
 if (a<b) f1(c);
 else if (d>b) f2(c);
 else if (e==0) f3(c);
}
c = {a:0,b:0,c:0}
x = a1
if (1>3)
 x = a2
foo(x,a2,a3,4,2,4,5,c)
```

`a1`함수는 인자 `a::a`를 바꿀 수 있다.
`foo`함수의 인자 `f1`의 가능한 람다는 `{a1,a2}`이다.

 - 더불어 인자 `a,b`는 `{f1,f2,f3}`의 실행 여부를 결정한다.
 - `f1={a1,a2}`이고 `f2={a2}`, `f3={a3}`이므로
 - 인자 변경 집합은 `f1={<p1>::a, <p1>::b}`, `f2={<p1>::b}`, `f3={<p1>::c}`이 된다.
 - `f1`은 `a,b`에 영향을 받으므로 `c::a, c::b`는 `a,b`의 영향을 받는다.
 - `c::b`는 또한 `a,b,d`의 영향을 받고, `c::c`는 `a,b,d,e`의 영향을 받는다.

최종적으로 `c::a`는 `a,b`의 영향을, `c::b`는 `a,b,d`의 영향을, `c::c`는 `a,b,d,e`의 영향을 받는다.

### 

## Dyanmic Reverse Tracing

 - 예를들어 `{"token":"asdfasdfasdf"...}`이 `POST` 데이터에 있다고 해보자. 이 요청은 웹 브라우저의 요청이 아닌 `xhr`이다.
 - 그렇다면 요청 데이터의 `token`은 어디서 생산되었을까?

다음 예제를 살펴보자

```js
data = {}
data.token = 'asdfasdfasdf'
xhr.download('http://koromo.xyz', data)
```

사람이라면 두 번째 줄에서 `token`이 생성되었음을 단번에 알 수 있다.
하지만 이를 자동화하려면 어떻게 해야할까?

 - 먼저 요청의 유형을 확인한다. 
 - 만약 `json` 포맷이라면 `json`을 역분석하여 `token`이 있는지를 확인한다.
 - 이제 `xhr.download`의 두 번째 파라미터의 `data::token`을 역추적한다.
