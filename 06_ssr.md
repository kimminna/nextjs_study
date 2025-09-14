# SSR

Next.js 의 가장 큰 장점 중 하나는 내장된 서버 사이드 렌더링 기법이라고 볼 수 있다.

SSR(서버 사이드 렌더링)은 일반적으로 클라이언트 사이드에서 동작하는 싱글 페이지 앱(SPA)를 서버에서 렌더링한 다음, 완전히 렌더링된 페이지를 클라이언트로 보내는 방식을 말한다. 클라이언트의 JS 번들은 이후에 SPA의 높은 상호 작용을 가능하게 한다.

SSR은 검색 엔진 최적화(SEO), 초기 페이지 로드 속도를 높일 수 있는 등 다양한 장점을 가진다.

## getServerSideProps 함수

Next.js에서 SSR을 활성화하려면 `getServerSideProps`라는 함수를 사용하면 된다. 이 함수는 페이지가 렌더링되기 전에 서버에서 실행되며, 페이지 렌더링에 필요한 데이터를 가져오는 데 사용된다.

getServerSideProps 함수는 **props 키를 포함하는 객체를 반환**해야 한다. 이 props는 페이지 컴포넌트의 props가 된다.

```tsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://api.example.com/data`);
  const data = await res.json();

  if (!data) {
    return {
      notFound: true,
    };
  }

  return {
    props: { data }, // 무조건 props를 객체 프로퍼티로 가지는 객체를 반환해야 함
  };
}
```

### 서버에서 데이터를 가져와서 컴포넌트로 전달하기

주로 getServerSideProps 함수는 SSR에 필요한 데이터를 가져오는 데 쓰이면 좋다. API 호출, 디비 쿼리 등 기타 비동기 작업에 쓰인다.

```tsx
export async function getServerSideProps(context) {
  const { params } = context;
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: { post },
  };
}

function BlogPostPage({ post }) {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

export default BlogPostPage;
```

- getServerSideProps의 context 매개변수를 사용하여 동적인 라우트 파라미터(id)에 접근한 다음, 해당 블로그의 포스트를 가져와서 페이지 컴포넌트에 props로 전달하는 예제

## SSR에서의 에러 처리와 예외 상황 다루기

만약 getServerSideProps 가 **notFound 키가 true로 설정된 객체**를 반환하면, Next.js는 404 페이지를 렌더링한다. 이는 특정 블로그 포스트와 같이 존재하지 않을 수 있는 데이터를 가져오려고 할 때 사용하면 좋다.

```tsx
export async function getServerSideProps(context) {
  try {
    const { params } = context;
    const res = await fetch(`https://api.example.com/posts/${params.id}`);
    const post = await res.json();

    if (!post) {
      return {
        notFound: true,
      };
    }

    return {
      props: { post },
    };
  } catch (error) {
    return {
      props: {
        error: "블로그 포스트를 가져오는 중에 오류가 발생했습니다.",
      },
    };
  }
}

function BlogPostPage({ post, error }) {
  if (error) {
    return <div>{error}</div>;
  }

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </div>
  );
}

export default BlogPostPage;
```

- fetch 작업 중 네트워크 에러가 발생하면 에러를 catch 하여 에러 메시지를 반환하는 예제
