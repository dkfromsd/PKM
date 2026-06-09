# 🎨 FrontEnd 101: Basic Grammar & Core Structure

이 가이드는 현대적인 프론트엔드 개발의 핵심인 **React, TypeScript, SVG**의 기본 문법과 구조를 학습하기 위해 작성되었습니다.

---

## 1. TypeScript (The Strong Foundation)
자바스크립트에 '타입'을 더해 에러를 방지하고 유지보수성을 높입니다.

### 📋 핵심 문법
- **Basic Types**: `string`, `number`, `boolean`, `any`, `unknown`
- **Interfaces & Types**: 객체의 형태를 정의합니다.
- **Generics**: 재사용 가능한 컴포넌트나 함수를 위한 타입 변수.

```typescript
// Interface 예시
interface User {
  id: number;
  name: string;
  email?: string; // Optional field
}

// Function with types
const greet = (user: User): string => {
  return `Hello, ${user.name}!`;
};
```

### 🔗 추천 레퍼런스
- [TypeScript Official Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Deep Dive (한글)](https://radlohead.gitbook.io/typescript-deep-dive/)

---

## 2. React (The UI Engine)
선언적이고 컴포넌트 기반의 UI 라이브러리입니다.

### 📋 핵심 구조
- **JSX/TSX**: HTML-like 문법을 자바스크립트 내에서 사용.
- **Hooks**: `useState` (상태 관리), `useEffect` (사이드 이펙트), `useRef` (DOM 접근).
- **Props**: 부모에서 자식으로 데이터를 전달.

```tsx
import React, { useState } from 'react';

const Counter: React.FC = () => {
  const [count, setCount] = useState<number>(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
};
```

### 🔗 추천 레퍼런스
- [React Official Documentation (New)](https://react.dev/)
- [Epic React by Kent C. Dodds](https://epicreact.dev/)

---

## 3. SVG (Scalable Vector Graphics)
수학적 공식으로 그려지는 그래픽으로, 데이터 시각화의 기본 요소입니다.

### 📋 핵심 태그
- `<svg>`: 캔버스 정의 (viewBox 중요!)
- `<rect>`, `<circle>`, `<line>`: 기본 도형
- `<path>`: 복잡한 곡선과 모양 (`d` 속성 학습 필수)

```html
<svg width="100" height="100" viewBox="0 0 100 100">
  <circle cx="50" cy="50" r="40" stroke="green" stroke-width="4" fill="yellow" />
</svg>
```

### 🔗 추천 레퍼런스
- [MDN SVG Tutorial](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial)
- [Pocket Guide to Writing SVG](http://www.sarasoueidan.com/blog/svg-coordinate-systems/)

---

## ⚠️ 주의 사항 및 팁
1. **Any 사용 지양**: TypeScript를 쓸 때 `any`를 남발하면 쓰는 의미가 없습니다. 최대한 구체적인 타입을 정의하세요.
2. **Key Prop**: 리스트를 렌더링할 때는 반드시 유니크한 `key`를 부여해야 React가 효율적으로 업데이트합니다.
3. **ViewBox vs Width/Height**: SVG에서 `viewBox`는 내부 좌표계를 정의하고, `width/height`는 실제 화면 크기를 결정합니다. 반응형 디자인의 핵심입니다.

---
_Next Step: [[102_Applied_Examples|102 응용 예제 (D3.js integration)]]_
