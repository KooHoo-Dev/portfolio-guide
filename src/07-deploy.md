# Part 7. GitHub Pages(github.io) 배포

지금까지(Part 1~6) 만든 **이력서·기술 문서·포트폴리오·발표물**을 웹에 올려 **링크 하나로 공유**하는 방법입니다.
`GitHub Pages`는 GitHub가 제공하는 **무료 정적 웹 호스팅**으로, 서버도 비용도 필요 없습니다.

## 왜 배포하나요?

- 이력서·자기소개서에 `https://내아이디.github.io/portfolio` 링크 한 줄이면, 리크루터가 **클릭 한 번**에 봅니다.
- 무료 · 24시간 · 설치 없음. 파일만 올리면 끝.
- "내가 만든 걸 지금 바로 보여줄 수 있다"는 건 생각보다 강한 인상을 줍니다.

> 단, **기업 보안 정책상 외부 링크가 막힌 곳**도 있습니다. 그래서 링크와 함께 **PDF 사본**도 꼭 준비하세요.
> (Part 1·2에서 강조한 "최종 제출은 PDF"와 같은 맥락입니다.)

---

## 7-1. GitHub Pages란?

레포지토리의 **정적 파일**(HTML·CSS·JS·이미지)을 웹 주소로 게시해 주는 기능입니다.

- 주소 형식: `https://<GitHub아이디>.github.io/<레포이름>/`
  - 예) 이 가이드는 `https://koohoo-dev.github.io/portfolio-guide/` 로 배포돼 있습니다.
  - 레포 이름을 아예 `<아이디>.github.io`로 만들면 주소가 `아이디.github.io`(레포명 생략)로 짧아집니다.
- ⚠ **무료 플랜은 공개(public) 레포에서만** Pages가 동작합니다. (비공개는 유료 플랜 필요)
- ⚠ Pages는 `.md`를 **원문(raw)으로만** 서빙합니다 — 렌더가 필요하면 아래 7-5를 참고하세요.

---

## 7-2. 사전 준비

1. **GitHub 계정**
2. **공개 레포** 생성
3. 배포할 파일을 레포에 올리기(push). **진입점은 반드시 루트의 `index.html`** 입니다.

> Git 기본 사용법(`add`·`commit`·`push`)은 이 가이드의 범위를 벗어납니다. 명령어가 낯설면
> **GitHub Desktop** 같은 GUI 도구로 클릭만으로 파일을 올릴 수 있습니다.

---

## 7-3. 방법 A — 브랜치에서 배포 (입문자 추천)

빌드 과정이 없는 정적 파일(HTML·이미지)이라면 이 방법으로 충분합니다. **가장 쉽습니다.**

```mermaid
flowchart LR
    A[공개 레포 생성] --> B["파일 push<br/>(루트에 index.html)"]
    B --> C["Settings → Pages"]
    C --> D["Source:<br/>Deploy from a branch"]
    D --> E["Branch: main / (root)<br/>→ Save"]
    E --> F["1~2분 뒤<br/>github.io 주소 발급"]
```

**단계**

1. 레포 → **Settings** → 좌측 메뉴 **Pages**.
2. **Source**에서 **"Deploy from a branch"** 선택.
3. **Branch**를 `main`, 폴더를 `/(root)`로 지정하고 **Save**.
   - 문서를 `docs/` 폴더에 모아 뒀다면 폴더를 `/docs`로 선택하면 됩니다.
4. 1~2분 뒤 Pages 화면 상단에 `https://<아이디>.github.io/<레포>/` 주소가 뜹니다.

> 실제 화면은 GitHub 레포의 **Settings → Pages**에 있습니다.

---

## 7-4. 방법 B — GitHub Actions (자동 배포 · 심화)

push할 때마다 **자동으로** 배포하거나, 빌드가 필요하거나, 레포 전체를 그대로 게시하고 싶을 때 씁니다.
**이 가이드 프로젝트가 쓰는 방식**입니다.

1. 레포에 `.github/workflows/deploy-pages.yml` 워크플로우를 추가합니다.
2. Settings → Pages → **Source**를 **"GitHub Actions"** 로 설정합니다(최초 1회).
3. 이후 `main`에 push하면 워크플로우가 돌아 **자동 배포**됩니다.

```yaml
# .github/workflows/deploy-pages.yml (핵심만)
name: Deploy to GitHub Pages
on:
  push: { branches: [main] }
permissions:            # Pages 배포에 필요한 최소 권한
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    environment: github-pages
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: actions/upload-pages-artifact@v3
        with: { path: . }        # 레포 전체를 업로드
      - uses: actions/deploy-pages@v4
```

> 핵심 스텝: `configure-pages` → `upload-pages-artifact`(게시할 파일 업로드) → `deploy-pages`.
> 입문 단계에서는 **방법 A로 충분**합니다. 자동화·빌드가 필요해지면 그때 Actions로 넘어가세요.

---

## 7-5. 배포 후 & 자주 겪는 문제

| 방법 | 쉬움 | 자동 배포 | 언제 |
|---|---|---|---|
| **A. 브랜치 배포** | ★★★ | 브랜치 push 시 | 빌드 없는 정적 파일 (입문 추천) |
| **B. GitHub Actions** | ★★ | push마다 자동 | 빌드 필요·레포 전체 게시·세밀한 제어 |

**👎 자주 하는 실수 / 👍 이렇게**

- **빈 화면·404** — 진입점 `index.html`이 **루트**에 있나요? 브랜치/폴더 설정이 맞나요?
- **이미지·CSS가 안 떠요** — 절대경로(`/assets/...`)는 프로젝트 페이지에서 깨집니다. **상대경로**(`assets/...`)를
  쓰세요. 또한 Pages 서버는 리눅스라 **대소문자를 구분**합니다 (`Class.png` ≠ `class.png`).
- **비공개 레포** — 무료 플랜은 Pages 불가. 공개로 바꾸거나 유료 플랜이 필요합니다.
- **`.md`가 코드 텍스트로 보여요** — Pages는 마크다운을 렌더하지 않습니다. ① HTML로 만들거나 ② GitHub
  저장소 뷰(blob) 링크를 쓰거나 ③ 이 가이드의 `guide.html`처럼 **렌더러 페이지**를 두세요.
- **반영이 안 돼요** — 배포·CDN 전파에 1~2분 걸립니다. 강력 새로고침(Ctrl+F5)도 해보세요.

**팁**

- 이력서·자소서에는 **짧고 기억하기 쉬운 주소**가 좋습니다. (`아이디.github.io` 레포로 만들면 깔끔)
- **커스텀 도메인**도 연결할 수 있습니다 (Settings → Pages → Custom domain).
- 다시 강조 — **기업 보안상 외부 링크가 막힌 곳**이 있으니, 링크와 함께 **PDF 사본**을 꼭 함께 준비하세요.

> 이 가이드도 그래서 github.io에 올라가 있습니다 — `koohoo-dev.github.io/portfolio-guide`.
> 여러분의 포트폴리오도 링크 하나로 "지금 바로" 보여줄 수 있게 만들어 두세요.
