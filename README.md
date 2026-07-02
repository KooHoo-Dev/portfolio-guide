# 유니티 개발자 취업·기술 문서 작성 가이드 v2

유니티(C#) 개발 지망생을 위한 **취업 문서 · 기술 문서 작성 가이드**입니다.
문서를 *무엇을 쓸지*에 더해, *어떻게 보여줄지*(레이아웃·다이어그램)와
*어떻게 빠르게 만들지*(AI 워크플로우)까지 다룹니다.

> 목적은 하나 — **잘 읽히는 문서로 "함께 일하고 싶은 개발자"임을 증명하는 것.**

## 구성

| Part | 내용 | 문서 |
|---|---|---|
| 0 | 오리엔테이션 | [00-orientation](src/00-orientation.md) |
| 1 | 취업 문서 (이력서·자소서·포트폴리오) | [01-job-documents](src/01-job-documents.md) |
| 2 | 기술 문서 5단 구조 | [02-tech-documents](src/02-tech-documents.md) |
| 3 | 프레젠테이션 레이아웃 8종 | [03-presentation-layouts](src/03-presentation-layouts.md) |
| 4 | Mermaid 다이어그램 5종 | [04-mermaid-diagrams](src/04-mermaid-diagrams.md) |
| 5 | AI 활용 워크플로우 | [05-ai-workflow](src/05-ai-workflow.md) |
| 6 | 좋은 레이아웃 프롬프트 모음 | [06-prompts](src/06-prompts.md) |

## 두 가지 산출물

이 가이드는 **하나의 Markdown 소스**에서 두 형태로 나온다.

- **읽기용 가이드** — `src/`의 `.md` 문서. GitHub에서 Mermaid가 네이티브 렌더링된다.
- **발표·배포용**
  - **발표 덱 (강의용)** — `slides/deck.html` (단일 HTML + React CDN, 방향키로 넘기는 인터랙티브 덱)
  - **React 레이아웃 쇼케이스** — `showcase/index.html` (Part 3의 8개 레이아웃 실물 예시)
  - **(백업) Marp 발표 덱** — `slides/presentation.md` → HTML/PDF (구버전, 참고용 보관)

## 폴더 구조

```
portfolio-guide-v2/
├── src/                 # Part 0~6 가이드 본문 (Markdown)
├── assets/diagrams/     # Mermaid 다이어그램 (PNG + mmd 소스)
├── slides/              # Marp 발표 덱 (md + theme.css + PDF/HTML)
├── showcase/            # React 레이아웃 쇼케이스 (단일 HTML)
├── CLAUDE.md            # Claude Code용 프로젝트 컨텍스트
└── README.md
```

## 다이어그램

관통 예제인 **크래프팅 시스템**을 5종 다이어그램으로 표현한다.

| 다이어그램 | 답하는 질문 | 파일 |
|---|---|---|
| 클래스 | 무엇으로 이루어졌나 (런타임 구조) | `assets/diagrams/class.*` |
| 시퀀스 | 시간 순서로 어떻게 상호작용하나 | `assets/diagrams/sequence.*` |
| 상태(FSM) | 어떤 상태를 오가나 | `assets/diagrams/state.*` |
| 플로우차트 | 어떤 갈림길을 지나나 (로직) | `assets/diagrams/flow.*` |
| ER | 데이터가 어떻게 저장되나 (저장 구조) | `assets/diagrams/er.*` |

## 사용법

**가이드 읽기** — `src/`의 `.md` 파일을 GitHub에서 순서대로 읽으면 된다.

**발표 덱 (강의용)** — `slides/deck.html`을 브라우저로 열면 된다(CDN 방식, 빌드 불필요). ← → 로 이동.

**레이아웃 쇼케이스** — `showcase/index.html`을 브라우저로 열면 된다(빌드 불필요).

**(백업) Marp 덱 빌드** (Marp 필요, 구버전 참고용)
```bash
cd slides
marp presentation.md --theme theme.css --no-stdin -o presentation.html --html
marp presentation.md --theme theme.css --no-stdin --pdf --allow-local-files -o presentation.pdf
```

## 크레딧

- 저자: 김재훈 (Jay / KooHoo)
- 문서 형식: Markdown · 다이어그램: Mermaid · 발표: Marp · 쇼케이스: React

---

> 이어서 작업하거나 Claude Code로 협업할 때는 [CLAUDE.md](CLAUDE.md)의
> 결정 사항·코드 표준·TODO를 먼저 확인하세요.
