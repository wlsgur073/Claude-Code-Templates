# TaskFlow API 컨벤션

add-endpoint 스킬을 위한 빠른 참조. 전체 사양은 `docs/api-conventions.md`를 참고하세요.

## 라우트 패턴

- 컬렉션: `GET /api/<resource>` — 페이지네이션이 있는 목록
- 단건: `GET /api/<resource>/:id` — ID로 조회
- 생성: `POST /api/<resource>` — Zod 스키마로 body 검증
- 수정: `PATCH /api/<resource>/:id` — 부분 업데이트
- 삭제: `DELETE /api/<resource>/:id` — 소프트 삭제 (`deleted_at` 설정)

## 응답 Envelope

모든 응답은 `src/api/response.ts`의 `sendSuccess()` 또는 `sendError()`를 사용합니다:

```json
{ "ok": true, "data": { ... } }
{ "ok": false, "error": { "code": "NOT_FOUND", "message": "..." } }
```

## 파일 네이밍

- 핸들러: `src/api/<resource>.ts`
- 서비스: `src/services/<resource>-service.ts`
- 레포지토리: `src/repos/<resource>-repo.ts`
- 모델: `src/models/<resource>.ts`
- 테스트: `tests/services/<resource>-service.test.ts`, `tests/api/<resource>.test.ts`
