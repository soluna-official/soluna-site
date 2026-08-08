# soluna.so

솔루나(Soluna) 사업자 소개 정적 사이트. 빌드 과정 없는 순수 HTML + CSS.

## 이 사이트가 하는 일

1. **사업체 실재 확인** — Dun & Bradstreet(D-U-N-S) 및 Google Play 조직 계정 심사에서
   "이 사업체가 실제로 존재하고 활동한다"는 근거가 된다. 그래서 푸터의 사업자 정보 표기가 핵심이다.
2. **스토어 심사 필수 URL 제공** — App Store / Google Play 심사에 요구되는
   개인정보처리방침 URL과 지원 URL을 제공한다.

| 용도 | URL |
|---|---|
| 랜딩 | `https://soluna.so/` |
| 개인정보처리방침 (스토어 제출용) | `https://soluna.so/privacy` |
| 앱 지원 (스토어 제출용) | `https://soluna.so/support` |

## 파일 구조

```
soluna-site/
├── index.html     랜딩 (소개 · 만드는 것 · 연락처 · 사업자 정보)
├── privacy.html   개인정보처리방침  → /privacy 로 서비스됨
├── support.html   앱 지원           → /support 로 서비스됨
├── styles.css     공통 스타일 (색 토큰 · 타이포 · 레이아웃)
├── favicon.svg    솔루나 마크 (sol + luna 일식)
└── README.md
```

`.html` 확장자 없이 `/privacy`, `/support`로 접근되는 것은 **Cloudflare Pages의 기본 동작**이다
(정적 파일의 `.html`을 자동으로 떼어낸다). 별도 설정 파일이 필요 없다.

## 로컬 실행

```bash
cd soluna-site
python3 -m http.server 8000
```

브라우저에서 `http://localhost:8000` 접속.

> ⚠️ 로컬 개발 서버에서는 `/privacy`가 404다. 이 확장자 생략은 Cloudflare Pages가 해주는 것이라
> `python3 -m http.server`에는 없다. 로컬에서는 `http://localhost:8000/privacy.html`로 확인할 것.
> 페이지 간 링크는 배포 기준(`/privacy`)으로 작성되어 있으므로 로컬에서 링크를 누르면 404가 뜨는 게 정상이다.

## 배포 — Cloudflare Pages

대시보드에서 진행한다.

1. **저장소 연결** — Cloudflare Dashboard → Workers & Pages → Create → Pages →
   Connect to Git → 이 저장소 선택
2. **빌드 설정**
   - Framework preset: `None`
   - Build command: **비워둠** (빌드 없음)
   - Build output directory: `/`
   - Root directory: `/` (저장소 루트에 파일이 있으므로 그대로)
3. **배포** — Save and Deploy. `*.pages.dev` 임시 주소가 먼저 생긴다.
4. **커스텀 도메인 연결** — 해당 Pages 프로젝트 → Custom domains → Set up a custom domain →
   `soluna.so` 입력. `www.soluna.so`도 붙이려면 같은 화면에서 한 번 더 추가한다.
5. **DNS** — 도메인이 이미 Cloudflare 네임서버를 쓰고 있으면 레코드가 자동 생성된다.
   가비아 등 외부 등록기관에 있으면 안내되는 CNAME을 등록기관 DNS에 추가한다.
6. **확인** — HTTPS 인증서 발급까지 보통 수 분. 아래 셋이 모두 200으로 열리는지 확인:
   `https://soluna.so/`, `https://soluna.so/privacy`, `https://soluna.so/support`

이후 `main` 브랜치에 push하면 자동 재배포된다.

## 앱 출시 전 반드시 처리할 것 (TODO)

사이트 안에 눈에 띄게 표시된 TODO 블록들이다. 앱 사양이 확정되면 채운다.

- [ ] **`privacy.html` 제1조 — 수집하는 개인정보 항목**
- [ ] **`privacy.html` 제4조 — 처리위탁(SDK) 표**
- [ ] **`support.html` — 앱 내 계정 삭제 메뉴 경로**
      (Google Play 계정 삭제 정책은 *앱 내 경로*와 *웹 요청 경로*를 모두 요구한다)
- [ ] **og:image** — 1200×630 대표 이미지 제작 후 세 페이지의 주석 처리된 메타 태그 해제

> ⚠️ **일치 요구사항**: `privacy.html`의 수집 항목·위탁 표는
> **Google Play Console의 데이터 보안(Data safety) 섹션** 및
> **App Store Connect의 앱 개인정보(App Privacy) 항목**과 100% 일치해야 한다.
> 불일치는 심사 반려 사유이며, 출시 후 발견되면 앱이 스토어에서 내려갈 수 있다.
> SDK를 하나 추가할 때마다(푸시·분석·크래시 리포트·광고) 이 문서를 **먼저** 고칠 것.

> ⚠️ **법률 자문 아님**: `privacy.html`은 국내 개인정보 보호법과 스토어 요구사항을 참고해 작성한
> **초안**이며 법률 자문이 아니다. 실제 서비스 개시 전 전문가 검토를 권한다.

## 디자인 메모

오늘수업(oneulclass) 랜딩과 **같은 가족으로 보이되 같은 페이지로는 보이지 않게** 만들었다.
오늘수업은 서비스이고 솔루나는 그것을 만드는 사업자라, 솔루나 쪽이 더 조용하고 정보 밀도가 낮다.

**가져온 것**
- Pretendard 폰트 스택 (CDN 버전도 동일: `pretendard@v1.3.9`)
- gray 스케일 — `#1F2024` / `#71727A` / `#E5E7EB`
- 본문 폭 760px, 12px 라운드
- **굵기 상한 600** — 오늘수업 tailwind 설정이 `bold`를 600으로 재정의한 특유의 규칙

**바꾼 것**
- primary 블루 `#3B82F6` → 솔루나 마크 색 (잉크 `#0b1322` + 태양 `#f5b840`)
- 차가운 배경 `#f8fafc` → 따뜻한 종이톤 `#fbfaf8`
- 카드·그라디언트·박스 밀도 제거, 행간 1.85로 확대
- 사업자 정보는 모노스페이스 콜로폰으로 처리

**기억에 남는 요소는 하나** — 해(sol)와 달(luna)이 겹친 일식 마크.
`oneulclass/docs/brand/soluna_mark.svg`에서 가져와 인라인 SVG로 넣었고,
서브페이지에서는 축소형이 홈 링크를 겸한다. 애니메이션은 쓰지 않았다.

**접근성**: 시맨틱 HTML, `:focus-visible` 포커스 링, 본문 건너뛰기 링크,
`prefers-reduced-motion` 존중(모션은 링크 밑줄 색 전환뿐), 375px 모바일 폭까지 반응형.

## 사업자 정보

전자상거래법상 표기 의무이자 D&B·스토어 심사 대조 근거다. 세 페이지 푸터에 동일하게 들어간다.
**값을 바꿀 때는 세 파일을 모두 고쳐야 한다** (빌드가 없어 공통 컴포넌트가 없다).

| 항목 | 값 |
|---|---|
| 상호 | 솔루나 (Soluna) |
| 대표자 | 이강산 (Lee Kangsan) |
| 사업자등록번호 | 720-17-02702 |
| 개업일 | 2025년 10월 23일 |
| 사업장 | 경기도 안산시 단원구 삼일로 310, 7층 706-43호 |
| 영문 주소 | 706-43, 310 Samil-ro, Danwon-gu, Ansan-si, Gyeonggi-do, 15368, Republic of Korea |
| 업태 | 정보통신업 (Information and communication) |
| 종목 | 응용 소프트웨어 개발 및 공급업 (Application software publishing) |
| 이메일 | hello@soluna.so |
