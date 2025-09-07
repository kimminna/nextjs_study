# API Routes

## API Routes란?

Next.js에서 **서버 사이드에서 실행되는 API 엔드포인트를 정의할 수 있는 기능**이다. pages/api 폴더에 있는 파일들은 API 엔드포인트로 처리되며, 클라이언트는 이 엔드포인트에 HTTP 요청을 보내어 데이터를 받아온다.

- EndPoint : pages/api에 위치한 파일은 /api/\* 경로로 매핑된다.
- Response: 해당 API는 HTML 페이지 대신 JSON 데이터나 상태 코드를 반환한다.

cf.) 엔드포인트는 클라이언트와 서버 간의 통신 지점을 의미한다. 웹 개발에서 엔드포인트는 클라이언트가 서버에 요청을 보내는 특정 URL 경로를 말하며, 서버는 이 엔드포인트에 맞는 동작을 수행한 후 응답을 반환한다.

```tsx
import type { NextApiRequest, NextApiResponse } from "next";

type ResponseData = {
  message: string;
};

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResponseData>
) {
  res.status(200).json({ message: "Hello from Next.js!" });
}
```

### API Routes의 특징

1. **서버에서만 실행된다.**

   pages/api 폴더의 코드는 클라이언트에서 실행되지 않으며, Next.js 서버에서만 실행된다. 즉, 브라우저로 보내는 JS 번들링 파일에 포함되어 있지 않아서 클라이언트에서는 접근할 수 없다.

2. **보안 처리**

   민감한 정보나 비공개 API 키 등을 클라이언트에게 노출하지 않고 서버 측에서만 처리할 수 있다.

3. **프론트 + 백엔드 통합**

   백엔드 서버 없이 프론트엔드와 함께 API를 개발하고 운영할 수 있다.
