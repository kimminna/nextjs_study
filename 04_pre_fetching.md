# Pre-Fetching

## 프리페칭이란?

사전에 페이지를 불러오는 기능을 말한다. 사용자가 현재 보고 있는 페이지에서 **링크를 통해 이동할 가능성이 있는** 모든 페이지를 미리 불러와 빠른 페이지 전환을 가능하게 한다.

Next.js에서는 페이지 전환을 빠르게 처리하기 위해 프리페칭을 제공한다. 사용자가 현재 페이지에서 다른 페이지로 이동하기 전에, 해당 페이지에 필요한 모든 데이터를 미리 불러와 준비해 둔다. 이를 통해 페이지를 이동할 때 서버에 추가로 요청할 필요 없이 빠르게 페이지를 전환할 수 있다.

### Next는 js 코드를 스플리팅하여 저장한다

Next.js는 효율적인 성능을 위해 페이지 별로 JS 코드를 스플리팅하여 미리 저장해 둔다. 즉, 사용자가 특정 페이지에 접속하면 해당 페이지에 필요한 js 코드만 전달된다.

이는 초기 페이지 로드 시 불필요한 코드를 전달하지 않음으로써 TTI를 줄이고, 페이지 로딩 속도를 최적화하기 위함이다.

하지만 다른 페이지로 이동할 때마다 해당 페이지의 JS 코드를 새로 불러와야 하므로 오히려 페이지 전환 속도가 느려질 수도 있다.

따라서 프리페칭을 이용하여 사용자가 현재 보고 있는 페이지에서 이동할 가능성이 있는 모든 페이지의 JS 코드를 미리 번들링한다. 이를 통해 페이지 이동 시 서버에 추가 요청을 보내지 않고, 즉시 페이지를 전환할 수 있다.

### 개발 모드에서는 Pre-Fetching이 작동하지 않는다.

개발 중에는 각 페이지를 서버에서 매번 새로 불러오기 때문에 프리페칭은 작동하지 않는다. 따라서 프로덕션 모드에서 프리페칭을 확인한다.

### 프로그래매틱 네비게이션에서도 Pre-Fetching 적용하기

Link 컴포넌트는 자동으로 프리페칭을 수행하지만, 프로그래매틱 페이지 이동에서는 자동으로 적용되지 않는다.

만약 프로그래매틱 네비게이션에서도 프리페칭을 적용하고 싶다면, useEffect를 사용하여 컴포넌트가 마운트될 때 useRouter의 router 객체의 메소드를 이용해서 직접 프리페칭을 활성화시켜야 한다.

```tsx
import { useRouter } from "next/router";
import { useEffect } from "react";

export default function App() {
  const router = useRouter();

  useEffect(() => {
    router.prefetch("/test"); // /test 페이지를 미리 불러옴
  }, []);

  const onClickButton = () => {
    router.push("/test");
  };

  return (
    <header>
      <button onClick={onClickButton}>Go to Test</button>
    </header>
  );
}
```

만약 자주 사용되지 않는 Link 컴포넌트의 프리페칭 기능을 해제하고 싶은 경우 Link 컴포넌트의 prefetch 속성을 false로 설정해 주면 된다.

`<Link href={"/search"} prefetch={false}>search</Link>`
