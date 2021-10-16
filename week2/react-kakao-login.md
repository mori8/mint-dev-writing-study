# [[react-kakao-login] 카카오 로그인 구현 ](https://velog.io/@kados22/react-kakao-login-%EC%B9%B4%EC%B9%B4%EC%98%A4-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84)

회사가 새로운 서비스_Numble_을 런칭하게 되면서 사이트를 처음부터 개발하게 되었습니다. 가장 먼저 시작한 것은 SNS 간편 로그인. Numble에서는 구글, 카카오, 네이버 간편 로그인을 제공하게 되었습니다. 

최대한 빠르게 마무리짓고 다른 기능으로 넘어가고자 웬만하면 라이브러리를 찾아서 이용하려고 했는데, 잘 나오지 않는 부분도 있고 해서 제가 한 번 하나씩 정리해보려고 합니다. 저는 NextJS, React, Typescript로 개발했습니다.

카카오 앱 키를 발급받는 과정은 생략하겠습니다. 앱 키는 JavaScript 키를 사용해주시고 카카오 로그인에서 활성화 설정 상태를 ON으로 설정해주신 다음에 시작해주세요!

### 1. 라이브러리 설치
저는 [react-kakao-login](https://github.com/wonism/react-kakao-login)을 사용했습니다.
`npm i react-kakao-login`

라이브러리 설치까지 완료되었다면 본격적으로 코드에 적용해봅시다.

### 2. script 태그 삽입
pages 폴더 안에 있는 _document.tsx 파일의 `<Head>` 안에 아래 태그를 작성해주세요.

```TypeScript
// pages/_document.tsx

import Document, { Html, Head, Main, NextScript } from "next/document";

class MyDocument extends Document {
	render(){
    	return(
          <Html>
          	<Head>
          		<script defer src="https://developers.kakao.com/sdk/js/kakao.js"
          ></script>
          	</Head>
          <body>...</body>
          </Html>
        );
    }
}
```

### 3. 컴포넌트 작성 
이제 필요한 페이지에다 라이브러리를 사용해봅시다!

```TypeScript
import React, { useState, useEffect } from "react";
import KakaoLogin from "react-kakao-login";

export default function KakaoLogin(){
	return(
    	<KakaoLogin
        token={String(process.env.NEXT_PUBLIC_KAKAO_APP_KEY)}
        onSuccess={() => {console.log("로그인성공", err);}} // 성공 시 실행할 함수
        onFail={(err) => {
          console.log("로그인실패", err);
        }}
        onLogout={() => {
          console.log("로그아웃");
        }}
        render={({ onClick }) => (
          <div
            onClick={(e) => {
              e.preventDefault();
              onClick();
            }}
          >
            카카오로 로그인하기
          </div>
        )}
      ></KakaoLogin>
    )
}
```

render property를 이용해서 로그인 버튼을 간단하게 커스텀할 수 있습니다.

### 그 와중에 발생했던 에러
`token={process.env.NEXT_PUBLIC_KAKAO_APP_KEY}`
카카오 앱 키를 env 파일에서 꺼내 사용하려는데 위의 코드처럼 작성을 하니 자꾸 에러가 났다!

> No overload matches this call.
  Overload 1 of 2, '(props: Props | Readonly<Props>): KakaoLogin', gave the following error.
    Type 'string | undefined' is not assignable to type 'string'.
      Type 'undefined' is not assignable to type 'string'.
  Overload 2 of 2, '(props: Props, context: any): KakaoLogin', gave the following error.
    Type 'string | undefined' is not assignable to type 'string'.
      Type 'undefined' is not assignable to type 'string'.ts(2769)
      
자꾸 내가 입력해준 값이 undefined일 수도 있다는데...

그래서 String으로 감싸줬더니 멀쩡하게 동작한다. 아무리 검색해봐도 세상 사람들 모두 String 없이 Next 환경 변수 쓰던데...
  혹시 왜  이런지 아시거나 더 나은 해결 방법 아시는 분이 계시다면 알려주세요...
