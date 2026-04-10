# LLM Wiki

LLM(Claude)이 유지·관리하는 마크다운 기반 지식 베이스입니다.

## 개요

사용자가 원본 문서를 `raw/`에 넣고 처리를 요청하면, LLM이 문서를 파싱하여 구조화된 위키 페이지를 생성·관리합니다. 축적된 지식을 활용하여 양식을 자동으로 채울 수도 있습니다. 한국 문서 형식(HWP, HWPX, PDF, XLSX)을 [kordoc](https://github.com/nicepkg/kordoc)으로 파싱합니다.

## 구조

```
LLM_wiki/
├── CLAUDE.md        # 위키 운영 규칙 (LLM 스키마)
├── .mcp.json        # kordoc MCP 서버 설정
├── raw/             # 원본 소스 (읽기 전용)
├── wiki/            # LLM이 관리하는 위키 페이지
│   ├── index.md     # 전체 페이지 카탈로그
│   ├── log.md       # 작업 로그
│   ├── overview.md  # 현황 대시보드
│   ├── sources/     # 소스별 요약
│   ├── projects/    # 프로젝트
│   ├── entities/    # 조직, 인물, 시스템
│   ├── concepts/    # 용어, 개념 정의
│   ├── reports/     # 분석, 질의 결과
│   └── issues/      # 이슈, 의사결정
├── templates/       # 양식 원본 (읽기 전용)
├── output/          # 채워진 양식 결과물
└── tools/           # (선택) 검색 스크립트 등
```

## 워크플로우

| 명령 | 설명 |
|------|------|
| **Ingest** | 원본 문서 → kordoc 파싱 → 위키 페이지 생성 → 인덱스/로그 갱신 |
| **Query** | 질문 → index.md 탐색 → 관련 페이지 읽기 → 출처 명시 답변 |
| **Fill** | 양식 파싱 → 위키 지식으로 빈 칸 매칭 → 미리보기 → 출력 |
| **Lint** | 모순, 고아 페이지, 빈 링크, 데이터 갭 점검 |

## 사용법

1. `raw/`에 문서를 넣는다
2. Claude에게 ingest를 요청한다
3. 위키에 질문한다
4. `templates/`에 양식을 넣고 채우기를 요청한다

## 기술 스택

- **문서 파싱**: [kordoc](https://github.com/nicepkg/kordoc) (MCP/CLI)
- **위키 형식**: Markdown + YAML frontmatter
- **내부 링크**: `[[페이지명]]` (Obsidian 호환)
