# Data Fetching

https://nextjs-ko.org/docs/app/building-your-application/data-fetching/fetching

### 데이터를 서버에서 가져오기 vs 클라이언트에서 가져오기

실시간 데이터(ex. 폴링)가 필요하지 않은 대부분의 경우, 서버 컴포넌트를 사용하여 서버에서 데이터를 가져오는 것이 좋다.

- **서버 컴포넌트에서 데이터를 가져왔을 때의 장점**

  - 네트워크 요청 수와 클라이언트-서버 워터폴을 줄일 수 있다.
  - 클라이언트에 노출되면 안 되는 민감한 정보(ex. 액세스 토큰 및 API 키)를 보호할 수 있다.
  - 애플리케이션 코드와 데이터베이스가 동일한 지역에 있는 경우 데이터 소스에 가까운 곳에서 데이터를 가져옴으로써 지연 시간을 줄일 수 있다.
  - 데이터 요청은 캐싱되고 재검증될 수 있다.

- **서버 컴포넌트에서 데이터를 가져왔을 때의 단점**
  - 전체 페이지가 서버에서 다시 렌더링된다.
  - 작은 UI 조각을 변형/재검증하거나 실시간 데이터를 지속적으로 가져와야 하는 경우(ex. 실시간 뷰), 클라이언트에서 데이터를 가져오는 것이 더 적합하다,

# Next.js에서 데이터를 가져오는 방법

## 서버의 fetch API

fecth 요청은 기본적으로 새로운 데이터를 검색한다. 이를 사용하면 전체 경로가 동적으로 렌더링되며, 데이터가 캐싱되지 않는 것이 기본값.

fetch 요청을 캐싱하기 위해서는 cache 옵션을 force-cache로 설정한다.

```tsx
import { Suspense } from 'react'


export default async function Cart() {
  const res = await fetch('https://api.example.com/...',
  {cache: 'force-cache'})

  return '...'
}

export default function Navigation() {
  return (
    <>
      <Suspense fallback={<LoadingIcon />}>
        <Cart />
      </Suspense>
    <>
  )
}
```

### Caching

캐싱은 서버에 요청하는 횟수를 줄이기 위해 데이터를 저장하는 과정이다. Next.js는 개별 데이터 요청을 위한 Data Cache를 내장하고 있다.

- caching 옵션
  - `{cache: ‘force-cache’}` - 기본값. 캐싱
  - `{cache: ‘no-store’}` - 캐싱 x

### Revalidating data

데이터 캐시를 비우고 최신 데이터를 가져오는 과정.

데이터가 변경될 때, 최신 정보를 표시하면서도 정적 렌더링의 속도 이점 유지 가능

- **시간 기반 revalidation**
  일정 시간이 지난 후 자동으로 데이터를 revalidate.
  데이터가 자주 변경되지 않고, 신선도가 중요하지 않은 경우에 사용한다.
  ```tsx
  fetch("https://...", { next: { revalidate: 3600 } }); // 최대 1시간마다 revalidate
  ```
- **on-demand revalidation**
  이벤트(ex. 폼)를 기반으로 수동으로 revalidate.
  ```tsx
  import { revalidatePath } from 'next/cache'

  export default async createPost() {
    try {
      // 데이터 수정
      revalidatePath('/posts')
    } catch(error) {}

  }
  ```

### Request Memoization

Next.js는 동일한 URL과 옵션을 가진 요청을 자동으로 메모이제이션한다. 이는 리액트 컴포넌트 트리의 여러 곳에서 동일한 데이터를 가져오기 위한 fetch 함수를 호출할 때 한 번만 실행되는 것을 의미한다.

경로 전체에서 동일한 데이터를 사용해야 하는 경우(ex. 레이아웃, 페이지 및 여러 컴포넌트), 트리 상단에서 데이터를 가져와 컴포넌트 간에 props를 전달할 필요 없이 각자 필요한 컴포넌트에서 데이터를 가져온다. 이때 네트워크를 통해 동일한 데이터에 대해 여러 요청을 하는 성능 문제를 걱정할 필요 없이 데이터를 페치할 수 있다.

```tsx
async function getItem() {
  // `fetch` 함수는 자동으로 메모이제이션되고 결과는
  // 캐싱됩니다.
  const res = await fetch("https://.../item/1");
  return res.json();
}

// 이 함수는 두 번 호출되지만 처음에는 한 번만 실행됩니다.
const item = await getItem(); // cache MISS

// 두 번째 호출은 경로 어디에서든 발생할 수 있습니다.
const item = await getItem(); // cache HIT
```

![image.png](attachment:9caeca25-51d5-4a04-bc6a-b29de880625e:image.png)

> 메모이제이션은 fetch 요청의 get 메서드에서만 적용.

> generateMetadata, generateStaticParams, Layouts, Pages 및 기타 서버 컴포넌트의 fetch 요청에 적용됨.

> 경로 렌더링이 완료되면 메모리가 리셋됨. 모든 요청 메모이제이션 항목에서 제거. (페이지당 렌더링.)
