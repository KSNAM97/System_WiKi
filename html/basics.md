# HTML 기초 (Hyper Text Markup Language)

## HTML(Hyper Text Markup Language)이란

HTML이라는 이름은 **HyperText**, **Markup**, **Language** 세 가지 개념이 합쳐진 것이다. 각각의 의미를 이해하면 HTML이 무엇을 위한 언어인지 파악할 수 있다. 웹 페이지의 제목, 단락, 이미지, 링크, 입력 폼 등을 배치하는 구조를 정의하는 데 쓰이며, 정적인 소개 페이지부터 서버사이드 템플릿, 이메일 뉴스레터에 이르기까지 폭넓게 활용된다.

### HyperText

- HyperText는 하이퍼링크를 통해 문서와 문서를 연결하는 기능을 제공
- 예를 들어, 우리가 웹사이트에서 특정 단어를 클릭하면 다른 페이지로 이동할 수 있는 것이 바로 HyperText의 특징이다.
  이런 연결은 `<>` 태그를 사용하여 구현된다.

### Markup

- Markup은 텍스트에 구조를 부여하거나 표시하기 위해 사용
- HTML에서는 특정 태그로 감싸서 구조를 정의

예: `<p>`는 문단을 나타내고, `<h1>`은 제목을 나타낸다.

### Language

HTML은 사람이 작성한 코드를 웹 브라우저가 해석하고 표현하는 언어이다. 마치 사람이 사용하는 언어와 문법처럼, HTML에도 고유의 문법과 규칙이 존재한다.

- 우리가 웹사이트에서 특정 글자나 버튼을 눌러서 다른 사이트로 이동하게 되는데 이런 문서들을 Hyper Text라고 한다.
- HTML는 콘텐츠를 표시하는 방법으로 웹페이지를 만들게 된다.
  A위치에는 이미지가 보여야 하고 B위치에는 텍스트가 보여야 한다.
- 사람과 웹브라우저 사이에서 사용하는 언어
- 웹사이트를 만들기 위해서 개발자가 할 일은 HTML 코드로
  웹페이지에 어떤 내용을 표기할지 HTML문서를 만드는 일이다.
- 완성된 코드를 웹브라우저에서 로딩하게되면 웹페이지가 만들어 진다.
- HTML 코드가 웹브라우저를 통해서 해석되고 표현되는 과정을 렌더링이라고 한다.

### HTML 문서 만들기

- HTML 문서는 파일의 확장자가 html이다.
- HTML 파일을 만들거나 수정하려면 텍스트 편집기를 사용해야 한다.
- 대표적인 텍스트 편집기 : Brackets, VScode등...
- 결과를 확인하기 위해서는 웹브라우저를 사용해야 한다.
- 대표적인 웹브라우저 : 크롬, 파이어폭스, 엣지, 사파리등...

### 개발자 도구

웹브라우저에는 개발자 도구가 탑재되어 있다.

- 개발자 도구란?
- 웹사이트 개발용 도구로 대부분의 최신 웹브라우저에는 개발자 도구가 탑재되어 있다.
- HTML, CSS 코드확인, 모바일 모니터링, 네트워크 상태점검, 스크립트 명령어 확인등
  다양한 기능을 통해서 개발자에게 편의를 제공한다.

### 코드 에디터 (텍스트 편집기)

- 코드 에디터란?
- 물론 메모장으로도 코드를 편집할 수 있지만 여러 가지 기능을 지원하는 코드 에디터를 일반적으로 많이 사용한다.
- 프로그래머가 프로그램 소스 코드를 편집하기 위해서 사용하는 소프트웨어
- 코드는 결국 텍스트다 그러나 이 텍스트를 더 빠르게, 더 편하게 작성하기 위해서는 코드 에디터를 사용하는 것이 좋다.
  (텍스트 자동완성, 하이라이팅 같은 많은 기능이 추가된 메모장이라고 생각하자)

대표적인 편집기:

- **Brackets** : 초보자에게 적합하며 간단한 인터페이스 제공.
- **Visual Studio Code(VSCode)** : 다양한 플러그인과 높은 확장성 제공.
- **Notepad++** : 가볍고 빠른 실행 속도를 제공.

### HTML은 언어이다.

- 우리가 한국어, 영어 등으로 사람들과 소통하기위해서는 해당 언어의 문법에 맞는 표현을 사용해야 하듯
  HTML 언어를 목적에 맞게 사용하기 위해서는 HTML의 문법에 맞는 표현을 사용해야 한다.
- HTML의 문법은 딱 한가지 태그(tag)만 기억하면 된다.
- 태그란? : HTML코드에서 정보(컨텐츠)를 정의하는 형식

### tag (시작과 끝)

- 태크는 `<>`과 `</>` 기호를 사용하여 컨텐츠의 시작과 끝을 표시한다.
- 각 태그는 콘텐츠를 감싸며 태그명은 콘텐츠의 성격과 의미를 나타낸다.

EX)
```html
<태그명>이곳에 콘텐츠를 기입한다.</>
```

### tag (단일 태그)

- 태크는 `<>`과 `</>` 기호를 사용하여 컨텐츠의 시작과 끝을 표시하지만
  경우에 따라 시작과 끝을 구분할 필요가 없는 태그도 존재한다.
  이를 단일태그라고 한다.
- 단일 태그는 콘텐츠를 감쌀 필요가 없기 때문에 콘텐츠를 감쌀 필요가 없다.

EX)
```html
<태그명/>
```

### tag (속성)

- 속성은 태그의 부가적인 기능을 정의하는 것이다. (선택사항)
- 속성은 시작 태그의 내부에서 정의한다.
- 속성의 개수는 특별한 제한이 없다.
- 태그명과 속성 또는 속성과 속성사이에는 공백(space)으로 구분하며 값은 큰따옴표안에 작성해야 한다.

EX)
```html
<태그명 속성명="속성값"></태그명>
<태그명 속성명="속성값" />
```

### HTML (주석처리)

- 주석은 사람에게는 보이지만 웹브라우저에게는 보이지 않는 코드이다.
- 주로 코드에 대한 메모를 남기기 위한 용도로 사용된다.

EX)
```html
<!-- 해당코드 안의 모든 코드는 처리되지 않는다. -->
```

**정리**: HTML은 HyperText(문서 연결), Markup(구조 표시), Language(문법 체계)가 합쳐진 개념이며, 태그의 시작/끝, 단일 태그, 속성, 주석 처리 방식이 HTML 문법의 기본 골격이다.

---

## HTML 기본 구조

HTML 문서는 `<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`라는 정해진 골격을 갖는다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>제목 작성t</title>
</head>

<body>
    
</body>
</html>
```

### `<!DOCTYPE html>`

- 문서의 첫 부분에서 유형을 지정하는 단일 태그이다.
- HTML은 시간이 흐르면서 버젼이 변경되었는데 현재 표준으로 사용되는 HTML5 버전의 문서를 의미
- 이 선언은 문서의 가장 첫 번째 줄에 있어야 하며, 브라우저가 HTML 문서를 렌더링할 때
  어떤 표준을 사용할지 알려준다.

### `<html></html>`

- `<html>` 태그는 HTML 문서의 루트(root) 요소
- `<html>` 태그부터 HTML 문서가 시작되고 `</html>`에서 HTML문서가 끝난다.
- 모든 HTML 요소는 이 태그 내부에 포함됩니다.
- `lang="en"` 속성은 문서의 언어를 영어로 설정
- 이 속성은 검색 엔진과 접근성 도구가 문서를 올바르게 이해하도록 도와준다.

### `<head></head>`

- 웹브라우저 화면에는 보이지않지만 웹브라우저가 알아야할 정보들을 입력시 사용되는 태그이다.
- `<meta charset="utf-8">` : UTF-8은 대부분의 언어를 지원하며, 웹에서 가장 널리 사용되는 인코딩 방식
- 문서의 제목, 문자 인코딩, 스타일 시트 및 스크립트 정보를 정의
- `<title></title>` : 문서의 제목을 나타낸다. 컨텐츠는 브라우저의 탭에 표시된다.

### `<body></body>`

- `<body>` 태그는 문서의 본문(content)을 정의한다.
- 실제 브라우저 화면에 표시될 내용을 입력하는 태그이다.
- body태그에는 아래의 태그들이 포함될 수 있다.
  - 텍스트를 표시하는 태그
  - 이미지를 표시하는 태그
  - 각종 사용자 인터페이스(버튼 , 입력란 , 다운로드 메뉴등...)를 나타내는 태그
- 태그안에 태그가 포함되면 포함 관계를 알려주기위해서 들여쓰기를 사용한다.

### `<!DOCTYPE html>` (보충 설명)

- 문서의 처 부분에서 문서 유형을 지정하는 단일 태그이다.
- 이때 문서 유형이란 웹브라우저에게 "이 문서는 XX이니까 잘 처리해줘"라는 메시지를 전달하는 개념이다.
- HTML은 첫 등장 후 시간이 흐르면서 버전을 변경해왔는데
  현재 표준으로 사용되고 있는 HTML버전을 사용하기위해서 적어주는 타입이 html이다.

### `<html></html>` (보충 설명)

- 문서 유형을 지정한 후 실제 문서가 시작되고 끝나는 것을 나태내는 태그이다.
- `<html>`에서 문서가 시작되고 `</html>`에서 문서가 끝나게된다.
- 이 태그의 내부에 다양한 태그들이 포함되어 문서의 내용을 구성한다.

### `<head></head>` (보충 설명)

- 웹브라우저 화면에는 보이지 않지만 웹 브라우저가 알아야 할 정보들은 모두 이 태그에 들어간다.

```html
<meta charset="UTF-8">
```
- 문자 인코딩 및 문서 키워드 등에 대한 요약 정보를 기입하는 단일 태그이다.
- 문자 인코딩이란 한글을 표시하기위해 문자 세트를 지정하는 작업으로 영문과 한글을 모두 사용하기 위해
  UTF-8방식을 사용한다.

```html
<title></title>
```
- 문서의 제목을 나타낸다. 콘텐츠는 바라우저 탭에 표시된다.

### `<body></body>` (보충 설명)

- 실제 브라우저 화면에 표시될 내용을 입력하는 태그이다.
- 여기에는 다음과 같은 유형의 태그들이 포함 될 수 있다.
  - 텍스트를 표시하는 태그
  - 이미지를 표시하는 태그
  - 각종 사용자 인터페이스(입력란, 버튼, 드롭다운 메뉴등...)를 나타내는 태그
- 태그안에 태그를 포함하는 방식으로 콘텐츠를 다양하게 구성할 수 있다.

**정리**: HTML 문서는 `<!DOCTYPE html>`로 문서 유형을 선언하고, `<html>`이 전체를 감싸며, 그 안에서 `<head>`는 화면에 보이지 않는 메타 정보를, `<body>`는 실제로 표시되는 콘텐츠를 담당한다.

---

## 텍스트 표시 방법

### 제목 (headline)

`<h>` 태그는 제목을 요소하는 태그이다. `<h>` 태그는 숫자와 함께 사용되며 숫자 1이 가장 크고 숫자 6이 가장 작다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML Study</title>
</head>
<body>
    <h1>h태그는 제목을 표시합니다. h1 </h1>
    <h2>h태그는 제목을 표시합니다. h2 </h2>
    <h3>h태그는 제목을 표시합니다. h3 </h3>
    <h4>h태그는 제목을 표시합니다. h4 </h4>
    <h5>h태그는 제목을 표시합니다. h5 </h5>
    <h6>h태그는 제목을 표시합니다. h6 </h6>
</body>
</html>
```

### 문단(paragraph)

- `<p>` 태그는 문단의 요소를 나타내는 태그로써 가장 많이 사용되는 텍스트 태그이다.
- 하나의 p 태그는 하나의 문단을 표현한다.
- 문단과 문단 사이에는 공백이 있다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML Study</title>
</head>
<body>
    <h1> 윤동주 - 별헤는 밤 </h1>

<p>계절이 지나가는 하늘에는
가을로 가득 차 있습니다.</p>

<p>나는 아무 걱정도 없이
가을 속의 별들을 다 헤일 듯합니다.</p>

<p> 여기는 &nbsp; &nbsp; &nbsp; &nbsp; 솔데스크 학원입니다.</p>

</body>
</html>
```

### 줄바꿈 (break)

- HTML에서는 enter를 사용해도 실제 결과에는 줄바꿈이되지 않는다.
- `<br />` 태그는 줄바꿈을 의미하는 태그이다.
- 공백을 두 번 이상 표시하고자 할때에는 `&nbap;`를 사용한다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML Study</title>
</head>
<body>
    <h1> 윤동주 - 별헤는 밤 </h1>

계절이 지나가는 하늘에는
가을로 가득 차 있습니다. <br>

나는 아무 걱정도 없이
가을 속의 별들을 다 헤일 듯합니다. <br>

가슴속에 하나둘 새겨지는 별을 <br>
이제 다 못 헤는 것은 <br>
쉬이 아침이 오는 까닭이요, <br>
내일 밤이 남은 까닭이요, <br>
아직 나의 청춘이 다하지 않은 까닭입니다.

</body>
</html>
```

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML Study</title>
</head>
<body>
따릅시다 사라지다 재산이 무슨 우선이는 하루가 얼른 데리다. <br> 우선 뺏기라 받칠 설명으로 전화하면 있으러 데뷔작에, 망설인가. 가방이 신문과 되기 주문에 저는 내불 가지다. <br> 7명 깜찍하지 개혁에 먹고 하다. 그것을 아마 <b> 국회가 벌써 방심하여라.</b> 문제는 인간적과 시대의 생전에 흔히 많을 때린 자제는 알다, <br> 때를 얼싸안다. 것 개발할 현재, 언뜻언뜻 같아 전의 내다. <br> 고객의 사람에 찾아보아서 광적을 입어서 되다 빠져나간 기류에 관련의 시집가다. 영향은 응급을 시련의 지나는 전체다 만일이어 하라. <br> <br>

<hr>

찾을 중년은, 물론 이 기념관인 <u> 봉착하니 도 흔하지요.</u> <br> 챙기라 불교를 죄송하면서 <strong>교육법을, 밝혀지다 우리다</strong>  짧다 낮아, 좋다. <s> 제의에 이 있을 풀면서 떨어지다.</s> <br> 그의 지금 밖에서 인간에서 <big>셋째 부담하자</big>, <small> 가지면서, 시를 돋보이다.</small> 넣어서 추상하고 않고 그의 떠오르라 구조는 생각이니 옮기다. <br> <mark> 거짓말을 체험적 땅속을 훼손되어 1일 곱다.</mark> 세련되고 또 규모의 자동차는 얼굴은 이곳을 자유형은 소리에 막연하다. <br> 순수성의 그는 크어요 <em>수준이 세우자 말한 있다</em> 현재 제도로 가지자. <br>

100<sup>2</sup> <br>
100<sub>2</sub>
</body>
</html>
```

위 예제에서 사용된 텍스트 강조/장식용 태그는 다음과 같다.

- `<b>` 태그는 글자를 굵게 표시하는 기능 (표현용)
- Strong 태그 또한 글자를 굵게 보이기는 하지만 실제는 강조의 기능을 담고 있다.
- `<b>` 태그는 글자를 굵게 표현하는 기능이라면 **Strong** 태그는 스크린 리더를 사용하면 더 강조하여 발음된다.
- `<em>` 태그로 감싼 부분은 컨텐츠를 기울여 이탤릭체로 표시하는 인라인 태그이다.
- `<u>` 태그는 밑줄을 의미한다.
- `<s>` 태그는 글의 취소를 의미한다.(del 태그도 같은의미이다.)
- 기본적으로 글자의 크기는 16px로 이루어진다.
- `<big>`, `<small>` 태그를 사용하여 글자의 크기를 조금 키우거나 줄일 수 있다.
- `<sup>` : 윗 첨자
- `<sub>` : 아랫 첨자
- `<mark>` 태그는 형광펜으로 보이게하는 태그이다.
- `<hr>` 태그는 세션을 구분할 때 사용한다.

**정리**: 텍스트는 `<h1>`~`<h6>` 제목, `<p>` 문단, `<br>` 줄바꿈으로 구조화하며, `<b>`/`<strong>`/`<em>`/`<u>`/`<s>`/`<big>`/`<small>`/`<sup>`/`<sub>`/`<mark>`/`<hr>` 등의 인라인 태그로 표현과 의미를 세밀하게 조정할 수 있다.

---

## 태그의 구분 & 인라인 텍스트 요소

### 태그의 구분

블록 레벨 요소를 만드는 태그 vs 인라인 요소를 만드는 태그.

### 블록 레벨 요소 (Block Element)

- 블록 요소는 기본적으로 한 줄 전체를 차지한다.
- 블록 요소 다음에 오는 요소는 자동으로 다음 줄에 배치된다.
- 너비(width), 높이(height), 바깥 여백(margin), 안쪽 여백(padding)을 자유롭게 설정할 수 있다.
- 주로 페이지의 구조나 영역을 만들 때 사용한다.

### 인라인 요소(Inline Element)

- 인라인 요소는 내용의 크기만큼만 영역을 차지한다.
- 다른 인라인 요소가 옆에 들어갈 공간이 있으면 같은 줄에 표시된다.
- 일반적으로 width, height가 제대로 적용되지 않는다.
- 위/아래 방향의 margin도 제한적으로 적용된다.
- 주로 문장 안에서 특정 글자나 부분을 꾸밀 때 사용한다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML Study</title>
</head>
<body>
    <h1> 블럭요소와 인라인 요소</h1>

    <p> p 태그는 블럭요소인가 인라인 요소인가?</p>

    <p>블록 요소는 기본적으로 한 줄 전체를 차지한다</p>

    <p>인라인 요소는 내용의 크기만큼만 영역을 차지한다.</p>

    <strong>스트롱 태그</strong>
    <em>em 태그는 글씨를 기울입니다.</em>
    <mark>mark 태그는 글씨를 형광처리 합니다.</mark>

</body>
</html>
```

**정리**: 블록 요소는 한 줄 전체를 차지하며 크기와 여백을 자유롭게 지정할 수 있고, 인라인 요소는 필요한 만큼만 공간을 차지하며 문장 안에서 부분 장식에 사용된다.

---

## 이미지 태그

### 이미지 표시하기

- img 태그는 이미지를 표시할 때 사용하는 태그이다.
- 단일 태그로써 닫는 태그는 필요하지 않다.
- 컨테츠를 적어주는 대신 표시할 이미지에 대한 정보를 속성으로 지정해야 한다.

기본 형태 : `<img src="표시할이미지경로" alt="이미지 설명" />`

- img 태그의 `src` 속성은 표시할 이미지의 위치 정보와 파일명을 입력받는 속성이다. (이미지의 url을 입력받는다.)
- 서버 또는 자신의 컴퓨터에 저장된 이미지 경로를 사용한다.
- `alt`는 Alternative의 약자로 대체 텍스트 역할을 한다.
- 이미지가 로딩되기 전이나 이미지 딜레이에 의해 로딩에 실패한 경우 이미지 대신에 대체 텍스트가 표시된다.
- alt를 사용하면 이미지를 볼수 없는 시각 장애인에게 웹페이지를 서비스해야하는 상황에대한 대비가 가능하다.
  (음성 인식기가 이미지 대신 이를 활용)

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>HTML 이미지 태그</title>
</head>
<body>
    <img src="../sample_image/sample.jpg" alt="이미지 로딩 실패" width="500" height="300">
    <img src="../sample_image/sample1.jpg" alt="이미지 로딩 실패" width="500" height="300">
</body>
</html>
```

**정리**: `<img>`는 닫는 태그가 없는 단일 태그로, `src`로 이미지 경로를, `alt`로 로딩 실패 시 대체 텍스트 및 접근성 정보를 지정한다.

---

## 컨테이너 태그

### 컨테이너 태그

- 컨테이너(Container) 태그는 여러 HTML 요소를 하나로 묶어 관리하기 위해 사용하는 태그이다.
- 컨테이너 태그 자체는 화면에 특별한 모양을 만들지 않는다.
- 주로 다음과 같은 목적으로 사용한다.
  - 여러 요소를 하나의 그룹으로 묶을 때
  - 웹 페이지의 영역을 구분할 때
  - 여러 요소에 같은 CSS 스타일을 적용할 때
  - 요소의 위치나 배치를 설정할 때
- 대표적인 컨테이너 태그에는 `<div>`와 `<span>`이 있다.

### 블록 레벨 요소(Block-level Element) — `<div>`

- 블록 레벨 요소는 자신이 속한 영역의 가로 너비를 모두 차지한다.
- 블록 요소 앞뒤에는 자동으로 줄바꿈이 발생한다.

```html
<div></div>
```

- div는 Division의 약자이다.
- 웹 페이지의 영역이나 구역을 나눌 때 사용하는 대표적인 컨테이너 태그이다.
- `<div>` 태그 자체에는 기본적인 모양이나 스타일이 없다.
- 주로 CSS와 함께 사용하여 크기, 색상, 위치, 여백 등을 설정한다.

```html
<div>
    <h1>공지사항</h1>
    <p>공지사항 내용입니다.</p>
</div>
```

- 위 코드에서는 `<h1>`과 `<p>`를 하나의 그룹으로 묶었다.
- `<div>`의 주요 용도
  - 헤더 영역 만들기
  - 메뉴 영역 만들기
  - 본문 영역 만들기
  - 사이드바 영역 만들기
  - 푸터 영역 만들기

### 인라인 요소(Inline Element) — `<span>`

- 인라인 요소는 자신에게 필요한 공간만 차지한다.
- 앞뒤에 자동으로 줄바꿈이 발생하지 않는다.
- 다른 글자나 인라인 요소와 같은 줄에 배치된다.

```html
<span></span>
```

- `<span>` 태그는 문장이나 글자의 특정 부분만 묶을 때 사용하는 컨테이너 태그이다.
- `<span>` 태그 자체에는 기본적인 모양이나 스타일이 없다.
- 주로 문장 일부에 CSS 스타일을 적용할 때 사용한다.

```html
<p>오늘 날씨는 <span>맑음</span>입니다.</p>
```

- 위 코드에서는 맑음이라는 글자만 `<span>`으로 묶었다.

```html
<p>오늘 날씨는 <span style="color: red;">맑음</span>입니다.</p>
```

- 맑음이라는 글자에만 빨간색을 적용한다.

### `<div>`와 `<span>`의 차이

| 구분 | `<div>` | `<span>` |
|------|---------|----------|
| 요소 종류 | 블록 요소 | 인라인 요소 |
| 차지하는 공간 | 한 줄 전체 | 필요한 공간만 |
| 자동 줄바꿈 | 발생함 | 발생하지 않음 |
| 주요 용도 | 큰 영역이나 여러 요소를 묶음 | 문장이나 글자의 일부를 묶음 |

### 실행 예시

```html
<div>첫 번째 영역</div>
<div>두 번째 영역</div>

<span>첫 번째 글자</span>
<span>두 번째 글자</span>
```

- `<div>`는 한 줄 전체를 사용하기 때문에 각각 다른 줄에 출력된다.
- `<span>`은 필요한 공간만 사용하기 때문에 같은 줄에 출력된다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Page Title</title>
</head>
<body>
    <div>
        <h1> HTML 문서를 만듭니다.</h1>
        <hr>
        <img src="../image/dog.jpg" width="150" alt="dot image">
    </div>

    <div>
        <h2>오늘 점심은 뭘먹을까?</h2>
        <p>오늘 점심 메뉴를 추천해주세요</p>
        <p>오늘 점심은 <span>간짜장 곱배기</span>를 먹어야겠어요</p>
        <img src="../image/cat.jpg" width="150" alt="dot image">
    </div>
</body>
</html>
```

- 컨테이너를 사용하여 특정 영역을 묶어서 분류하게되면 CSS나 자바스크립트를 사용하여 상호작용을 시킬수 있다.

### 전역 속성(Global attributes)

- 전역속성은 모든 HTML 태그에서 공통을 사용하는 태그이다.
- 속성이란 태그의 부가적인 기능을 정의하는 것으로 선택사항이다.
- 속성은 시작 태그의 내부에 정의한다.
- 속성의 개수에는 제한이 없다.

```html
<태그명 property="value"  property="value">컨텐츠</태그명>
```

대표적인 전역 속성

- `id` : 요소에 고유한 이름을 부여하는 식별자 역할 속성
- `class` : 요소를 그룹별로 묶을수 있는 식별자 역할 속성
- `style` : 요소에 적용할 CSS 스타일을 선언하는 속성
- `title` : 요소의 추가 정보를 제공하는 텍스트 속성 (사용자에게 툴팁을 제공)

**정리**: `<div>`는 영역 전체를 차지하는 블록 컨테이너, `<span>`은 문장 일부만 감싸는 인라인 컨테이너이며, `id`/`class`/`style`/`title` 같은 전역 속성은 모든 태그에 공통으로 적용할 수 있다.

---

## 링크

### 링크(link)

- 링크란 현제 문서에서 다른 문서로 이동하는 수단을 의미한다.

**링크(anchor)**

- a 태그 요소는 href 속성을 통해 다른 페이지, 전화번호 , 이메일 주소와 그외 다른 url로 연결할 수 있는 링크를 만든다.
- anchor는 인라인 요소이며 컨텐츠는 주로 링크의 목적지를 나타낸다.

```html
<body>
    <a href="">컨텐츠</a>
</body>
```

- 웹브라우저에 출력된 Naver에 마우스를 가져가면 이동할수 있다는 의미로 손가락으로 표시된다.
- `<a>`태그안의 href를 속성이라고 한다. (속성은 여러 가지를 사용할 수 있다.)
- href는 hyperlink Reference의 약자이다.(href="" 의 쌍따옴표 안에 경로를 설정하게되면 해당 경로로 이동하게된다.)

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <title>Page Title</title>
    <link rel='stylesheet' href='main.css'>
</head>
<body>
    <div>
        <h1> 네이버로 이동하려면
            <a href="https://www.naver.com" target="_self">클릭</a>
        </h1>

        <h1> 구글로 이동하려면 
            <a href="https://www.google.com" target="_blank">클릭</a>
        </h1>
    </div>

    <div>
        <a href="https://www.naver.com" target="_blank">
            <img src="../image/naver.jpg" alt="dog image" title="네이버 이동" width="200"> <br>
        </a><br>
        <a href="https://www.google.com" target="_blank">
            <img src="../image/google.jpg" alt="cat image" title="구글 이동" width="200">
        </a>
    </div>
</body>
</html>
```

**정리**: `<a href="...">`는 다른 문서/URL로 이동하는 하이퍼링크를 만들며, `target="_self"`/`target="_blank"`로 같은 탭/새 탭 이동 여부를 제어하고, 이미지를 감싸면 이미지 자체가 클릭 가능한 링크가 된다.

---

## 입력 태그 (Form)

### input 태그

- 사용자로부터 값을 입력받을수 있는 대화형 컨드롤(또는 필드)을 라고하며 일반적으로 입력창이라고도 한다.
- 기본적으로 인라인 레벨 요소이며 단일 태그이다.

### 입력요소 input (type 속성)

- input은 type에 따라 입력 요소의 형태나 입력 데이터 유형등이 달라진다.
- 사용 가능한 type은 20가지이며 따로 type을 작성하지 않으면 기본값은 text이다.
- type에 따라 값이 달라지기 때문에 적용가능한 추가 속성의 종류도 달라진다.
- input 태그에는 name 속성의 식별자를 추가할 수 있다.(name값을 사용하여 데이터를 구분할 수 있다.)
  예를들어 input태그의 text가 2개일 때 각각 입력한 값이 다를 경우 name값이 없으면
  두 값을 구분하지 못한다. 각각의 데이터에 대한 식별자로 사용되는 것이 name이다.
  같은 값을 갖은 데이터라고 해도 name 속성을 사용하여 각각의 데이터를 구분할 수 도 있다.

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>Page Title</title>
    <link rel='stylesheet' href='main.css'>
</head>
<body>

    <div>
    <h3>로그인 하세요</h3>
    id 	<input name="id" type="text" placeholder="아이디" autofocus> <br>
    pw 	<input name="passwd" type="password" placeholder="비밀번호"> <br>
    <input type="button" value="로그인">
    <button>회원가입</button>
    </div>

    <hr>

    <div>
       color : 	<input type="color"> <br>
       date : 	<input type="date"> <br>
       range : 	<input type="range" max="100" min="0" step="5"> <br>
       		<button>음량+</button><button>음량-</button> <br>
       phone1 : 	010 <input type="number"> <input type="number"> <br>
       phone2 : 	<input type="number" value="010" readonly> <input type="number"> <input type="number">
    </div>
</body>
</html>
```

### 입력요소 (select, text-area)

#### select

- select는 다수의 옵션을 포함할 수 있는 선택 메뉴이다.
- 메뉴 안에 포함되는 옵션은 option 태그를 사용하여 표시한다.
- 네이버 카페에서 검색시 "제목만 검색", "제목+내용" 검색으로 선택하는 기능과 같다.
- select 태그를 사용하게되면 드롭다운 메뉴가 만들어지고 드롭다운 메뉴를 선택하게되면 선택지가 펼처진다.
- select 태그도 input 태그와 마찬가지로 name 속성을 사용하여 데이터를 구분한다.
- value옵션을 지정할 수 있다. (실제로 처리될 값을 의미한다.)

```html
<!DOCTYPE html>
<html lang='en'>
<head>
    <h1>오늘의 점심 메뉴</h1>
    <select name="menu">
        <option value="" selected disabled>점심 메뉴</option>
        <option value="hambuger">햄버거</option>
        <option value="pasta">파스타</option>
        <option value="pizza">피자</option>
        <option value="jjajang">짜장명</option>
        <option value="jjampong">짬뽕</option>
    </select>
    </div>
    <hr>
    <select name="menu" multiple>
        <option value="" selected disabled>점심 메뉴</option>
        <option value="hambuger">햄버거</option>
        <option value="pasta">파스타</option>
        <option value="pizza">피자</option>
        <option value="jjajang">짜장명</option>
        <option value="jjampong">짬뽕</option>
    </select>
    </div>
</body>
</html>
```

### form태그

- form태그는 양식을 만들어 주는 태그이다.
- form태그를 사용하여 입력한 데이터(입려값)를 서버로 보내기위해 사용하는 태그
- 서버 : 정보나 데이터를 제공하는 host를 의미한다.
- 클리이언트가 데이터를 요청하게되면 서버는 요청한 데이터를 제공한다.

| client | | Web Server |
|---|---|---|
| 1) | --- www.naver.com ---> | |
| 2) | <--- Main page --- | |

- 로그인 또한 같은 방식으로이루어 진다.
- 우리가 ID와 Password를 입력하게되면 해당 서버는 아이디와 비밀번호를 비교하여
  정보가 일치하게되면 로그인 처리가되고 정보가 불일치하게되면 ID와 Password를 다시 입력하라는 창이 열리게된다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>회원가입</title>
</head>
<body>

    <h2>회원가입</h2>

    <form action="membership.html" method="get">

        아이디 :
        <input type="text"
               name="id"
               placeholder="아이디를 입력하세요"
               required
               autofocus>
        <br><br>

        비밀번호 :
        <input type="password"
               name="password"
               placeholder="비밀번호를 입력하세요"
               required>
        <br><br>

        이름 :
        <input type="text"
               name="name"
               placeholder="이름을 입력하세요"
               required>
        <br><br>

        성별 :
        <input type="radio" name="gender" value="남성"> 남성
        <input type="radio" name="gender" value="여성"> 여성
        <br><br>

        전화번호 :
        <input type="text"
               name="phone1"
               value="010"
               size="3"
               readonly>
        -
        <input type="text"
               name="phone2"
               size="4"
               maxlength="4"
               inputmode="numeric">
        -
        <input type="text"
               name="phone3"
               size="4"
               maxlength="4"
               inputmode="numeric">
        <br><br>

        이메일 :
        <input type="email"
               name="email"
               placeholder="example@email.com">
        <br><br>

        주소 :
        <input type="text"
               name="address"
               placeholder="주소를 입력하세요">
        <br><br>

        관심 분야 :
        <input type="checkbox" name="interest" value="네트워크"> 네트워크
        <input type="checkbox" name="interest" value="리눅스"> 리눅스
        <input type="checkbox" name="interest" value="프로그래밍"> 프로그래밍
        <br><br>

        <input type="submit" value="회원가입">
        <input type="reset" value="다시 작성">

    </form>

</body>
</html>
```

**정리**: `<input>`은 `type` 속성값에 따라 텍스트, 비밀번호, 색상, 날짜, 범위, 라디오, 체크박스 등 다양한 입력 형태를 제공하며, `<select>`/`<option>`은 드롭다운 선택 메뉴를, `<form>`은 이 입력 요소들을 묶어 `action`/`method`로 지정한 서버로 데이터를 전송하는 역할을 한다.
