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

## ⚠️ 이 저장소는 공개다 — 민감정보 금지

**사이트에 실제로 표시될 내용만** 넣는다. 커밋 메시지와 히스토리까지 영구히 공개된다.

넣지 말 것: 자격증명(키·토큰·`.env`) · 서버 설정 경로/내용/IP · 내부 운영 메모 ·
미공개 사업 정보 · 개인 연락처. 이런 내용은 비공개 저장소 `oneulclass`의 `docs/`로 보낸다.

사업자 정보 공개는 예외가 아니라 목적이다 — 전자상거래법상 표기 의무이자 심사 근거다.

> 상세 규칙과 push 전 점검 명령은 [`CLAUDE.md`](./CLAUDE.md) 참조.

## 파일 구조

```
soluna-site/
├── index.html     랜딩 (소개 · 만드는 것 · 연락처 · 사업자 정보)
├── privacy.html   개인정보처리방침  → /privacy 로 서비스됨
├── support.html   앱 지원           → /support
├── 404.html       존재하지 않는 경로 (Pages가 404 상태로 자동 서빙)
├── styles.css     공통 스타일 (색 토큰 · 타이포 · 레이아웃)
├── favicon.svg    솔루나 마크 (sol + luna 일식)
├── CLAUDE.md      AI 어시스턴트 가이드 (공개 저장소 규칙 · 구조 · 디자인)
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

### 전제: 네임서버를 Cloudflare로 옮겨야 한다

apex 도메인(`soluna.so`)에는 DNS 표준상 CNAME을 넣을 수 없고, Cloudflare Pages는 고정 A IP를
제공하지 않는다. 따라서 **가비아 DNS를 유지한 채로는 apex를 Pages에 연결할 수 없다.**
Cloudflare 네임서버를 써야 CNAME flattening으로 apex가 동작한다.

> 이것은 **도메인 이전(이관)이 아니다.** 도메인 등록·갱신은 계속 가비아에서 한다
> (만료 2027-08-15). 바뀌는 것은 "DNS 질의에 누가 답하느냐"뿐이고, 무료이며 언제든 되돌릴 수 있다.

**2026-08-08 기준 soluna.so 존 실측** (`dig`로 확인):

| 레코드 | 값 | 비고 |
|---|---|---|
| NS | ns.gabia.net / ns.gabia.co.kr / ns1.gabia.co.kr | 가비아 |
| A | **없음** | 웹이 존재한 적 없음 |
| MX | `1 smtp.google.com` | **Google Workspace — 끊기면 안 됨** |
| TXT | `v=spf1 include:_spf.google.com ~all` | SPF |
| TXT | `google-site-verification=G0ae8FNJUO_bAfmHKI_kU6QFZebY5G3Oo7i__1ITCxo` | |
| DKIM / DMARC | 미설정 | 별건 (아래 참고) |
| DNSSEC | **미적용** | 적용돼 있었다면 NS 변경 전 해제 필요 |

옮겨야 할 실제 레코드는 위 **3개(MX 1 + TXT 2)뿐**이다.

### 순서 (이 순서를 지켜야 메일이 안 끊긴다)

1. **Cloudflare에 존 추가** — Add a site → `soluna.so` → Free 플랜
2. **레코드 3개를 먼저 입력** — 자동 스캔이 가져오더라도 위 표와 눈으로 대조한다.
   ⚠️ **이 단계를 건너뛰고 4번으로 가면 `hello@soluna.so` 메일이 반송된다.**
3. Cloudflare가 알려주는 네임서버 2개(`○○.ns.cloudflare.com`)를 받아둔다
4. **가비아에서 네임서버 교체** — My가비아 → 도메인 관리 → soluna.so → 네임서버 설정 →
   기존 가비아 3개를 지우고 Cloudflare 2개 입력
5. **전파 대기** — 보통 수 분~수 시간(최대 48시간). Cloudflare 상태가 `Active`로 바뀐다.
   전파 중에는 구/신 네임서버가 같은 답을 주므로 메일은 무중단이다.
6. **Pages 프로젝트 생성** — Workers & Pages → Create → Pages → Connect to Git → 이 저장소
   - Framework preset: `None`
   - Build command: **비워둠** (빌드 없음)
   - Build output directory: `/`
   - Root directory: `/`
   - Save and Deploy → `*.pages.dev` 임시 주소가 먼저 생긴다
7. **커스텀 도메인 연결** — 해당 프로젝트 → Custom domains → `soluna.so` 추가.
   `www.soluna.so`도 붙이려면 한 번 더 추가한다. 존이 Cloudflare에 있으므로 레코드는 자동 생성된다.
8. **확인** — 인증서 발급까지 보통 수 분.
   ```bash
   dig +short NS soluna.so          # Cloudflare 네임서버로 바뀌었는지
   dig +short MX soluna.so          # 1 smtp.google.com 이 그대로 나오는지
   curl -sI https://soluna.so/         | head -1
   curl -sI https://soluna.so/privacy  | head -1
   curl -sI https://soluna.so/support  | head -1
   ```
   그리고 **외부 메일 주소에서 `hello@soluna.so`로 실제 발송 테스트**를 반드시 한다.
   D&B 연락이 이 주소로 오므로 이 검증을 건너뛰면 안 된다.

이후 `main` 브랜치에 push하면 자동 재배포된다.

### 되돌리기

가비아에서 네임서버를 원래 3개로 다시 넣으면 끝이다. 도메인 이전과 달리 잠금도 대기 기간도 없다.

### 대안 — GitHub Pages (네임서버를 안 건드리는 경우)

apex를 A 레코드로 지원하므로 가비아 DNS를 그대로 두고 붙일 수 있다(= 메일 리스크 0).
대신 무료 계정은 저장소가 공개여야 하고, 리다이렉트·헤더 제어가 약하다.
GitHub이 안내하는 apex용 A 레코드 4개 + `www` CNAME을 가비아에 추가하는 방식이다.

### 하지 말 것 — 기존 애플리케이션 서버에 얹기

기술적으로 가능하지만 권하지 않는다. privacy·support URL은 앱이 스토어에 올라가 있는
**내내** 살아 있어야 하는데, 앱 서버에 묶으면 배포 재시작·장애·인증서 갱신 실패가
그대로 심사 URL 장애가 된다. 별도 인증서를 직접 관리해야 하는 부담도 생긴다.

(검토 근거와 서버 설정별 주의사항은 oneulclass 저장소의 운영 문서에 정리해 둔다.)

## 참고 — DKIM / DMARC 미설정 (별건)

현재 soluna.so에 DKIM·DMARC가 없다. 필수는 아니지만, D&B·스토어와 주고받는 메일이
스팸으로 분류될 확률을 줄이려면 DNS를 만지는 김에 Google Workspace에서 DKIM을 켜고
DMARC TXT를 추가해두는 것이 좋다.

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

전자상거래법상 표기 의무이자 D&B·스토어 심사 대조 근거다. **네 페이지 푸터에 동일하게** 들어간다.
**값을 바꿀 때는 네 파일을 모두 고쳐야 한다** (빌드가 없어 공통 컴포넌트가 없다).

아래 값은 2026-08-08 발급 **사업자등록증명원과 글자 단위로 일치**시킨 것이다.
⚠️ 등록증 주의문에 따라 **영문 상호 `Soluna`는 임의기재로 법적 효력이 없다** — 법적 상호는 `솔루나`다.
(D-U-N-S·Play 조직명은 `Soluna`로 등록. 구글이 대조하는 것은 D-U-N-S 레코드다.)

| 항목 | 값 |
|---|---|
| 상호 | 솔루나 (Soluna) |
| 대표자 | 이강산 (Lee Kangsan) |
| 사업자등록번호 | 720-17-02702 |
| 개업일 | 2025년 10월 23일 |
| 사업장 | 경기도 안산시 단원구 삼일로 310, 7층 706-43호(선부동, 서울프라자) |
| 영문 주소 | 706-43, 310 Samil-ro, Danwon-gu, Ansan-si, Gyeonggi-do, 15368, Republic of Korea |
| 업태 | 정보통신업 (Information and communication) |
| 종목 | 응용 소프트웨어 개발 및 공급업 (Application software publishing) |
| 이메일 | hello@soluna.so |
