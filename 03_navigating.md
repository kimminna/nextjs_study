# 페이지 이동

### HTML a 태그에 관하여

HTML로 코드를 작성할 때는 페이지 이동을 위해 a 태그를 사용하지만, a 태그는 CSR 방식으로 작동하지 않고, 매번 서버에 새로운 페이지 요청을 보내기 때문에 페이지 이동 시 화면 성능이 저하되는 문제가 있다.

## Link 컴포넌트 사용하기

Next 에서는 Link 컴포넌트를 제공한다. Link 컴포넌트는 CSR 방식으로 페이지를 전환하여 빠르고 부드러운 사용자 경험을 제공한다. next/link에서 import 하여 사용한다.

```tsx
import Link from "next/link";
import type { AppProps } from "next/app";

export default function App({ component, pageProps }: AppProps) {
  return (
    <>
      <header>
        <Link href={"/"}>index</Link>
        <Link href={"/search"}>search</Link>
      </header>
      <Component {...pageProps} />
    </>
  );
}
```

## Programmatic Navigation

특정 이벤트나 함수가 실행될 때 페이지를 이동시키는 것을 프로그래매틱 페이지 이동이라고 한다. 사용자가 직접 링크를 클릭하지 않아도 상황에 맞춰 페이지 전환을 처리한다. 예로, 버튼 클릭, 폼 제출 등에 사용한다.

- **useRouter 훅으로 페이지 이동 처리하는 방법**

      useRouter 훅이 반환하는 router 객체는 페이지를 이동시킬 수 있는 메서드가 포함되어 있다.

      - `push()` - 주어진 경로로 페이지 전환
      - `replace()` - 이전 페이지 기록을 남기지 않고 전환(뒤로가기 방지)
      - `back()` - 뒤로가기

```tsx

  import Link from "next/link";
  import type {AppProps} from "next/app";
  import {useRouter} from "next/router";

export default function
const router = useRouter();
const onClickButton = () => {
router.push("/test");
};

App({component,pageProps}: AppProps) {
return (
<>
<header>
<Link href={"/"}>index</Link>
<Link href={"/search"}>search</Link>
<button onClick={onClickButton}>/test 페이지로 이동</button>
</header>
<Component {...pageProps} />
</>
);
}

```
