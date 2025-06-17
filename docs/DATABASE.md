# 데이터베이스 문서

## 데이터베이스 개요
이 문서는 WoreBoth 게임의 데이터베이스 스키마를 설명합니다. 주요 테이블들의 구조, 관계, 인덱스 등을 포함합니다.

## 테이블 관계도
```
[teams] 1---* [players] *---1 [career]
[teams] *---1 [game_sessions]
[game_sessions] 1---* [game_questions]
```

## 테이블 설명

### 팀 테이블 (teams)
```sql
CREATE TABLE teams (
  id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  country VARCHAR(100),
  league VARCHAR(100),
  logo_url TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 설명
- 각 팀의 기본 정보를 저장합니다.
- `logo_url`: 팀 로고 이미지 URL
- `updated_at`: 데이터의 마지막 업데이트 시점

#### 인덱스
```sql
CREATE INDEX idx_teams_league ON teams(league);
CREATE INDEX idx_teams_name ON teams(name);
```

### 선수 테이블 (players)
```sql
CREATE TABLE players (
  id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  nationality VARCHAR(100),
  position VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 설명
- 각 선수의 기본 정보를 저장합니다.
- `name`: 선수 이름
- `nationality`: 선수 국적
- `position`: 선수 포지션 (예: GK, DF, MF, FW)
- `updated_at`: 선수 정보의 마지막 업데이트 시점

#### 인덱스
```sql
CREATE INDEX idx_players_name ON players(name);
CREATE INDEX idx_players_nationality ON players(nationality);
```

### 선수 경력 테이블 (career)
```sql
CREATE TABLE career (
  player_id VARCHAR(255) REFERENCES players(id),
  club_id VARCHAR(255) REFERENCES teams(id),
  from_year INTEGER,
  to_year INTEGER,
  PRIMARY KEY (player_id, club_id)
);
```

#### 설명
- 선수의 경력을 저장합니다.
- `from_year`: 경력 시작 연도
- `to_year`: 경력 종료 연도
- 복합 기본키: 선수-클럽 쌍을 고유하게 식별

#### 인덱스
```sql
CREATE INDEX idx_career_player ON career(player_id);
CREATE INDEX idx_career_club ON career(club_id);
```

### 게임 세션 테이블 (game_sessions)
```sql
CREATE TABLE game_sessions (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(255),
  score INTEGER DEFAULT 0,
  settings JSONB,
  started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ended_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 설명
- 게임 세션의 정보를 저장합니다.
- `user_id`: 사용자 ID
- `score`: 게임 점수
- `settings`: 게임 설정 (팀 A, 리그 필터 등)
- `started_at`: 게임 시작 시간
- `ended_at`: 게임 종료 시간

#### 인덱스
```sql
CREATE INDEX idx_game_sessions_user ON game_sessions(user_id);
```

### 게임 질문 테이블 (game_questions)
```sql
CREATE TABLE game_questions (
  id SERIAL PRIMARY KEY,
  session_id INTEGER REFERENCES game_sessions(id),
  team_a_id VARCHAR(255) REFERENCES teams(id),
  team_b_id VARCHAR(255) REFERENCES teams(id),
  answered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 설명
- 각 게임 세션의 질문 기록을 저장합니다.
- `session_id`: 게임 세션 ID
- `team_a_id`: 팀 A ID
- `team_b_id`: 팀 B ID
- `answered_at`: 답변 시점

#### 인덱스
```sql
CREATE INDEX idx_questions_session ON game_questions(session_id);
CREATE INDEX idx_questions_teams ON game_questions(team_a_id, team_b_id);
```

## 데이터베이스 최적화

### 인덱스 전략
- 자주 사용되는 조회 조건에 인덱스 추가
- 복합 인덱스는 적절히 사용하여 쿼리 성능 최적화
- 자주 업데이트되는 컬럼은 인덱스를 신중히 추가

### 데이터 정합성
- 외래 키 제약 조건으로 데이터 무결성 보장
- 트랜잭션으로 데이터 일관성 유지
- 적절한 기본값과 제약 조건 사용

### 성능 최적화
- 적절한 인덱스 사용으로 쿼리 성능 향상
- 정규화와 반정규화의 균형 잡기
- 캐싱 전략 적용
- 배치 처리로 대량 데이터 처리 최적화

#### 인덱스 목록
```sql
CREATE INDEX idx_teams_league ON teams(league);
CREATE INDEX idx_teams_name ON teams(name);
CREATE INDEX idx_players_name ON players(name);
CREATE INDEX idx_players_nationality ON players(nationality);
CREATE INDEX idx_career_player ON career(player_id);
CREATE INDEX idx_career_club ON career(club_id);
CREATE INDEX idx_game_sessions_user ON game_sessions(user_id);
CREATE INDEX idx_questions_session ON game_questions(session_id);
CREATE INDEX idx_questions_teams ON game_questions(team_a_id, team_b_id);
```
