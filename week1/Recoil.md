1. [[POM]NextJS에 Recoil 도입하기](https://velog.io/@kados22/POMNextJS%EC%97%90-Recoil-%EB%8F%84%EC%9E%85%ED%95%98%EA%B8%B0)

PieceOfMood에서는 새로운 서비스 런칭을 준비하고 있습니다. 기존 서비스를 유지&보수하는 일도 즐거운 일이지만 아무래도 방대한 양의 코드를 모조리 개선해보려고 손을 대는 짓은 선뜻 하지 못할 일이죠. 어찌됐든 새로운 서비스를 처음부터 개발하게 되었으니, 기쁜 마음으로 [Recoil](https://recoiljs.org/docs/introduction/getting-started)을 도입하게 되었고 초반이지만 느낀점을 써봅니다.

# 이전 서비스에서는...
이전 서비스에서는 상태관리를 위해 ContextAPI를 사용했습니다. 따로 패키지를 설치할 필요없다는 점이 장점이죠. 잘 모르는 ContextAPI를 사용하려고 공식 문서를 들락거렸던 기억이 납니다. 그래도 기존에 짜여진 코드를 활용할 수 있었기 때문에 큰 어려움은 없었습니다.

그래도 ContextAPI를 사용하면서 아쉬웠던 부분은 있었습니다. 어쩌면 제가 ContextAPI에 대한 이해가 부족해서 그럴 수도 있겠지만요.

# 도입하고 보니...
도입하고 보니 이전보다 코드가 확연하게 깔끔해졌습니다!
우선 state를 import는 해오는 코드가 이전에 비해서 깔끔해지고, 매개변수로 넘겨주는 setState 함수를 대폭 줄일 수 있었습니다.

전
```Javascript
//pages/login/index.tsx
import React, { useState } from "react";
import { withAuthContext, useAuthContext } from "@src/context/Auth";

function Login() {
  const {
    state: { isAuthenticating, step, userData },
    action: { setIsAuthenticating, setStep },
    } = useAuthContext();
  
  return (
    <MainLayout>
      <div>This is Login Page</div>
      {isAuthenticating && <Spinner />}
      {step == AuthStep.READY && <Ready setStep={setStep} />}
      {step == AuthStep.AGREE && <Agreements setStep={setStep} />}
      {step == AuthStep.PROFILE_REQUIRED && (
        <RequiredProfile setStep={setStep} />
      )}
    </MainLayout>
  );
}

export default withAuthContext(Login);
```

후
```Typescript
//pages/login/index.tsx
import React, { useState } from "react";
import { useRecoilValue } from "recoil";

export default function Login() {
  import { stepState } from "@src/states/Auth";
  
  return (
    <MainLayout>
      <div>This is Login Page</div>
      {isAuthenticating && <Spinner />}
      {step == AuthStep.READY && <Ready />}
      {step == AuthStep.AGREE && <Agreements />}
      {step == AuthStep.PROFILE_REQUIRED && (
        <RequiredProfile />
      )}
    </MainLayout>
  );
}
```

```Typescript
//src/states/Auth.tsx
import React from "react";
import { atom } from "recoil";
import { AuthStep } from "@src/interfaces/login";

export const stepState = atom({
  key: "stepState",
  default: AuthStep.READY,
});
```

짧은 코드라 획기적으로는 느껴지지 않을 수 있지만 개인적으로는 굉장히 편리하다고 느끼면서 사용하고 있습니다. 이미 많이들 쓰고있는 기술이지만 다시 한 번 더 추천드리면서! 글을 마무리 지어보겠습니다.

2. [[NextJS] RecoilRoot 위치 설정](https://velog.io/@kados22/NextJS-Recoil-RecoilRoot-%EC%9C%84%EC%B9%98-%EC%84%A4%EC%A0%95)
[Recoil 공식문서](https://recoiljs.org/docs/introduction/getting-started)를 보면 RecoilRoot 컴포넌트를 root Component에 적절히 넣어두라고 나와있다.

> Components that use recoil state need RecoilRoot to appear somewhere in the parent tree. A good place to put this is in your root component:

공식 문서의 예시에서는 App()에 적혀있는데...NextJS에는 App()이 없다. 그러므로 NextJS의 경우에는 _app.tsx에 RecoilRoot를 적어주면 된다.

```Typescript
//pages/_app.tsx
import "../styles/globals.css";
import type { AppProps } from "next/app";
import { RecoilRoot } from "recoil";

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <RecoilRoot>
      <Component {...pageProps} />
    </RecoilRoot>
  );
}
export default MyApp;
```
