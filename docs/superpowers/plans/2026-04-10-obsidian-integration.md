# Obsidian 연동 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** LLM_wiki 폴더를 Obsidian vault로 직접 사용할 수 있도록 설정 파일을 구성한다.

**Architecture:** `.obsidian/` 디렉토리에 vault 설정 파일들을 생성하고, `.gitignore`로 git 추적에서 제외한다. CLAUDE.md에 Obsidian 연동 규칙을 추가한다.

**Tech Stack:** Obsidian (vault 설정 JSON), Git (.gitignore), Markdown (CLAUDE.md)

**Note:** 이 작업은 전부 설정 파일이므로 TDD 대신 Obsidian 실행으로 검증한다.

---

## 파일 구조

| 작업 | 파일 | 역할 |
|------|------|------|
| 생성 | `.gitignore` | `.obsidian/` git 제외 |
| 생성 | `.obsidian/app.json` | vault 기본 설정 (새 파일 위치, 첨부파일 경로, wikilinks, 폴더 제외) |
| 생성 | `.obsidian/core-plugins-migration.json` | 코어 플러그인 활성화 상태 |
| 수정 | `CLAUDE.md:36` | Obsidian 연동 규칙 추가 (핵심 원칙 섹션 뒤) |

---

### Task 1: `.gitignore` 생성

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: `.gitignore` 파일 생성**

```
.obsidian/
```

- [ ] **Step 2: git에서 제외 확인**

Run: `git status`
Expected: `.gitignore`가 untracked로 표시되고, 이후 `.obsidian/` 폴더는 추적되지 않음

- [ ] **Step 3: 커밋**

```bash
git add .gitignore
git commit -m "chore: .gitignore 추가 — .obsidian/ 제외"
```

---

### Task 2: `.obsidian/app.json` 생성

**Files:**
- Create: `.obsidian/app.json`

- [ ] **Step 1: `.obsidian/` 디렉토리 생성**

```bash
mkdir -p .obsidian
```

- [ ] **Step 2: `app.json` 작성**

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

설정 설명:
- `newFileLocation: "folder"` + `newFileFolderPath: "raw"` — 새 파일 생성 시 `raw/`에 저장
- `attachmentFolderPath: "raw/assets"` — 첨부파일(이미지 등) `raw/assets/`에 저장
- `useMarkdownLinks: false` — `[[Wikilinks]]` 형식 사용
- `showFrontmatter: true` — YAML frontmatter 표시
- `userIgnoreFilters` — `tools/`, `raw/done/`, `docs/` 탐색에서 제외

- [ ] **Step 3: git status로 `.obsidian/`이 무시되는지 확인**

Run: `git status`
Expected: `.obsidian/app.json`이 표시되지 않음 (`.gitignore`에 의해 제외)

---

### Task 3: `.obsidian/core-plugins-migration.json` 생성

**Files:**
- Create: `.obsidian/core-plugins-migration.json`

- [ ] **Step 1: `core-plugins-migration.json` 작성**

이 파일은 Obsidian 코어 플러그인의 활성화/비활성화 상태를 제어한다.

```json
{
  "file-explorer": true,
  "global-search": true,
  "graph": true,
  "backlink": true,
  "page-preview": true,
  "tag-pane": true,
  "outline": true,
  "file-recovery": true,
  "command-palette": true,
  "markdown-importer": false,
  "word-count": true,
  "switcher": true,
  "starred": true,
  "note-composer": false,
  "daily-notes": false,
  "random-note": false,
  "publish": false,
  "sync": false,
  "canvas": false,
  "slides": false,
  "audio-recorder": false,
  "templates": false,
  "workspaces": false,
  "zk-prefixer": false
}
```

활성화 플러그인 요약:
- `file-explorer` — 폴더 구조 탐색
- `global-search` — 위키 내 전문 검색
- `graph` — `[[링크]]` 관계 시각화
- `backlink` — 역방향 링크 탐색
- `page-preview` — 링크 hover 시 미리보기
- `tag-pane` — frontmatter 태그 필터링
- `outline` — 문서 목차 표시
- `file-recovery`, `command-palette`, `word-count`, `switcher`, `starred` — 기본 유틸리티

- [ ] **Step 2: git status로 `.obsidian/`이 여전히 무시되는지 확인**

Run: `git status`
Expected: `.obsidian/` 관련 파일 없음

---

### Task 4: `CLAUDE.md` 업데이트

**Files:**
- Modify: `CLAUDE.md:39` (핵심 원칙 섹션 뒤, 페이지 컨벤션 섹션 앞)

- [ ] **Step 1: CLAUDE.md에 Obsidian 연동 섹션 추가**

`### 핵심 원칙` 블록과 `---` 구분선 사이, `## 페이지 컨벤션` 앞에 다음 내용을 삽입:

```markdown
### Obsidian 연동

- 이 디렉토리는 Obsidian vault로도 사용된다.
- `wiki/` 페이지 작성 시 Obsidian 호환성을 유지한다:
  - 내부 링크는 `[[페이지명]]` 형식
  - YAML frontmatter 필수
  - 마크다운 표준 문법 사용
- 사용자는 Obsidian에서 `raw/`에 자료를 투입하고, `wiki/`는 주로 조회용으로 사용한다.
- `.obsidian/` 설정은 git에서 제외된다 (`.gitignore`).
```

- [ ] **Step 2: 커밋**

```bash
git add CLAUDE.md
git commit -m "docs: CLAUDE.md에 Obsidian 연동 규칙 추가"
```

---

### Task 5: Obsidian에서 vault 열어서 검증

- [ ] **Step 1: Obsidian 실행 후 vault 열기**

1. Obsidian 실행
2. "Open folder as vault" 선택
3. `C:\Users\dlads\0. project Folder\llm_wiki\LLM_wiki` 경로 선택
4. "Trust author and enable plugins" 선택

- [ ] **Step 2: 설정 확인**

확인 항목:
- 좌측 파일 탐색기에 `raw/`, `wiki/`, `templates/`, `output/` 폴더가 보이는지
- `tools/`, `raw/done/`, `docs/` 폴더가 **보이지 않는지**
- `wiki/index.md`를 열어서 frontmatter가 표시되는지
- `[[페이지명]]` 링크가 보라색으로 렌더링되는지

- [ ] **Step 3: Graph View 확인**

1. 좌측 아이콘에서 Graph View 클릭 (또는 Ctrl+G)
2. `wiki/` 내 페이지들이 노드로 표시되는지 확인

- [ ] **Step 4: 자료 투입 테스트**

1. Obsidian에서 Ctrl+N으로 새 노트 생성
2. 새 노트가 `raw/` 폴더에 생성되는지 확인
3. 테스트 노트 삭제
