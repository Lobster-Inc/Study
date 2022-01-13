# Theia Setup Guide

## macOS

- 준비물: Node.js >= 12.14.1, Yarn

```bash
git clone <https://github.com/eclipse-theia/theia> \\
    && cd theia \\
    && yarn \\
    && yarn download:plugins \\
    && yarn browser build \\
    && yarn browser start

```

- 서버 실행 후에 브라우저를 통해 localhost:3000 번으로 접속하면 된다.
- *애플 M1 기기는 electron 버전 이슈 때문에 빌드가 불가능하다.*

## Dockerfile

```docker
FROM ubuntu:20.04
MAINTAINER "oponeuser7@o.cnu.ac.kr"
WORKDIR /root

RUN export DEBIAN_FRONTEND=noninteractive && \\
	apt-get update -y && \\
	apt install -y git vim curl && \\
	curl -o- <https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh> | bash && \\
	. ~/.bashrc && \\
	nvm install v12.22.7 && \\
	npm i -g corepack && \\
	apt install -y make gcc pkg-config build-essential libx11-dev libxkbfile-dev && \\
	apt install -y libsecret-1-dev && \\
	git clone <https://github.com/eclipse-theia/theia> \\
    && cd theia \\
    && yarn \\
    && yarn download:plugins \\
    && yarn browser build

```

- *무언가 이슈가 있어 도커에서 로컬로의 포트 포워딩이 되지 않는다.*
- 따라서 현재 사용 불가능

---

# 어휘분석

- 어휘분석(Lexical Analysis)는 소스코드를 읽어서 의미를 가진 최소의 단위인 토큰(Token)으로 분리하는 과정이다.
- `y = x+2;` 라는 코드는 어휘분석 시 y, =, x, +, 2, ;의 토큰들로 분리된다.
- if, while 등은 (컴파일러에서 이것들은 statement라고 불리운다) 따로 하나의 토큰들로 인식된다.

# 구문분석

- 구문분석(Syntax Analysis)란 간단히 말하자면 어휘분석을 통해 생성된 토큰들로부터 파스 트리(Parse Tree)를 생성하는 과정이며 이를 파싱이라고 한다. 즉, 파싱은 어휘분석 -> 구문분석의 순서로 이루어진다.
- 구문분석에서는 터미널과 논터미널이라는 개념이 등장하는데, 터미널은 토큰과 같고 논터미널은 또 다른 논터미널의 나열(순열)로 해석된다.
- 즉 ‘program’ 이라는 논터미널은 ‘declaration’이라는 논터미널의 나열로 해석될 수 있으며 ‘declaration’들은 ‘statement’ 또는 ‘expression’의 나열이라는 식으로 재귀적으로 해석될 수 있으며 이는 트리의 개념에 부합한다. (declaration과 expression, statement는 무작위로 선택한 단어가 아니고 컴파일러에서 고유의 의미를 가지는 개념들이지만 나중에 설명하도록 하겠다)

# AST

- AST(Abstract Syntax Tree)란 간단히 말해서 파스 트리에서 괄호와 세미콜론 등의 없어도 그만인 것들을 제외한 트리를 말한다.
- AST는 객체지향적인 디자인을 포함한다. 따라서 트리의 한 노드는 타입, 위치, 자식들에 대한 레퍼런스 등의 Property를 가질 수가 있다.
- AST와 관련된 개념으로는 SDT, SDD 등이 있다. SDT나 SDD나 근본적으로는 AST 노드의 프로퍼티를 이용해서 무언가 의미있는 것을 만들어내는 동작을 말한다. 그리고 이것은 다음부터 설명할 린터와 밀접한 관련이 있다.

# Linter

- 린터는 소스코드를 분석하기 위한 도구이다.
- 린터를 사용하면 소스코드가 사용자가 정의한 규칙에 부합하는지 확인하고 또 부합하도록 수정하는 것이 가능해진다.
- 린터는 리스너(Lintener)와 규칙(Rule)로 이루어지는데, 리스너는 트리를 방문하기 위한 도구라고 생각하면 되고 규칙은 사용자가 정의한 규칙이다.
- 리스너는 AST를 순회하며 규칙이 존재하는 노드마다 해당 노드가 규칙에 부합하는지 확인한다. 그리고 리스너는 규칙에 부합하지 않을 경우에 사용자에게 알리기위한 notification code를 포함한다.
- 간단한 하나의 예로 deprecate된 함수 사용을 찾아내는 규칙을 생각할 수 있다. 리스너는 deprecated function을 모아놓은 collection을 가지고 있다. 그리고 함수 호출 노드를 만났을 때 collection과 비교해서 deprecate된 함수를 사용하고 있지는 않은지 검사하고 만약 그렇다면 알림을 report한다.
- 린터는 AST를 생성한다고는 하지만 이 트리는 소스 코드에 dependent한 부분들도 정보로 가지고 있다. (탭이나 공백이라던지) 따라서 코딩 스타일에 대한 검사도 충분히 가능하다.
- 그리고 린터는 단순한 report의 기능 뿐만 아니라 이상적인 형태로 코드를 변경하는 Fix 기능도 가지고 있다. 자세히는 모르지만 이 구현을 위해서 파스 트리로부터 소스 코드를 생성하는 SDT(Semantic Directed Translation) 기술이 사용되지 않을까 생각한다.

# 정적분석과 동적분석

- 프로그래밍 언어에서 정적, 동적이라고 하면 보통 컴파일 타임과 런타임을 말한다.
- 정적 분석은 프로그램을 컴파일 타임에 분석하는 것이다. C언어 파일을 gcc로 컴파일 할 때에 주로 볼 수 있는 에러들이 그것이다. 린터는 엄밀히 말하자면 컴파일 과정은 아니지만 어찌됬든 프로그램 실행 전이므로 정적분석이다. 즉, 정적분석은 ‘소스코드를 분석하는 행위’로 정의할 수 있겠다.
- 동적 분석은 예상되다시피 프로그램을 런타임에 분석하는 것이다. 런타임에 분석한다는 것은 곧 프로그램이 실행중에 분석한다는 것을 말한다. 쉽게 생각할수 있는 예로는 디버거를 이용한 디버깅이 있다. 프로그램 실행 과정에서 변화하는 상태(변수의 값)을 체크하여 프로그램을 분석한다. 메소드의 동작을 검사하는 단위 테스트(Unit Test)도 동적분석의 일환으로 볼 수 있다. 즉, 분석에 프로그램의 실행이 포함되면 동적분석이라고 생각할 수 있겠다.
