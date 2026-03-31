---
title: "TaskFlow API 엔드포인트 규칙"
description: "API 핸들러 파일 작업 시에만 적용되는 규칙"
paths:
  - "src/api/**/*.ts"
---

# API 엔드포인트 규칙

- 모든 엔드포인트는 `src/models/`의 Zod 스키마로 입력을 검증해야 함
- 모든 라우트 핸들러에 `asyncHandler` 래퍼 사용:
  ```typescript
  router.get('/tasks', asyncHandler(async (req, res) => { ... }))
  ```
- `src/api/response.ts`의 `sendSuccess()` 또는 `sendError()`를 사용하여 응답 반환
- 핸들러에서 레포지토리 메서드를 직접 호출하지 말 것 — 서비스를 통할 것
- 공개 엔드포인트에는 `src/api/middleware/rateLimit.ts`를 통해 Rate limiting 포함
- 모든 엔드포인트에 JSDoc 태그로 문서화 필수: `@route`, `@method`, `@auth`
