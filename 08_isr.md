# ISR(Incremental Static Regeneration)

ISR은 **정적 페이지를 필요할 때 부분적으로 다시 생성하는 방식**을 말한다.

1. SSR처럼 매 요청마다 렌더링되지 않는다.
2. SSG처럼 빌드 타임에 모든 페이지를 미리 만들지도 않는다.

다음과 같은 방식으로 **SSG의 성능 + SSR의 유연성**을 모두 얻는다.

1. 첫 요청 시에 캐싱된 정적 HTML이 있으면 즉시 반환한다.
2. revalidate 시간이 지나면, Stale 데이터를 반환하면서 백그라운드에서 새 HTML을 생성한다.
3. 이후 요청부터는 갱신된 HTML 파일을 반환한다.

## 사용법

ISR을 사용하기 위해서는 `revalidate` 속성을 `getStaticProps`에 추가한다.

```tsx
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// 이 함수는 빌드 시 서버 측에서 호출됩니다.
// 재검증이 활성화되고 새로운 요청이 들어오면,
// 서버리스 함수에서 다시 호출될 수 있습니다.
export async function getStaticProps() {
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  return {
    props: {
      posts,
    },
    // Next.js는 페이지 재생성을 시도합니다:
    // - 요청이 들어올 때
    // - 최대 10초마다 한 번씩
    revalidate: 10, // 초 단위
  };
}

// 이 함수는 빌드 시 서버 측에서 호출됩니다.
// 경로가 생성되지 않은 경우,
// 서버리스 함수에서 다시 호출될 수 있습니다.
export async function getStaticPaths() {
  const res = await fetch("https://.../posts");
  const posts = await res.json();

  // 게시물을 기반으로 사전 렌더링할 경로를 가져옵니다.
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }));

  // 빌드 시 이러한 경로만 사전 렌더링합니다
  // 경로가 존재하지 않는 경우, { fallback: 'blocking' }은
  // 온디맨드로 페이지를 서버 렌더링합니다.
  return { paths, fallback: "blocking" };
}

export default Blog;
```

ISR이 동작하기 위해서는 해당 라우트가 정적으로 렌더링이 가능해야 한다. 만약 동적 경로를 가지는 페이지가 있다면, `generateStaticParams`를 통해 생성 가능한 경로들을 미리 확장하고, 라우트에 revalidate를 선언하여 ISR을 활성화한다.

## 주의사항

동적 API 가 사용되는 경우에는 ISR을 사용할 수 없다.

cookies, headers, searchParams와 같은 값들은 요청마다 값이 달라지기 때문에, Full Route Cache를 사용할 수 없고 SSR로 강제된다.

## 캐싱 재검증 : 시간 기반? 온 디맨드?

https://tech.kakao.com/posts/743
