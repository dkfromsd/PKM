# 📈 FrontEnd 102: Applied Examples (React + D3.js + SVG)

이 단계에서는 React의 선언적 렌더링과 D3.js의 강력한 수학적 연산 능력을 결합하여 데이터를 시각화하는 방법을 학습합니다.

---

## 1. D3.js (Data-Driven Documents)
데이터를 기반으로 DOM(특히 SVG)을 조작하는 라이브러리입니다. React와 함께 쓸 때는 D3를 **'수학 계산 엔진'**으로 사용하고 렌더링은 **'React'**에 맡기는 방식이 가장 추천됩니다.

### 📋 핵심 개념
- **Scales**: 데이터를 화면 좌표로 변환 (`scaleLinear`, `scaleBand`).
- **Shapes**: 선, 곡선, 원형 차트를 위한 경로 생성기 (`line`, `arc`).
- **Axes**: 좌표축 생성.

---

## 2. 실전 코드 예시 (Bar Chart)
TypeScript와 React를 활용한 간단한 막대 그래프 컴포넌트 구조입니다.

```tsx
import React, { useRef, useEffect } from 'react';
import * as d3 from 'd3';

interface DataPoint {
  label: string;
  value: number;
}

const BarChart: React.FC<{ data: DataPoint[] }> = ({ data }) => {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    if (!svgRef.current) return;

    const svg = d3.select(svgRef.current);
    const width = 500;
    const height = 300;

    // 1. Scales 정의
    const xScale = d3.scaleBand()
      .domain(data.map(d => d.label))
      .range([0, width])
      .padding(0.2);

    const yScale = d3.scaleLinear()
      .domain([0, d3.max(data, d => d.value) || 0])
      .range([height, 0]);

    // 2. 렌더링 로직 (React 방식 또는 D3 Selection 방식)
    // 여기서는 D3 Selection으로 간단히 구현
    svg.selectAll('.bar')
      .data(data)
      .join('rect')
      .attr('class', 'bar')
      .attr('x', d => xScale(d.label) || 0)
      .attr('y', d => yScale(d.value))
      .attr('width', xScale.bandwidth())
      .attr('height', d => height - yScale(d.value))
      .attr('fill', '#8b4513');
  }, [data]);

  return <svg ref={svgRef} width={500} height={300}></svg>;
};
```

---

## 3. 학습 레퍼런스 & 강의
- **[D3 by Observable](https://d3js.org/getting-started)**: D3의 공식 시작 가이드.
- **[Fullstack D3 and Data Visualization](https://www.newline.co/fullstack-d3)**: 유료지만 가장 구조적인 학습이 가능한 책/강의.
- **[React + D3 by Amelia Wattenberger](https://wattenberger.com/blog/react-and-d3)**: React와 D3를 조화롭게 섞는 최고의 튜토리얼.
- **[LogRocket: D3 with TypeScript](https://blog.logrocket.com/creating-visualizations-d3-typescript/)**: 타입 안정성을 고려한 시각화 가이드.

---

## ⚠️ 주의 사항
1. **렌더링 주도권**: React와 D3 모두 DOM을 조작하려 합니다. 충돌을 피하기 위해 `useRef`를 사용하여 특정 영역만 D3에 맡기거나, D3는 오직 '값 계산'에만 사용하고 `<rect>` 태그 등은 React의 `map` 함수로 그리는 것이 안전합니다.
2. **반응형 대응**: SVG의 `viewBox`와 `preserveAspectRatio` 속성을 공부하여 다양한 화면 크기에서도 그래프가 깨지지 않게 처리하세요.
3. **성능**: 데이터가 만 개 단위 이상일 경우 Canvas API 사용을 고려해야 합니다. (D3도 Canvas 지원)

---
_Next Step: [[103_Real_World_Implementation|103 실제 적용 (Domain & Deployment)]]_
