# Obsidian 연동 설계

## 개요

LLM_wiki 폴더를 Obsidian vault로 직접 열어, Claude가 생성/관리하는 위키 페이지를 Obsidian에서 탐색하고, Obsidian에서 raw/에 자료를 투입할 수 있게 한다.

## 배경

- LLM_wiki는 이미 Obsidian 호환 형식(`[[Wikilinks]]`, YAML frontmatter, Markdown)으로 구성됨
- 별도 동기화 없이 같은 폴더를 vault로 열면 양방향 반영이 자동으로 이루어짐
- Obsidian은 조회/탐색 + 자료 투입 용도, wiki/ 편집은 Claude가 담당

## 접근 방식

**A안 채택: Vault 직접 연결**

LLM_wiki/ 폴더 자체를 Obsidian vault로 연다. 다른 방식(symlink, 별도 vault + 동기화 스크립트) 대비 설정이 단순하고 유지보수가 거의 없다.

검토한 대안:
- B안 (Symlink): Windows symlink 권한 문제, 링크 깨질 위험
- C안 (별도 vault + 동기화): 스크립트 유지보수 필요, 충돌 처리 복잡

## 설계

### 1. `.gitignore` 변경

`.obsidian/` 디렉토리를 git 추적에서 제외한다.

```
.obsidian/
```

### 2. `.obsidian/app.json` — Vault 기본 설정

```json
{
  "newFileLocation": "folder",
  "newFileFolderPath": "raw",
  "attachmentFolderPath": "raw/assets",
  "useMarkdownLinks": false,
  "showFrontmatter": true,
  "userIgnoreFilters": ["tools/", "raw/done/", "docs/"]
}
```

- `newFileLocation: "folder"` + `newFileFolderPath: "raw"` — Obsidian에서 새 파일 생성 시 raw/에 저장
- `attachmentFolderPath: "raw/assets"` — 첨부파일(이미지 등) raw/assets/에 저장
- `useMarkdownLinks: false` — `[[Wikilinks]]` 형식 사용 (프로젝트 컨벤션 일치)
- `showFrontmatter: true` — YAML frontmatter 표시

### 3. `.obsidian/core-plugins.json` — 코어 플러그인

활성화할 코어 플러그인:

| 플러그인 | ID | 이유 |
|---------|-----|------|
| Graph View | `graph` | `[[링크]]` 관계 시각화 |
| Tags | `tag-pane` | frontmatter 태그 기반 필터링 |
| Backlinks | `backlink` | 역방향 링크로 관련 페이지 탐색 |
| Search | `search` | 위키 내 전문 검색 |
| Page Preview | `page-preview` | 링크 hover 시 미리보기 |
| File Explorer | `file-explorer` | 폴더 구조 탐색 |
| Outline | `outline` | 문서 목차 표시 |

### 4. `.obsidian/core-plugins-migration.json` — 플러그인 활성화 상태

위 플러그인들을 활성화하는 설정 파일.

### 5. 폴더 제외 설정

Obsidian 탐색에서 제외할 폴더:
- `tools/` — 스크립트 폴더, Obsidian에서 볼 필요 없음
- `raw/done/` — 파싱 완료 파일, 탐색 노이즈 방지
- `docs/` — 설계 문서, Obsidian 위키와 무관

`.obsidian/app.json`의 `userIgnoreFilters`로 설정.

### 6. `CLAUDE.md` 업데이트

CLAUDE.md에 Obsidian 연동 관련 규칙 추가:

```markdown
## Obsidian 연동

- 이 디렉토리는 Obsidian vault로도 사용된다.
- `wiki/` 페이지 작성 시 Obsidian 호환성을 유지한다:
  - 내부 링크는 `[[페이지명]]` 형식
  - YAML frontmatter 필수
  - 마크다운 표준 문법 사용
- 사용자는 Obsidian에서 `raw/`에 자료를 투입하고, `wiki/`는 주로 조회용으로 사용한다.
- `.obsidian/` 설정은 git에서 제외된다 (`.gitignore`).
```

### 7. 워크플로우

**자료 투입 (Obsidian → Claude):**
1. 사용자가 Obsidian에서 raw/에 파일 드래그&드롭 또는 복사
2. Claude Code에서 "ingest 해줘" 요청
3. Claude가 기존 ingest 워크플로우 수행
4. 결과가 Obsidian에 자동 반영

**위키 조회 (Claude → Obsidian):**
1. Claude가 wiki/에 페이지 생성/수정
2. Obsidian에서 바로 보임
3. Graph View, 검색, 백링크로 탐색

## 구현 범위

### 포함

1. `.gitignore` — `.obsidian/` 제외 추가
2. `.obsidian/app.json` — vault 기본 설정
3. `.obsidian/core-plugins.json` — 코어 플러그인 목록
4. `.obsidian/core-plugins-migration.json` — 플러그인 활성화 상태
5. `CLAUDE.md` — Obsidian 연동 규칙 추가
6. git commit

### 제외

- 커뮤니티 플러그인 설치 (필요 시 나중에)
- 동기화 스크립트 (같은 폴더이므로 불필요)
- Obsidian 테마 커스터마이징
- wiki/ 편집 제한 강제 (컨벤션으로 관리)
- workspace.json (Obsidian이 첫 실행 시 자동 생성)
