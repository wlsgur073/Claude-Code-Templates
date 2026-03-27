# 생태계 (Ecosystem)

바로 사용할 수 있는 Claude Code 컴포넌트 모음. 원하는 컴포넌트를 프로젝트의 `.claude/` 디렉토리에 복사하여 사용하세요.

## 카테고리

| 디렉토리 | 내용 |
|-----------|---------------|
| [skills/](skills/) | 재사용 가능한 멀티스텝 워크플로우 자동화 |
| [hooks/](hooks/) | 도구 사용 전/후 훅 설정 |
| [agents/](agents/) | 전문화된 에이전트 역할 정의 |

## 사용법

```bash
# 예시: 스킬을 프로젝트에 복사
cp -r ecosystem/skills/<skill-name> your-project/.claude/skills/

# 예시: 훅 설정을 settings.json에 병합
# (훅 항목을 .claude/settings.json에 수동으로 복사)
```

## 기여

유용한 스킬, 훅, 에이전트를 공유하고 싶으신가요? [CONTRIBUTING.md](../docs/CONTRIBUTING.md)를 참고하세요.

> 컴포넌트가 곧 추가될 예정입니다. 업데이트를 확인해주세요.
