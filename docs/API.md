# API 문서

## 게임 시작

### POST /api/game/start
팀 A와 팀 B에서 뛴 선수를 맞히는 게임을 시작합니다.

#### Request Body
```typescript
{
  teamA: string,      // 팀 A 이름
  excludeLoan: boolean, // 임대 제외 여부
  leagues: string[]    // 허용할 리그 목록 (선택적)
}
```

#### Response
```typescript
{
  question: {
    teamA: string,    // 팀 A
    teamB: string,    // 팀 B
    questionId: string // 문제 식별자
  }
}
```

## 답안 제출

### POST /api/game/answer
사용자의 답안을 제출하고 정답 여부를 확인합니다.

#### Request Body
```typescript
{
  questionId: string, // 문제 식별자
  answer: string      // 사용자의 답
}
```

#### Response
```typescript
{
  isCorrect: boolean,  // 정답 여부
  correctAnswer: string, // 정답
  score: number,       // 점수
  nextQuestion: {      // 다음 문제
    teamA: string,
    teamB: string,
    questionId: string
  }
}
```

## 에러 코드

### HTTP 400 Bad Request
- 잘못된 요청 형식
- 팀이 존재하지 않음
- 잘못된 리그 지정

### HTTP 404 Not Found
- 팀 A와 팀 B를 뛴 선수가 없음

### HTTP 500 Internal Server Error
- 서버 내부 오류
