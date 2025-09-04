# 폴더 구조

`src`

`ㄴ pages`

`ㄴ index.tsx`

`ㄴ _app.tsx`

`ㄴ _document.tsx`

## src/pages

page의 역할을 할 컴포넌트들을 보관하고, 파일들을 경로에 맞게 보관할 수 있다.

## index.tsx

웹 애플리케이션의 홈페이지 역할을 하는 컴포넌트 파일로, 이 파일에서 default로 내보내는 컴포넌트들은 Home이라는 이름을 가진다. 해당 컴포넌트는 / 경로로 연결된 페이지로 작동한다.

사용자가 웹사이트의 루트 경로로 접속하면, index.tsx 파일에 있는 Home 컴포넌트가 렌더링되며, 이 컴포넌트가 return 하는 HTML 요소들이 브라우저에 표시된다.

## \_app.tx

모든 페이지의 부모 컴포넌트 역할을 수행한다.

- 주요 props
  - **Component**
    현재 렌더링될 페이지 컴포넌트.
    특정 경로로 이동할 때 렌더링되어야 하는 페이지가 전달된다.
  - **pageProps**
    Component에게 전달될 page의 props들을 모두 객체로 보관해 둔 props로, 페이지에 필요한 데이터를 담고 있다.

App 컴포넌트에서는 두 props를 받아 return 문에서 현재 페이지(Component)를 렌더링하고, 동시에 pageProps를 해당 페이지에 구조 분해 할당을 통해 전달하는 구조로 동작한다.

## \_document.tsx

HTML 문서의 최상위 구조를 정의하는 역할을 수행한다. 모든 페이지에 공통적으로 적용되는 HTML 태그와 설정을 처리한다. (리액트의 index.html 과 유사한 역할을 수행함.)
