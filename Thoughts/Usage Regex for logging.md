
### 원본 URL:

[https://newrelic.com/blog/log/extracting-log-data-with-regex](https://newrelic.com/blog/log/extracting-log-data-with-regex)

---

### **정규표현식으로 로그 데이터 추출하기**

**Regex Parsing: Using regular expressions to extract data from your logs**

**작성자**: James Buchanan (Principal Solutions Architect) **최종 업데이트**: 2023년 11월 15일 | 읽는 시간: 11분

---

### **서론**

개발자 도구 상자에서 **정규표현식(Regex)** 은 스위스 아ーミ 나이프와 같습니다. 거의 항상 더 좋은 정규표현식이 존재하죠.

좋은 정규표현식을 만드는 과정은 반복적입니다. 새로운 데이터, 특히 **엣지 케이스(edge cases)** 를 많이 넣을수록 품질과 신뢰성이 높아집니다.

단순히 “동작하는” 정규표현식은 충분할 수 있지만, **대규모 시스템**, **신뢰할 수 없는 데이터**, **여러 로그 형식**을 다룰 때는 **견고하고, resilient하며, 성능 좋은** 정규표현식을 만들어야 합니다.

이번 글에서는 로그에서 **이름-값 쌍(name-value pairs)** 을 추출하는 실전 사례를 통해 강력한 정규표현식을 만드는 방법을 알아보겠습니다.

---

### **실제 사용 사례**

New Relic 고객의 로그 파싱을 도와주면서 나온 요구사항입니다:

- 로그에 여러 개의 attr=value 형태의 이름-값 쌍이 존재
- 값(value)에는 공백이 포함될 수 있음
- 모든 쌍을 다 추출할 필요는 없음
- 일부 쌍은 항상 존재하지만, 일부는 없을 수도 있음
- 쌍의 순서는 바뀔 수 있음

**예시 로그**:

text

```
my favourite pizza=ham and pineapple drink=lime and lemonade venue=london name=james buchanan
```

여기서 우리는 pizza, drink, name 필드만 추출하고 싶고, venue는 제외하고 싶습니다.

---

### **TL;DR – 최종 정규표현식**

**일반 Regex 버전**:

regex

```
(?:^|\s+)(?=.*?attrname=(?<attrname>[^=]+?(?=(?:\s+\b\w+\b=|\s*?$))))?
```

**Grok 패턴 버전** (New Relic 추천):

regex

```
(?:^|\s+)(?=%{DATA}attrname=(?<attrname>[^=]+?(?=(?:\s+%{WORD}=|%{SPACE}?$))))?
```

이 규칙의 장점:

- 키-값 쌍이 **모두 존재하지 않아도** 동작
- **순서가 바뀌어도** 동작
- 값 안에 **공백**이 있어도 제대로 추출

---

### **단계별 설명**

#### 1. 처음에 만든 취약한 규칙

regex

```
pizza=(?<pizza>%{DATA})drink=(?<drink>%{DATA})name=(?<name>%{GREEDYDATA})
```

→ **문제**: 순서가 고정되어 있고, 값이 하나라도 빠지면 전체 파싱이 실패합니다.

#### 2. Lookahead(전방탐색)를 사용한 개선

**pizza 값만 안전하게 추출**:

regex

```
(?=%{DATA}pizza=(?<pizza>[^=]+?(?=(?:\s+%{WORD}=|%{SPACE}?$))))
```

**여러 필드 동시 추출** (최종 버전):

regex

```
(?:^|\s+)(?=%{DATA}pizza=(?<pizza>[^=]+?(?=(?:\s+%{WORD}=|%{SPACE}?$))))?(?=%{DATA}drink=(?<drink>[^=]+?(?=(?:\s+%{WORD}=|%{SPACE}?$))))?(?=%{DATA}name=(?<name>[^=]+?(?=(?:\s+%{WORD}=|%{SPACE}?$))))?
```

---

### **성능 최적화**

Lookahead는 성능 비용이 발생할 수 있습니다. 성능을 높이려면 규칙 앞에 (?:^|\s+) 를 추가하세요.

---

### **로그 파싱용 Regex 베스트 프랙티스**

- **단순하게 시작**하라
- **너무 greedy한 패턴(.*)** 은 피하라 → .*? (non-greedy) 사용
- **Non-capturing group** (?:...) 적극 활용
- **Named groups** (?<name>...) 사용으로 가독성 높이기
- **성능 테스트** 필수 (특히 대용량 로그)
- **엣지 케이스**를 항상 고려