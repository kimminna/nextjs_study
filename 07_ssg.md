# SSG(Static Site Generation)

Next.js는 기본적으로 렌더링 할 경우 서버에서 사전 렌더링(pre-rendering) 하는 방식을 사용한다.

HTML을 미리 렌더링 한 후 사용자의 요청이 오면 Chunk 단위로 Javascript를 보내주어 이벤트가 작동하도록 한다. 이를 Hydration이라고 한다.

서버에서 사전 렌더링 하는 방법은 SSG와 SSR로 나뉘어진다.

페이지가 Static Site Generation을 사용하는 경우, 페이지 HTML 파일은 빌드 타임에 생성된다. 페이지가 한 번 빌드되고, CDN에 의해 제공되므로 매 요청마다 서버가 페이지를 렌더링하는 것보다 훨씬 빠르게 제공할 수 있다. 요청 시 새로운 데이터(최신 데이터 반영)을 가져올 필요가 없거나, 사전 렌더링 된 HTML을 캐시하려는 경우 사용하면 좋다.

## Preview Mode에서 확인하기

개발자 모드의 미리보기 모드를 사용해서 미리 생성되는 정적 페이지 파일에 어떤 요소가 포함되어 있는지 확인 가능!

## 정적 경로에 적용하기

페이지에서 `getStaticProps` 라는 함수를 export 하는 경우, Next.js에서는 `getStaticProps`로부터 반환된 props를 사용하여 해당 페이지를 빌드 타임에 미리 렌더링한다.

- 코드 예시
  ```tsx

  export const getStaticProps = async () => {
    console.log("index");
    const [allBooks, randomBooks] = await Promise.all([
      fetchBooks(),
      fetchRandomBooks(),
    ]); // 병렬 호출
    return {
      props: { allBooks, randomBooks },
    };
  };
  export default function Home({
    allBooks,
    randomBooks,
  }: InferGetStaticPropsType<typeof getStaticProps>) {
    return (
      <div className={style.container}>
        <section>
          <h3>지금 추천하는 도서</h3>
          {randomBooks.map((book) => (
            <BookItem key={book.id} {...book} />
          ))}
        </section>
        <section>
          <h3>등록된 모든 도서</h3>
          {allBooks.map((book) => (
            <BookItem key={book.id} {...book} />
          ))}
        </section>
      </div>
    );
  ```

### 어떤 경우에 사용하면 좋을까?

- 페이지 렌더링에 필요한 데이터가 유저의 요청 이전인, 빌드 타임에 사용 가능해야 할 때
- 페이지가 SEO를 위해 반드시 미리 렌더링되어야 하고 매우 빨라야만 할 때
  - getStaticProps는 HTML과 JSON 파일을 생성한다. 두 가지 모두 성능을 위해 CDN에 캐시를 저장하는 특징을 가짐.
- 데이터가 개별이 아니라, 공개적으로 캐시되어야 할 때

## 동적 경로에 적용하기

페이지에 **동적 경로**가 있고, `getStaticProps`를 사용하려는 경우 정적으로 생성할 경로 목록들을 미리 정의해야 한다.

동적 경로를 사용하는 페이지에서 `getStaticPaths`(정적 사이트 생성)라는 함수를 export하면 Next.js는 getStaticPaths에 지정된 모든 경로를 정적으로 미리 렌더링한다.

### 어떤 경우에 사용하면 좋을까?

- 데이터가 DB에서 제공되는 경우
- 데이터가 파일 시스템에서 제공되는 경우
- 데이터가 공개적으로 캐시될 수 있는 경우(특정 유저가 아님)
- 페이지가 사전 렌더링 되어야 하고(SEO를 위함) 매우 빨라야 하는 경우

### 폴백 옵션 설정하기

getStaticPaths의 fallback 옵션을 사용하면 빌드 중에 생성할 페이지들을 제어할 수 있다. 빌드 중에 더 많은 페이지를 생성하면 빌드 속도가 느려진다.

path에 대한 빈 배열을 반환하면 모든 페이지가 요청이 될 때만 생성된다. 이 경우 여러 환경에 Next.js 애플리케이션을 배포할 때 유용하다.

- 코드 예시

  ```tsx
  export async function getStaticPaths() {
    // 이 경우(미리보기 환경에서) 정적 페이지를
    // 프리 렌더링하지 마십시오.
    // (빌드는 빨라지지만 초기 페이지 로드는 느려짐)
    if (process.env.SKIP_BUILD_STATIC_GENERATION) {
      return {
        paths: [],
        fallback: "blocking",
      };
    }

    // 외부 API 엔드포인트를 호출하여 포스트 가져오기
    const res = await fetch("https://.../posts");
    const posts = await res.json();

    // posts를 기반으로 프리 렌더링할 경로 가져오기
    // 프로덕션 환경에서는 모든 페이지를 프리 렌더링합니다.
    // (빌드 속도는 느려지지만 초기 페이지 로드 속도는 빨라짐)
    const paths = posts.map((post) => ({
      params: { id: post.id },
    }));

    // { fallback: false } 는 다른 경로가 404임을 의미합니다.
    return { paths, fallback: false };
  }
  ```

- **fallback** 옵션 정리

  - **blocking** - ssr 방식
  - **true** - ssr + 데이터가 없는 폴백 상태의 페이지부터 반환. 후속으로 데이터 띄움
  - **false** - 404 not found 페이지 반환

- fallback : true일 경우, 다음과 같이 로딩 중일 때 보이는 로딩 화면을 router.isFallback 으로 제어 가능
  ```tsx

  export const getStaticPaths = () => {
    return {
      paths: [
        {
          params: { id: "1" },
        },
        { params: { id: "2" } },
        { params: { id: "3" } },
        { params: { id: "4" } },
      ],
      fallback: true,
  };
  export const getStaticProps = async (context: GetStaticPropsContext) => {
    const id = context.params!.id;
    const book = await fetchOneBook(Number(id));

    if (!book) {
      return {
        notFound: true,
      };
    }
    return {
      props: { book },
    };
  };
  export default function Page({
    book,
  }: InferGetStaticPropsType<typeof getStaticProps>) {
    const router = useRouter();

    if (router.isFallback) return "로딩 중입니다";

    if (!book) return "문제가 발생했습니다. 다시 시도해 주세요.";

    const { title, subTitle, description, author, publisher, coverImgUrl } = book;
    return (
      <div className={style.container}>
        <div
          className={style.cover_img_container}
          style={{ backgroundImage: `url('${coverImgUrl}')` }}
        >
          <img src={coverImgUrl} />
        </div>
        <div className={style.title}>{title}</div>
        <div className={style.subTitle}>{subTitle}</div>
        <div className={style.author}>
          {author} | {publisher}
        </div>

        <div className={style.description}>{description}</div>
      </div>
    );
  }

  ```
