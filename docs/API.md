# API 문서

## Header

```typescript
{
  "Content-Type": "application/json",
}
```

</br>

## Response

```typescript
{
  status: number,
  message: string,
  data: any
}
```

</br>

## 인증 _변경예정_

### 구글 로그인

#### POST /api/auth/google

구글 OAuth를 통해 로그인/회원가입을 처리합니다.

##### Request Body

```typescript
{
  credential: string; // 구글 OAuth 인증 토큰
}
```

##### Response Headers

```typescript
{
  "Set-Cookie": "access_token=your_token_value; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600"
}
```

- `HttpOnly`: JavaScript에서 쿠키에 접근할 수 없도록 설정
- `Secure`: HTTPS를 통한 요청에서만 쿠키가 전송됨
- `SameSite=Strict`: 다른 사이트에서의 요청으로 쿠키가 전송되지 않음
- `Path=/`: 쿠키가 모든 경로에서 사용됨
- `Max-Age=3600`: 쿠키의 유효기간 (초 단위, 이 예시는 1시간)

##### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    user: {
      id: string,
      name: string,
      email: string,
      picture: string
    }
  }
}
```

</br>

### 로그아웃

#### POST /api/auth/logout

세션을 종료하고 쿠키를 삭제합니다.

##### Response Headers

```typescript
{
  "Set-Cookie": "access_token=; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=0"
}
```

##### Response Body

```typescript
{
  status: 200,
  message: "Logout successful",
  data: null
}
```

</br>

## 게임

### 리그 목록 조회

#### GET /api/v1/leagues

사용 가능한 리그 목록을 조회합니다.

#### Request Query

```typescript
{
  limit?: number,    // 조회할 리그 수 (기본값: 10)
  offset?: number,   // 페이지네이션 오프셋 (기본값: 0)
  sortBy?: string,   // 정렬 기준 ('name', 'country', 'teams')
  sortOrder?: string  // 정렬 순서 ('asc', 'desc')
}
```

#### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    leagues: {
      id: string,        // 리그 ID
      name: string,      // 리그명
      country: string,   // 국가
      logo: string,      // 리그 로고 URL
      teams: number,     // 소속 팀 수
      createdAt: string, // 생성 일시 (ISO 8601)
      updatedAt: string, // 업데이트 일시 (ISO 8601)
      version: string    // 리그 버전
    }[],
    total: number,
    currentPage: number,
    totalPages: number,
    hasNext: boolean,
    hasPrevious: boolean,
    limit: number,
    offset: number
  }
}
```

#### Error Response

```typescript
// 리그가 존재하지 않을 때
{
  status: 404,
  message: "No leagues found",
  errorType: "NO_LEAGUES_FOUND",
  details: {
    filters: {
      country: "KR"
    }
  }
}
```

#### Rate Limit
- 100 requests/minute
- 500 requests/minute per IP
- 1000 requests/hour per user

#### Authentication
- No authentication required

#### Example

```typescript
// Request
GET /api/v1/leagues?limit=10&offset=0&sortBy=name&sortOrder=asc

// Response
{
  status: 200,
  message: "Leagues fetched successfully",
  data: {
    leagues: [
      {
        id: "premier",
        name: "Premier League",
        country: "England",
        logo: "https://example.com/leagues/premier/logo.png",
        teams: 20,
        createdAt: "2025-06-18T00:00:00+09:00",
        updatedAt: "2025-06-18T00:00:00+09:00",
        version: "20250618000000"
      },
      // ... 더 많은 리그
    ],
    total: 5,
    currentPage: 1,
    totalPages: 1,
    hasNext: false,
    hasPrevious: false,
    limit: 10,
    offset: 0
  }
}
```

### 팀 목록 조회

#### GET /api/teams

- 리그를 보내면 해당 리그의 팀 목록을 조회합니다.
- 리그를 보내지 않으면 모든 팀 목록을 조회합니다.
- 사용자가 팀 A를 선택할 때 사용됩니다.
- 로고와 클럽이름이 출력됩니다.

##### Request Body

```typescript
{
  league?: string
}
```

- `league`: 리그

##### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    teams: {
      id: string,
      name: string,
      league: string,
      logo: string
    }[]
  }
}
```

- `teams`: 팀 목록
  - `id`: 팀 ID
  - `name`: 팀 이름
  - `league`: 리그
  - `logo`: 로고 URL

</br>

### 게임 시작

#### POST /api/game/start

팀 A와 팀 B에서 뛴 선수를 맞히는 게임을 시작합니다. 일반 모드와 하드 모드가 있습니다:

- 일반 모드: 팀 A를 선택하고, 해당 팀과 다른 팀에서 뛴 선수를 맞히는 모드
- 하드 모드: 팀 A 선택 없이, 랜덤하게 두 팀에서 뛴 선수를 맞히는 모드

##### Request Body

```typescript
{
  teamA?: string,        // 팀 A ID
  timeLimit?: number,    // 문제 제한 시간 (초)
  allowedLeagues?: string[], // 허용할 리그 목록
  sessionType: string,   // 세션 타입 ('time' 또는 'count')
  sessionLimit?: number  // 세션 제한 (time: 제한시간, count: 문제 수)
}
```

##### 필드 설명

```typescript
{
  status: number,
  message: string,
  data: {
    sessionId: string,    // 게임 세션 ID
    mode: string,         // 게임 모드
    sessionType: string,  // 세션 타입
    startTime: string,    // 게임 시작 시간 (ISO 8601)
    timeLimit: number,    // 시간 제한 (time 모드일 때)
    questionCount: number, // 문제 수 (count 모드일 때)
    nextQuestion: {
      teamA: string,      // 팀 A ID
      teamALogo: string,  // 팀 A 로고 URL
      teamB: string,      // 팀 B ID
      teamBLogo: string,  // 팀 B 로고 URL
      questionId: string, // 문제 식별자
      version: string     // 문제 버전
      createdAt: string   // 문제 생성 시간 (ISO 8601)
    },
    sessionVersion: string // 세션 버전
  }
}
```

#### Error Response

```typescript
// 잘못된 게임 모드
{
  status: 400,
  message: "Invalid game mode",
  errorType: "INVALID_MODE",
  details: {
    mode: "invalid-mode"
  }
}

// 팀이 존재하지 않을 때
{
  status: 404,
  message: "Team not found",
  errorType: "TEAM_NOT_FOUND",
  details: {
    teamId: "123"
  }
}

// 리그가 존재하지 않을 때
{
  status: 400,
  message: "League not found",
  errorType: "LEAGUE_NOT_FOUND",
  details: {
    leagueId: "invalid-league"
  }
}
```

#### Rate Limit
- 10 requests/minute per user
- 50 requests/minute per IP
- 100 requests/hour per user

#### Authentication
- Required
- Token: `woreboth_auth`

#### Example

```typescript
// Request
POST /api/v1/game/start
{
  mode: "normal",
  sessionType: "count",
  questionCount: 10,
  version: "1.0.0",
  timezone: "Asia/Seoul"
}

// Response
{
  status: 200,
  message: "Game started",
  data: {
    sessionId: "session123",
    mode: "normal",
    sessionType: "count",
    startTime: "2025-06-18T15:30:45+09:00",
    questionCount: 10,
    nextQuestion: {
      teamA: "123",
      teamALogo: "https://example.com/teams/123/logo.png",
      teamB: "456",
      teamBLogo: "https://example.com/teams/456/logo.png",
      questionId: "question123",
      version: "20250618000000",
      createdAt: "2025-06-18T15:30:45+09:00"
    },
    sessionVersion: "20250618000000"
  }
}
```

##### 응답 필드 설명

- `sessionId`: 게임 세션을 식별하는 ID
- `nextQuestion`: 다음 문제 정보
  - `teamA`: 팀 A ID
  - `teamALogo`: 팀 A 로고 URL
  - `teamB`: 팀 B ID
  - `teamBLogo`: 팀 B 로고 URL
  - `questionId`: 문제 식별자

##### 예시

```typescript
// 일반 모드 시작 예시
{
  teamA: "123",
  timeLimit: 10,
  allowedLeagues: ["Premier League", "La Liga"],
  sessionType: "count",
  sessionLimit: 10
}

// 하드 모드 시작 예시
{
  timeLimit: 10,
  allowedLeagues: null,
  sessionType: "time",
  sessionLimit: 60
}
```

</br>

### 답안 제출

#### POST /api/game/answer

사용자의 답안을 제출하고 정답 여부를 확인합니다. 하드 모드에서는 오답 시 게임이 자동으로 종료됩니다.

##### Request Body

```typescript
{
  questionId: string,   // 문제 식별자
  answer: string,       // 사용자의 답
  timeSpent?: number    // 소요 시간 (초)
}
```

##### 필드 설명

- `questionId`: 문제 식별자
- `answer`: 사용자가 입력한 선수 이름
- `timeSpent`: (옵션) 문제에 소요된 시간 (초)

##### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    isCorrect: boolean,      // 정답 여부
    score: number,           // 현재 점수
    remainingTime?: number,  // 남은 시간 (time 모드일 때)
    remainingQuestions?: number, // 남은 문제 수 (count 모드일 때)
    nextQuestion?: {
      teamA: string,         // 팀 A ID
      teamALogo: string,     // 팀 A 로고 URL
      teamB: string,         // 팀 B ID
      teamBLogo: string,     // 팀 B 로고 URL
      questionId: string     // 다음 문제 식별자
    }
  }
}
```

##### 응답 필드 설명

- `isCorrect`: 정답 여부
- `score`: 현재까지의 총 점수
- `remainingTime`: time 모드일 때 남은 시간 (초)
- `remainingQuestions`: count 모드일 때 남은 문제 수
- `nextQuestion`: 다음 문제 정보 (게임이 끝나지 않았을 때만 반환)
  - `teamA`: 팀 A ID
  - `teamALogo`: 팀 A 로고 URL
  - `teamB`: 팀 B ID
  - `teamBLogo`: 팀 B 로고 URL
  - `questionId`: 다음 문제 식별자

##### 예시

```typescript
// 요청 예시
{
  questionId: "123",
  answer: "Son Heung-min",
  timeSpent: 5
}

// 정답 응답 예시
{
  status: 200,
  message: "Correct answer",
  data: {
    isCorrect: true,
    score: 100,
    remainingTime: 95,  // time 모드일 때
    remainingQuestions: 9,  // count 모드일 때
    nextQuestion: {
      teamA: "456",
      teamALogo: "https://example.com/teams/456/logo.png",
      teamB: "789",
      teamBLogo: "https://example.com/teams/789/logo.png",
      questionId: "234"
    }
  }
}

// 오답 응답 예시 (하드 모드)
{
  status: 200,
  message: "Game over",
  data: {
    isCorrect: false,
    score: 100,
    remainingTime: 0,
    remainingQuestions: 0
  }
}
```

</br>

### 게임 세션 종료

#### POST /api/v1/game/session/end

게임 세션을 종료합니다.

#### Request Body

```typescript
{
  sessionId: string,  // 게임 세션 ID
  version: string     // 클라이언트 버전
  timezone: string    // 클라이언트 시간대
}
```

#### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    sessionId: string,      // 게임 세션 ID
    endTime: string,        // 세션 종료 시간 (ISO 8601)
    sessionVersion: string  // 세션 버전
  }
}
```

#### Error Response

```typescript
// 잘못된 세션 ID
{
  status: 400,
  message: "Invalid session ID",
  errorType: "INVALID_SESSION_ID",
  details: {
    sessionId: "invalid-session"
  }
}

// 세션이 존재하지 않을 때
{
  status: 404,
  message: "Session not found",
  errorType: "SESSION_NOT_FOUND",
  details: {
    sessionId: "123"
  }
}

// 세션이 이미 종료되었을 때
{
  status: 400,
  message: "Session already ended",
  errorType: "SESSION_ALREADY_ENDED",
  details: {
    sessionId: "123",
    endTime: "2025-06-18T15:30:45+09:00"
  }
}
```

#### Rate Limit
- 10 requests/minute per user
- 50 requests/minute per IP
- 100 requests/hour per user

#### Authentication
- Required
- Token: `woreboth_auth`

#### Example

```typescript
// Request
POST /api/v1/game/session/end
{
  sessionId: "session123",
  version: "1.0.0",
  timezone: "Asia/Seoul"
}

// Response
{
  status: 200,
  message: "Session ended",
  data: {
    sessionId: "session123",
    endTime: "2025-06-18T15:30:45+09:00",
    sessionVersion: "20250618000000"
  }
}
```

### 게임 결과 저장

#### POST /api/v1/game/result

게임 결과를 저장합니다.

#### Request Body

```typescript
{
  sessionId: string,     // 게임 세션 ID
  score: number,         // 최종 점수
  correctCount: number,  // 정답 수
  totalQuestions: number, // 총 문제 수
  playTime: number,      // 총 플레이 시간(초)
  version: string,       // 클라이언트 버전
  timezone: string       // 클라이언트 시간대
  answers: {
    questionId: string,  // 문제 ID
    answer: string,      // 선택한 팀 ID
    isCorrect: boolean,  // 정답 여부
    timeSpent: number    // 문제에 소요된 시간(초)
  }[]                    // 모든 문제의 답변 정보
}
```

#### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    gameId: string,          // 게임 ID
    finalScore: number,      // 최종 점수
    correctCount: number,    // 정답 수
    totalQuestions: number,  // 총 문제 수
    playTime: number,        // 총 플레이 시간(초)
    answers: {
      questionId: string,    // 문제 ID
      answer: string,        // 선택한 팀 ID
      isCorrect: boolean,    // 정답 여부
      timeSpent: number,     // 문제에 소요된 시간(초)
      createdAt: string      // 답변 생성 시간 (ISO 8601)
    }[],                     // 모든 문제의 답변 정보
    gameVersion: string      // 게임 버전
    createdAt: string        // 게임 생성 시간 (ISO 8601)
    updatedAt: string        // 게임 업데이트 시간 (ISO 8601)
  }
}
```

#### Error Response

```typescript
// 잘못된 세션 ID
{
  status: 400,
  message: "Invalid session ID",
  errorType: "INVALID_SESSION_ID",
  details: {
    sessionId: "invalid-session"
  }
}

// 세션이 존재하지 않을 때
{
  status: 404,
  message: "Session not found",
  errorType: "SESSION_NOT_FOUND",
  details: {
    sessionId: "123"
  }
}

// 세션이 이미 종료되었을 때
{
  status: 400,
  message: "Session already ended",
  errorType: "SESSION_ALREADY_ENDED",
  details: {
    sessionId: "123",
    endTime: "2025-06-18T15:30:45+09:00"
  }
}
```

#### Rate Limit
- 10 requests/minute per user
- 50 requests/minute per IP
- 100 requests/hour per user

#### Authentication
- Required
- Token: `woreboth_auth`

#### Example

```typescript
// Request
POST /api/v1/game/result
{
  sessionId: "session123",
  score: 1000,
  correctCount: 10,
  totalQuestions: 10,
  playTime: 150,
  version: "1.0.0",
  timezone: "Asia/Seoul",
  answers: [
    {
      questionId: "q1",
      answer: "123",
      isCorrect: true,
      timeSpent: 15
    },
    // ... 더 많은 답변
  ]
}

// Response
{
  status: 200,
  message: "Game result saved",
  data: {
    gameId: "game123",
    finalScore: 1000,
    correctCount: 10,
    totalQuestions: 10,
    playTime: 150,
    answers: [
      {
        questionId: "q1",
        answer: "123",
        isCorrect: true,
        timeSpent: 15,
        createdAt: "2025-06-18T15:30:45+09:00"
      },
      // ... 더 많은 답변
    ],
    gameVersion: "20250618000000",
    createdAt: "2025-06-18T15:30:45+09:00",
    updatedAt: "2025-06-18T15:30:45+09:00"
  }
}
```

## 리더보드

### 리더보드 조회

#### GET /api/leaderboard

전체 사용자의 게임 점수 순위를 조회합니다.

##### Request Body

```typescript
{
  limit?: number,      // 조회할 기록 수 (기본값: 10)
  offset?: number,     // 페이지네이션 오프셋 (기본값: 0)
  mode?: string,       // 모드 필터 ('normal' 또는 'hard', 기본값: 전체)
  period?: string      // 기간 필터 ('all', 'week', 'month', 기본값: 'all')
}
```

##### 필드 설명

- `limit`: 한 페이지에 표시할 기록 수
  - 기본값: 10
- `offset`: 페이지네이션 오프셋
  - 기본값: 0
- `mode`: 게임 모드 필터
  - 'normal': 일반 모드만 표시
  - 'hard': 하드 모드만 표시
  - 기본값: 전체 모드 표시
- `period`: 기간 필터
  - 'all': 모든 기간
  - 'week': 최근 1주일
  - 'month': 최근 1개월
  - 기본값: 'all'

##### Response Body

```typescript
{
  status: 200,
  message: string,
  data: {
    leaderboard: {
      rank: number,           // 순위
      userId: string,         // 사용자 ID
      userName: string,       // 사용자 이름
      userImage: string,      // 사용자 프로필 이미지 URL
      score: number,          // 최고 점수
      correctRate: number,    // 정답률 (백분율)
      averageTime: number,    // 평균 소요 시간 (초)
      gamesPlayed: number,    // 플레이한 게임 수
      lastPlayedAt: string    // 최근 플레이 일시 (ISO 문자열)
    }[],
    total: number,           // 전체 리더보드 수
    currentPage: number,     // 현재 페이지
    totalPages: number       // 전체 페이지 수
  }
}
```

##### 응답 필드 설명

- `leaderboard`: 리더보드 항목 리스트
  - `rank`: 순위
  - `userId`: 사용자 ID
  - `userName`: 사용자 이름
  - `userImage`: 사용자 프로필 이미지 URL
  - `score`: 최고 점수
  - `correctRate`: 정답률 (백분율)
  - `averageTime`: 평균 소요 시간 (초)
  - `gamesPlayed`: 플레이한 게임 수
  - `lastPlayedAt`: 최근 플레이 일시
- `total`: 전체 리더보드 항목 수
- `currentPage`: 현재 페이지 번호
- `totalPages`: 전체 페이지 수

##### 예시

```typescript
// 요청 예시 (최근 1주일 하드 모드 리더보드)
{
  limit: 10,
  offset: 0,
  mode: "hard",
  period: "week"
}

// 응답 예시
{
  status: 200,
  message: "Leaderboard fetched successfully",
  data: {
    leaderboard: [
      {
        rank: 1,
        userId: "123",
        userName: "Player1",
        userImage: "https://example.com/users/123/profile.jpg",
        score: 1000,
        correctRate: 95,
        averageTime: 15,
        gamesPlayed: 5,
        lastPlayedAt: "2025-06-18T00:00:00+09:00"
      },
      // ... 더 많은 항목
    ],
    total: 100,
    currentPage: 1,
    totalPages: 10
  }
}
```

### 나의 게임 기록 조회

#### GET /api/game/history

사용자의 게임 기록을 조회합니다. 기록은 최신순으로 정렬됩니다.

#### Request Query

```typescript
{
  limit?: number,      // 조회할 기록 수 (기본값: 10)
  offset?: number,     // 페이지네이션 오프셋 (기본값: 0)
  mode?: string,       // 모드 필터 ('normal' 또는 'hard', 기본값: 전체)
  period?: string,     // 기간 필터 ('all', 'week', 'month', 기본값: 'all')
  sortBy?: string,     // 정렬 기준 ('date', 'score', 'time', 기본값: 'date')
  sortOrder?: string,  // 정렬 순서 ('asc', 'desc', 기본값: 'desc')
  version?: string     // 클라이언트 버전
  timezone?: string    // 클라이언트 시간대
}
```

#### 필드 설명

- `limit`: 한 페이지에 표시할 기록 수
  - 기본값: 10
- `offset`: 페이지네이션 오프셋
  - 기본값: 0
- `mode`: 게임 모드 필터
  - 'normal': 일반 모드만 표시
  - 'hard': 하드 모드만 표시
  - 기본값: 전체 모드 표시
- `period`: 기간 필터
  - 'all': 모든 기간
  - 'week': 최근 1주일
  - 'month': 최근 1개월
  - 기본값: 'all'
- `sortBy`: 정렬 기준
  - 'date': 플레이 일시
  - 'score': 점수
  - 'time': 플레이 시간
  - 기본값: 'date'
- `sortOrder`: 정렬 순서
  - 'asc': 오름차순
  - 'desc': 내림차순
  - 기본값: 'desc'
- `version`: 클라이언트 버전
  - 기본값: 최신 버전
- `timezone`: 클라이언트 시간대
  - 기본값: Asia/Seoul

#### Response Body

```typescript
{
  status: number,
  message: string,
  data: {
    history: {
      id: string,             // 게임 ID
      mode: string,           // 게임 모드 ('normal' 또는 'hard')
      score: number,          // 점수
      correctCount: number,   // 정답 수
      totalQuestions: number, // 총 문제 수
      playTime: number,       // 플레이 시간 (초)
      averageTime: number,    // 평균 문제 소요 시간 (초)
      questions: {            // 문제별 통계
        correct: number,      // 정답 문제 수
        incorrect: number,    // 오답 문제 수
        averageTime: number   // 평균 문제 소요 시간 (초)
      },
      playedAt: string,       // 플레이 일시 (ISO 8601)
      version: string         // 게임 버전
      createdAt: string       // 게임 생성 시간 (ISO 8601)
      updatedAt: string       // 게임 업데이트 시간 (ISO 8601)
    }[],
    total: number,           // 전체 게임 수
    currentPage: number,     // 현재 페이지
    totalPages: number,      // 전체 페이지 수
    hasNext: boolean,        // 다음 페이지 존재 여부
    hasPrevious: boolean,    // 이전 페이지 존재 여부
    stats: {                 // 전체 통계
      totalGames: number,     // 총 게임 수
      totalScore: number,     // 총 점수
      averageScore: number,   // 평균 점수
      highestScore: number,   // 최고 점수
      averageCorrectRate: number, // 평균 정답률 (백분율)
      averagePlayTime: number,    // 평균 플레이 시간 (초)
      bestGame: {            // 최고 게임 기록
        id: string,          // 게임 ID
        score: number,       // 점수
        correctRate: number, // 정답률 (백분율)
        playTime: number,    // 플레이 시간 (초)
        playedAt: string     // 플레이 일시 (ISO 8601)
      }
    }
  }
}
```

#### Error Response

```typescript
// 기록이 존재하지 않을 때
{
  status: 404,
  message: "No game history found",
  errorType: "NO_HISTORY_FOUND",
  details: {
    filters: {
      mode: "hard",
      period: "week"
    }
  }
}

// 잘못된 기간 필터
{
  status: 400,
  message: "Invalid period",
  errorType: "INVALID_PERIOD",
  details: {
    period: "invalid-period"
  }
}
```

#### Rate Limit
- 100 requests/minute
- 500 requests/minute per IP
- 1000 requests/hour per user

#### Authentication
- Required
- Token: `woreboth_auth`

#### Example

```typescript
// Request
GET /api/v1/game/history?limit=10&offset=0&mode=hard&period=week&sortBy=score&sortOrder=desc

// Response
{
  status: 200,
  message: "Game history fetched successfully",
  data: {
    history: [
      {
        id: "game123",
        mode: "hard",
        score: 1000,
        correctCount: 10,
        totalQuestions: 10,
        playTime: 150,
        averageTime: 15,
        questions: {
          correct: 10,
          incorrect: 0,
          averageTime: 15
        },
        playedAt: "2025-06-18T15:30:45+09:00",
        version: "20250618000000",
        createdAt: "2025-06-18T15:30:45+09:00",
        updatedAt: "2025-06-18T15:30:45+09:00"
      },
      // ... 더 많은 기록
    ],
    total: 50,
    currentPage: 1,
    totalPages: 5,
    hasNext: true,
    hasPrevious: false,
    stats: {
      totalGames: 50,
      totalScore: 5000,
      averageScore: 100,
      highestScore: 1000,
      averageCorrectRate: 85,
      averagePlayTime: 120,
      bestGame: {
        id: "game123",
        score: 1000,
        correctRate: 100,
        playTime: 150,
        playedAt: "2025-06-18T15:30:45+09:00"
      }
    }
  }
}

### HTTP 400 Bad Request

- 잘못된 요청 형식

```typescript
{
  status: 400,
  message: "Invalid request format",
  errorType: "INVALID_REQUEST",
  details: {
    field: "teamA",
    message: "Team A ID is required"
  }
}
```

- 팀이 존재하지 않음

```typescript
{
  status: 400,
  message: "Team not found",
  errorType: "TEAM_NOT_FOUND",
  details: {
    teamId: "123"
  }
}
```

- 잘못된 리그 지정

```typescript
{
  status: 400,
  message: "Invalid league",
  errorType: "INVALID_LEAGUE",
  details: {
    league: "invalid-league"
  }
}
```

### HTTP 404 Not Found

- 팀 A와 팀 B를 뛴 선수가 없음

```typescript
{
  status: 404,
  message: "Player not found",
  errorType: "PLAYER_NOT_FOUND",
  details: {
    teamA: "team1",
    teamB: "team2"
  }
}
```

### HTTP 500 Internal Server Error

- 서버 내부 오류

```typescript
{
  status: 500,
  message: "Internal server error",
  errorType: "SERVER_ERROR",
  details: null
}
```
