# 🌐 FrontEnd 103: Real World Implementation (Domain & Deployment)

이 단계에서는 실제 도메인([penfromthenorthwest.com](https://penfromthenorthwest.com))을 기반으로 풀스택 기능을 연동하고 클라우드에 배포하는 실무 프로세스를 다룹니다.

---

## 1. 프로젝트 아키텍처 분석

### 🏗️ 주요 구성 요소
- **Domain**: `penfromthenorthwest.com` (향후 독립 도메인 브랜딩)
- **Deployment**: **Cloudflare Pages** (프론트엔드 호스팅 및 전역 엣지 네트워크)
- **Backend (Auth & Storage)**: **Firebase**
  - **Firebase Auth**: 블로그 포스팅 작성을 위한 관리자 로그인 기능.
  - **Firestore**: 블로그 글 데이터 및 사용자 메타데이터 저장.
- **AI Integration**: **Google Gemini API**
  - 기능: 사용자가 업로드한 이미지(Prompt)를 분석하여 비디오 클립 생성.
  - 연동 방식: Private GitHub에 API Key를 환경 변수로 관리하여 배포.

---

## 2. 핵심 기능 구현 가이드

### 🔐 Firebase 관리자 로그인 (Blog Page)
일반 사용자는 읽기만 가능하고, 로그인한 'Mickey'만 작성 버튼이 보이도록 구현합니다.

```typescript
import { getAuth, onAuthStateChanged } from "firebase/auth";

const auth = getAuth();
onAuthStateChanged(auth, (user) => {
  if (user && user.uid === ADMIN_UID) {
    // 블로그 작성 UI 활성화
    showEditor(true);
  } else {
    // 읽기 전용 모드
    showEditor(false);
  }
});
```

### 🤖 Gemini API 연동 (Image-to-Video)
이미지를 프롬프트와 함께 전달하여 시각적 결과물을 생성하는 흐름입니다.

```typescript
// cloudflare worker 또는 서버리스 함수 예시
export async function onRequest(context) {
  const { IMAGE_DATA, PROMPT } = await context.request.json();
  const GEMINI_API_KEY = context.env.GEMINI_API_KEY;

  // Gemini API 호출 로직...
  // 생성된 비디오 링크 또는 결과 반환
}
```

---

## 3. 학습 레퍼런스 & 배포 팁
- **[Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)**: 깃허브 레포지토리와 연동하여 자동 배포(`git push` 시 즉시 반영).
- **[Firebase Web Setup Guide](https://firebase.google.com/docs/web/setup)**: SDK 초기화 및 인증 환경 설정.
- **[Google AI SDK for Web](https://ai.google.dev/gemini-api/docs/quickstart?hl=ko)**: 프론트엔드에서 직접 혹은 프록시를 통해 Gemini를 호출하는 법.

---

## ⚠️ 실무 주의 사항
1. **API Key 보안**: 절대 API 키를 프론트엔드 코드(`index.html` 등)에 하드코딩하지 마세요. Cloudflare의 **Environment Variables**나 **Secrets** 기능을 사용해 감춰야 합니다.
2. **Firebase Rules**: `firestore.rules`에서 권한을 엄격히 설정하여 타인이 글을 수정하거나 삭제하지 못하게 차단해야 합니다.
3. **CORS 이슈**: Cloudflare와 Gemini/Firebase 간의 통신 시 도메인 허용(CORS) 설정을 확인하세요.

---
_Next Step: [[104_Portfolio_Strategy|104 포트폴리오 전략 (Final Showcase)]]_
