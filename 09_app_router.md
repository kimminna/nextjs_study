# App Router

Next.js 버전 13 이후부터 공유 레이아웃, 중첩 라우팅, 로딩 상태, 오류 처리 등을 지원하는 React Server Components 기반의 새로운 App Router가 도입되었다.

App Router는 app 이라는 새로운 디렉토리에서 작동한다. pages 디렉토리도 사용이 가능하므로, page → app 으로 마이그레이션하면서 다른 경로는 이전 동작을 위해 pages 디렉토리에 유지할 수 있다.

> App Router가 Pages Router보다 우선 순위가 높으므로, 디렉토리 간 경로가 동일 한 URL 경로로 해석되면 빌드 오류가 발생하여 충돌을 방지한다.

기본적으로 app 내부 컴포넌트는 React Server Components다.

### 왜 등장했을까?

- **레이아웃 작성 경험 개선**
  중첩 레이아웃을 만들고, 라우트 간 공유할 수 있으며, 네비게이션 탐색 중에도 상태가 보존되는 레이아웃을 쉽게 만들 수 있어야 한다.
- **고급 라우팅 솔루션의 필요성**
  많은 Next.js 애플리케이션이 대시보드 혹은 콘솔 형태로 사용되는데, 이러한 앱들은 고급 라우팅 솔루션이 필요하다.

## 파일 시스템 기반 라우터

App Router는 여러 폴더들을 기반으로 라우터가 구성되는 파일 시스템 기반 라우터다. 각 폴더가 URL 세그먼트에 매핑된다.

- **폴더**
  라우트를 정의하는 데 사용된다. root 폴더부터 시작해서, 페이지 파일이 포함된 leaf 폴더까지 계층 구조를 따라 경로가 정의된다.
- **파일**
  라우트 세그먼트(url path에 있는 부분들)에 표시되는 UI를 생성한다.

## 중첩 라우팅 지원

폴더를 서로 중첩시켜서 Nested Routes를 구성할 수 있다.

![image.png](attachment:07abbaae-9104-4981-9000-44a871482ec5:image.png)

경로로 설정할 각 Segment - page.tsx 파일들에 UI를 추가한다.

## 파일명 컨벤션

특정 동작에 따라 UI를 생성할 수 있는 특수 파일 세트가 존재한다.

각 라우트에 해당하는 각 폴더 안의 파일명을 파일명 컨벤션에 맞게 설정하면, **라우트 별로 고유한 페이지 / 여러 페이지 간 공유되는 UI / 404 페이지** 등을 정의할 수 있다.

- `page.tsx` - 각 라우트 고유 UI
- `layout.tsx` - 라우트 간 공유되는 UI
- `loading.tsx` - 라우트가 로드 중일 때 UI
- `not-found.tsx` - 찾을 수 없는 라우트
- `error.tsx` - 라우트에 대한 에러 바운더리 정의
- `global-error.tsx` - 루트 오류를 구체적으로 처리
- `route.tsx` - Web Request, Response API를 사용해 각 라우트에 대한 custom request handler를 정의하기 위한 파일
- `template.tsx` - layout과 page를 래핑하는 layout.tsx와 비슷하지만, 각 하위 항목에 대해 새 인스턴스를 만듦. (네비게이션 시 기존 상태를 보존하지 않음)
- `default.tsx` - 병렬 라우트에서 사용되는 Fallback UI

# 주요 기능

## Layouts - 여러 라우트 간 공유되는 UI

레이아웃은 UI를 공유하고, 네비게이팅이 일어나는 동안 여러 페이지에 걸쳐 상태를 보존하며, 리렌더링을 방지하고, 그 외 복잡한 인터페이스도 쉽게 배치하도록 도와준다. 또한, 중첩 라우팅 구조 내에서의 레이아웃은 중첩이 가능하다.

- 코드 예시
  ```tsx
  // app/dashboard/layout.tsx
  export default function DashboardLayout({
    children, // page 혹은 nested layout
  }) {
    return (
      <section>
        <nav>{/* 라우트들이 공유 할 UI를 작성.. */}</nav>
        {children}
      </section>
    );
  }
  ```

## Server Components

코드 일부가 서버나, 클라이언트에서 렌더링되도록 하는 두 가지 컴포넌트가 존재한다. (Server Component, Client Component)

서버 컴포넌트는 **클라이언트로 전송되는 js 번들 파일의 양을 줄여서, 초기 페이지 로드 속도를 높이는 데 특화**된 기능이다.

## Streaming

서버에서 UI를 점진적으로 렌더링할 수 있는 기능을 지원한다. 페이지의 HTML 파일을 더 작은 청크로 분할한 뒤, 준비가 완료되면 클라이언트로 해당 청크를 보낸다. 그 결과 사용자는 모든 데이터가 로드되기까지 기다리지 않고, 페이지의 일부를 더 빠르게 보고 상호작용할 수 있게 된다.

초기 페이지 로딩 성능뿐만 아니라, 전체 라우트의 렌더링을 차단하는 느린 데이터 페치 속도를 가지고 있는 UI 개선에 도움을 준다.

**React의 Suspense와 함께 특수 파일인 loading.tsx를 활용하여 의미 있는 로딩 UI를 구축할 수 있다.**

> 라우트 세그먼트의 컨텐츠가 로드되는 동안, 서버에서 스켈레톤/스피너 등을 이용해 즉시 로드 중인 상태임을 알리는 대체 UI를 표시하고, 렌더링이 완료되면 로드 중인 컨텐츠로 자동 교체하는 방식.

- 코드 예시
  ```tsx
  // app/dashboard/page.tsx

  export default function Posts() {
    return (
      <section>
        <Suspense fallback={<p>Loading feed...</p>}>
          <PostFeed />
        </Suspense>
        <Suspense fallback={<p>Loading weather...</p>}>
          <Weather />
        </Suspense>
      </section>
    );
  }
  ```
- 활용
  우선 순위가 높거나(ex. 제품 정보) 데이터에 의존하지 않는(ex. 뼈대, 레이아웃) 컴포넌트를 먼저 내보낼 수 있다. 우선순위가 낮은 컴포넌트들은 데이터를 페치한 뒤에 보내진다.

## Data Fetching Supports

컴포넌트 단에서의 데이터 가져오기를 지원한다.

app 디렉토리의 레이아웃 / 페이지 / 컴포넌트 내에서 확장된 Fetch API를 통해 데이터 페치를 수행할 수 있다.

페이지 단에서의 데이터 페치만 가능했던 Page Router과는 다르게, App Router에서는 컴포넌트 단에서도 데이터를 페치할 수 있도록 변경되었다.

Page Router에서 사용되던 getServerSideProps 등과 같은 데이터 페칭 메소드는 App router에서 새로운 API로 대체된다.

1. (On the server) [fetch API 이용](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#fetching-data-on-the-server-with-fetch)
2. (On the server) [서드파티 라이브러리 이용](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#fetching-data-on-the-server-with-third-party-libraries)
3. (On the client) [Route handler 이용](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#fetching-data-on-the-client-with-route-handlers)
4. (On the client) [서드파티 라이브러리 이용](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating#fetching-data-on-the-client-with-third-party-libraries)
