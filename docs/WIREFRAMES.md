# 와이어프레임

## 게임 시작 화면

### 컴포넌트 구조
```typescript
App
├── GameStartScreen
│   ├── TeamInput
│   ├── LoanCheckbox
│   ├── LeagueFilter
│   └── StartButton
```

### 레이아웃
```typescript
// 팀 입력 컴포넌트
TeamInput
├── InputField
├── ValidationMessage
└── AutoCompleteList

// 리그 필터 컴포넌트
LeagueFilter
├── FilterList
├── CheckboxGroup
└── ClearButton
```

## 문제 화면

### 컴포넌트 구조
```typescript
QuestionScreen
├── TeamDisplay
│   ├── TeamA
│   └── TeamB
├── AnswerInput
│   ├── InputField
│   └── Timer
└── SubmitButton
```

### 레이아웃
```typescript
// 팀 표시 컴포넌트
TeamDisplay
├── TeamLogo
├── TeamName
└── TeamInfo

// 답 입력 컴포넌트
AnswerInput
├── InputField
├── TimerDisplay
└── ErrorMessage
```

## 결과 화면

### 컴포넌트 구조
```typescript
ResultScreen
├── ResultIndicator
│   ├── Correct/WrongIcon
│   └── CorrectAnswer
├── ScoreDisplay
└── NextButton
```

### 레이아웃
```typescript
// 결과 표시 컴포넌트
ResultIndicator
├── Icon
├── Message
└── CorrectAnswer

// 점수 표시 컴포넌트
ScoreDisplay
├── CurrentScore
├── TotalScore
└── Progress
```
