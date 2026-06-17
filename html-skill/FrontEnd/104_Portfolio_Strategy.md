# 📁 FrontEnd 104: Portfolio Strategy & Showcase

앞선 101~103 단계를 통해 축적된 기술과 결과물을 집대성하여, 외부(고용주, 협업자)에게 나를 증명할 수 있는 **포트폴리오 페이지**를 구성하는 전략입니다.

---

## 1. 포트폴리오 핵심 구성 요소
단순한 링크 모음이 아니라, 나의 **'문제 해결 능력'**을 보여주는 스토리텔링이 필요합니다.

### 🍱 포함되어야 할 메뉴
- **About Me**: 엔지니어로서의 철학과 주 기술 스택(React, TS, D3, AI).
- **Projects**: 각 프로젝트별 **배경 - 기술적 난제 - 해결 방법 - 결과(Metrics)** 기술.
- **Interactive Dataviz**: D3.js로 만든 실시간 데이터 차트 (가장 시각적인 강력함).
- **Live Demo**: `penfromthenorthwest.com`과 같은 실제 작동하는 앱 링크.

---

## 2. 기술적 구현 전략 (The New Window)
사용자가 포트폴리오를 보다가 메인 사이트에서 이탈하지 않도록, 세부 프로젝트나 데모는 **새 창(`_blank`)**으로 열리게 설계합니다.

```tsx
// Portfolio Card Component
const ProjectCard: React.FC<ProjectProps> = ({ title, url }) => {
  return (
    <div className="card">
      <h3>{title}</h3>
      <a
        href={url}
        target="_blank"
        rel="noopener noreferrer" // 보안과 성능을 위해 필수
        className="btn"
      >
        View Live Project
      </a>
    </div>
  );
};
```

---

## 3. 포트폴리오 고도화 팁
- **Performance**: Lighthouse 점수 관리 (SEO, 접근성, 속도).
- **Responsive**: 모바일에서도 완벽하게 작동하는 D3 차트 (ViewBox 최적화).
- **Dark/Light Mode**: 테마 토글 기능을 넣어 프론트엔드 역량 과시.
- **Micro-interactions**: 버튼 호버 효과, 카드 등장 애니메이션(Framer Motion 등).

---

## 🚀 Final Checklist
1. [ ] 101에서 배운 **TypeScript**로 모든 코드의 타입 안정성이 확보되었는가?
2. [ ] 102에서 배운 **D3.js**를 활용해 복잡한 데이터를 직관적으로 시각화했는가?
3. [ ] 103에서 구현한 **Firebase/Gemini API**가 실시간으로 원활하게 작동하는가?
4. [ ] **Cloudflare**를 통해 안정적으로 서빙되고 있는가?

---
_축하합니다! 이제 당신의 지식과 기술이 담긴 완벽한 프론트엔드 포트폴리오를 세상에 내놓을 준비가 되었습니다. 🥘✨_
