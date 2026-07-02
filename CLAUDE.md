# CLAUDE.md

이 문서는 **Claude Code가 이 프로젝트를 이어서 작업할 때 참조하는 컨텍스트**입니다.
프로젝트의 목적, 확정된 결정 사항, 작업 규칙, 코드 표준을 담습니다.

---

## 프로젝트 개요

**유니티 개발자 대상 취업·기술 문서 작성 가이드 v2.**
기존에 배포하던 "포트폴리오 문서 작성 가이드"의 업그레이드 버전으로,
취업 문서 작성법에 더해 **레이아웃·다이어그램·AI 활용**까지 다룬다.

- **대상 독자**: 유니티(C#) 개발 지망 수강생. AI 활용 수준은 "웹 챗봇으로 간단한
  코드·문서 생성이 가능한 입문~초급".
- **저자**: 김재훈 (Jay / KooHoo) — 유니티 강사, 1인 개발자.
- **두 산출물**: ① Markdown 소스(GitHub Repo에서 읽기) ② 프레젠테이션(Marp 발표 덱 + React 쇼케이스).

### 가이드 구성 (Part 0~6)

| Part | 파일 | 내용 |
|---|---|---|
| 0 | `src/00-orientation.md` | 오리엔테이션 (가이드 지도, 두 산출물, 도구 스택) |
| 1 | `src/01-job-documents.md` | 취업 문서 (이력서·자소서·포트폴리오) |
| 2 | `src/02-tech-documents.md` | 기술 문서 5단 구조 (개요→구조→기능→선택→회고) |
| 3 | `src/03-presentation-layouts.md` | 프레젠테이션 레이아웃 8종 + 스타일·테마 |
| 4 | `src/04-mermaid-diagrams.md` | Mermaid 다이어그램 5종 (AI 생성 전제) |
| 5 | `src/05-ai-workflow.md` | AI 활용 워크플로우 (웹 챗봇 기준) |
| 6 | `src/06-prompts.md` | 좋은 레이아웃 프롬프트 모음 (한글, 복붙 카드) |

---

## 폴더 구조

```
portfolio-guide-v2/
├── src/                    # Part 0~6 Markdown 소스 (읽기용 가이드 본문)
│   └── _example-spec.md    # 관통 예제(크래프팅) 스펙 — 내부 참조용, 변경 금지 기준선
├── assets/diagrams/        # Mermaid 5종: PNG(렌더링본) + mmd(소스)
├── slides/                 # Marp 발표 덱
│   ├── presentation.md     #   슬라이드 소스
│   ├── theme.css           #   커스텀 테마 (미니멀 + 다크 강조)
│   ├── presentation.pdf    #   빌드 결과 (PDF)
│   ├── presentation.html   #   빌드 결과 (HTML)
│   └── images/             #   슬라이드용 다이어그램 PNG (assets 복사본)
└── showcase/               # React 레이아웃 쇼케이스
    ├── index.html          #   단일 HTML (React CDN + Tailwind CDN)
    └── images/             #   쇼케이스용 다이어그램 PNG (assets 복사본)
```

> **이미지 중복 주의**: 다이어그램 PNG가 `assets/`, `slides/images/`, `showcase/images/`
> 세 곳에 복사돼 있다. 단일 소스로 통합할지는 아직 미결(아래 TODO 참조).

---

## 관통 예제: 크래프팅 시스템 (변경 금지 기준선)

Part 4~6과 발표물의 **모든 예시**는 이 하나의 예제로 관통한다. 상세는 `src/_example-spec.md`.

- **도메인**: 재료를 소비해 아이템을 제작. 확률 성공 / 실패 시 재료 50% 소비 / 이벤트로 결과 전파.
- **클래스 8개**: `CraftingManager`, `ICraftingService`, `Recipe`, `Inventory`, `Item`,
  `ItemStack`, `CraftingUI`, `CraftingResult`.
- **네임스페이스**: `Game.Crafting`
- **예시 레시피**: 나무 x3 + 돌 x2 → 돌도끼 x1 (successRate 0.8)
- **다이어그램 매핑**: 클래스=관계, 시퀀스=성공 흐름, 상태(FSM)=제작 전이,
  플로우차트=RequestCraft 분기, ER=저장 데이터 구조.

> 새 콘텐츠를 추가할 때도 이 예제를 유지해 일관성을 지킬 것. 예제를 바꿔야 한다면
> 먼저 `_example-spec.md`를 갱신하고, 의존하는 모든 Part·다이어그램·발표물을 함께 수정.

---

## 확정된 결정 사항 (Decisions)

이 프로젝트를 진행하며 확정한 사항들. 되돌리기 전에 이유를 검토할 것.

1. **두 산출물, 하나의 소스** — Markdown을 소스로, 발표물을 파생. md 일원화를 우선.
2. **발표 덱 = Marp** — github.io 배포 + PDF export + Markdown 소스 3조건을 만족.
   Mermaid는 Marp가 직접 렌더링하지 않으므로 **PNG 이미지로 삽입**.
3. **레이아웃 쇼케이스 = React** — Part 3의 8개 레이아웃을 "실물"로 보여주는 용도.
   **단일 HTML + React CDN(unpkg) + Tailwind CDN** 방식(입문자가 빌드 없이 파일 하나로 사용).
   정식 npm/Vite 프로젝트는 수강생 자율 학습 과제로만 언급(강제하지 않음).
4. **Mermaid 5종** — 클래스/시퀀스/상태(FSM)/플로우차트/ER. 테마는 **default**로 통일.
5. **Part 4는 "AI 생성 전제"** — 기본 문법은 저자가 별도 교육. 가이드는 "읽고 고치는 법"과
   완성본, AI 프롬프트에 집중.
6. **Mermaid → 이미지 변환은 웹 도구(mermaid.live) 중심** — CLI(mmdc)는 참고로만.
7. **AI 도구는 "웹 기반 AI 챗봇"으로 중립 표기** — 특정 제품명 고정하지 않음.
8. **프롬프트는 한글 + 복붙 카드 형식** — 코드 블록 프롬프트 + "이럴 때" 한 줄 설명.
9. **발표 덱 테마** — 미니멀(밝은 배경) 기본 + 코드/주의점 슬라이드만 다크 강조.
   강조색 청록 1색, 폰트 Pretendard(본문)·JetBrains Mono(코드).

---

## 가이드 콘텐츠 원칙 (문서를 쓸 때)

가이드 **본문을 작성·수정**할 때 지키는 원칙. (Jay 개인 코드 표준과 구분됨.)

- **핵심 철학**: "잘 읽히는 문서 + 역량을 드러내는 문서". 목적은 *잘 읽히는 문서로
  "함께 일하고 싶은 개발자"임을 증명하는 것.*
- **"이해 없이 복붙 금지"** — AI 결과물도 본인이 이해·검증해야 한다는 원칙을 관통 유지.
  기술 문서의 목적은 "코드를 짤 줄 안다"가 아니라 "내 코드를 설명할 줄 안다".
- **Jay 개인 코딩 규칙을 가이드 프롬프트에 강제하지 않음** — 수강생 배포용이므로,
  예시로 제시하되 각자 조정하도록 둔다. (Part 6 프롬프트에 SOLID·네이밍 강제 조항 없음.)
- **어투**: 정돈된 경어체. 표·예시(👎아쉬운/👍좋은)로 대비. 과한 포맷팅 지양.
- **"법칙이 아니다"** — 구조·규칙은 목적을 위한 수단이라는 태도를 유지.

---

## 코드 작성 표준 (C# 예제·스니펫을 쓸 때)

가이드 안의 **C# 예제 코드**를 작성하거나, Jay가 이어서 코드를 다룰 때 적용하는 표준.
(저자 Jay의 선호를 반영. 가이드 본문 정책과 별개.)

**아키텍처**
- 유지보수 가능한 코드 우선. **SOLID 원칙** 준수. 확장성·재사용성·디자인 패턴 활용.
- **느슨하게 결합된 상호작용** — 이벤트 기반 프로그래밍 선호. (예: `OnCraftCompleted` 이벤트)
- 인터페이스 의존(DIP) — 구현체가 아닌 계약에 의존. (예: `CraftingUI`는 `ICraftingService`에 의존)

**코드 스타일 (Microsoft C# 코딩 규칙)**
- `PascalCase` — 클래스·인터페이스·속성·메서드
- `camelCase` — 지역 변수·매개변수·private 필드
- **표현식 본문 멤버**(`=>`)로 간결하게
- **명확한 네임스페이스** 적용 (예제는 `Game.Crafting`)

**프로그래밍 선호**
- 성능 최적화: **지연 평가 및 캐싱** 선호
- 컬렉션 조작: **LINQ 적극 사용** (예: `recipes.FirstOrDefault(r => r.Id == recipeId)`)
- 테스트 용이성: 의존성 주입(예: `Func<float> roll`을 주입해 확률 판정을 결정론적으로 테스트)

**문서·도구 환경**
- 모든 문서는 **Markdown**.
- 선호 IDE: **JetBrains** 제품군 (Rider, WebStorm, DataGrip).

---

## 빌드 방법

**Marp 발표 덱** (slides/)
```bash
# HTML
marp presentation.md --theme theme.css --no-stdin -o presentation.html --html
# PDF (Chromium 필요)
marp presentation.md --theme theme.css --no-stdin --pdf --allow-local-files -o presentation.pdf
```

**Mermaid 다이어그램 렌더링** (assets/diagrams/)
- 입문자/일반: [mermaid.live](https://mermaid.live)에 `.mmd` 내용을 붙여 PNG/SVG로 export.
- CLI: `mmdc -i class.mmd -o class.png` (mermaid-cli 설치 필요).

**React 쇼케이스** (showcase/)
- `index.html`을 브라우저로 열면 됨(CDN 방식이라 빌드 불필요).

> **환경 참고**: 이 프로젝트를 만든 원 환경은 일부 CDN·브라우저 다운로드가 네트워크
> 정책으로 제한되어, Marp PDF는 로컬 Chromium 경로를 지정해 빌드했다. 일반 환경에서는
> 표준 명령이 그대로 동작한다.

---

## 다음 할 일 (TODO)

이전 후 이어서 진행할 작업. (Jay가 직접 하거나 Claude Code로 이어감.)

- [x] **이미지 중복 정리** — **현 3벌 유지로 결정.** `assets/diagrams/`가 원본(단일 소스),
      `slides/images/`·`showcase/images/`는 복사본. 쇼케이스·슬라이드를 각각 "폴더 하나만
      올리면 배포되는 자기완결 산출물"로 둔다는 결정 #3을 지키기 위함(통합 시 `../assets/`로
      경로가 결합돼 이식성이 깨짐). 다이어그램 수정 시 `assets/` 갱신 후 두 복사본에 반영할 것.
- [x] **GitHub Pages 배포 설정** — **허브 랜딩 방식으로 결정.** 루트 `index.html`(허브) →
      발표 덱(`slides/presentation.html`)·쇼케이스(`showcase/index.html`)·가이드 본문 링크.
      `.github/workflows/deploy-pages.yml`이 main push 시 **레포 전체를 그대로 게시**(재빌드 없음).
      Marp/Mermaid는 로컬 빌드 유지. 배포 URL: `https://koohoo-dev.github.io/portfolio-guide/`.
      **남은 수동 작업**: GitHub 레포 Settings → Pages → Source를 "GitHub Actions"로 1회 설정.
- [x] **경로 최종 점검** — `src/*.md`의 `../assets/diagrams/*.png`는 GitHub 렌더링 경로와 일치.
      슬라이드·쇼케이스는 각자 `images/*.png` 상대참조로 정상. 확인 완료.
- [x] **Part 3 ↔ 쇼케이스 상호 참조** — `03-presentation-layouts.md` 3-2절에 쇼케이스 링크 추가.
- [ ] **README 링크 점검** — 모든 내부 링크·이미지가 GitHub에서 정상 렌더링되는지 확인.
      (Pages 배포 후 실제 사이트에서 허브→발표 덱·쇼케이스 이동, 본문 이미지 표시 재확인 권장.)
- [ ] **(선택) 취업 문서 원본 복원 여부** — Part 1 기술 스택 표를 원본 전체로 확장할지,
      Part 2 '고민과 선택'에 PlayerPrefs vs JSON 사례를 부록으로 추가할지.

---

## 작업 규칙 (Jay와 협업 시)

- **파일 생성 작업은 먼저 역질문** — 부족한 점·보강할 점을 Jay에게 먼저 물은 뒤,
  답변을 원 요청과 합쳐 실행한다.
- Jay를 **Jay 또는 KooHoo**로 호칭.
- 큰 작업은 **Task 단위로 쪼개** 진행하고, 각 Task 후 확인받는다.
