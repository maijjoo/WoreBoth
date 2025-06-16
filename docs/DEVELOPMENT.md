# 개발 문서

## 개발 환경 설정

### ESLint 설정
```json
{
  "extends": [
    "next/core-web-vitals",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/no-explicit-any": "off"
  }
}
```

### Husky 설정
```json
{
  "lint-staged": {
    "**/*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "**/*.{js,jsx}": ["eslint --fix", "prettier --write"],
    "**/*.json": ["prettier --write"],
    "**/*.md": ["prettier --write"]
  }
}
```

## 테스트 설정

### 단위 테스트
```bash
# 테스트 실행
pnpm test

# 테스트 커버리지 확인
pnpm test:coverage
```

### E2E 테스트
```bash
# E2E 테스트 실행
pnpm test:e2e

# 테스트 리포트 생성
pnpm test:e2e:report
```

## 성능 최적화

### 코드 분석
```bash
# 코드 복잡도 분석
pnpm analyze

# 번들 크기 확인
pnpm size
```

### 캐시 설정
```typescript
// 캐시 설정
const CACHE_DURATION = 7 * 24 * 60 * 60 * 1000; // 7일

// API 캐시
const apiCache = new Map<string, any>();

// 메모리 캐시
const memoryCache = new Map<string, any>();
```

## 배포

### 배포 전 검사
```bash
# 빌드 검사
pnpm build

# 타입 검사
pnpm type-check

# 보안 검사
pnpm audit
```

### 배포 명령어
```bash
# 프론트엔드 배포
pnpm deploy:frontend

# 백엔드 배포
pnpm deploy:backend
```
