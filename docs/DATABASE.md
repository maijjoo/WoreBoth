# 데이터베이스 문서

## 팀 테이블
```sql
CREATE TABLE teams (
  id VARCHAR(255) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  country VARCHAR(100),
  league VARCHAR(100),
  last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 게임 세션 테이블
```sql
CREATE TABLE game_sessions (
  id SERIAL PRIMARY KEY,
  user_id VARCHAR(255),
  score INTEGER DEFAULT 0,
  settings JSONB,
  started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ended_at TIMESTAMP
);
```

## 게임 질문 테이블
```sql
CREATE TABLE game_questions (
  id SERIAL PRIMARY KEY,
  session_id INTEGER REFERENCES game_sessions(id),
  team_a VARCHAR(255) NOT NULL,
  team_b VARCHAR(255) NOT NULL,
  correct_answers JSONB,
  user_answer VARCHAR(255),
  is_correct BOOLEAN,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 인덱스
```sql
CREATE INDEX idx_teams_name ON teams(name);
CREATE INDEX idx_teams_country ON teams(country);
CREATE INDEX idx_teams_league ON teams(league);
CREATE INDEX idx_game_sessions_user ON game_sessions(user_id);
CREATE INDEX idx_game_questions_session ON game_questions(session_id);
```
