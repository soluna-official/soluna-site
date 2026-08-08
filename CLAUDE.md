# Claude Code 가이드 — soluna-site

솔루나 사업자 소개 정적 사이트. 빌드 과정 없는 순수 HTML + CSS.

## 응답 언어
- **한글 응답** 기본. 코드/기술 내용도 한글 설명과 함께.

## ⚠️ 이 저장소는 공개(Public)다 — 민감정보 금지

GitHub `soluna-official/soluna-site`는 **공개 저장소**다.
소스뿐 아니라 **커밋 메시지와 전체 히스토리가 영구히 공개**된다.
한 번 push되면 나중에 지워도 포크·캐시·아카이브에 남는다고 가정할 것.

### 넣어도 되는 것
**사이트에 실제로 표시될 내용만.** 그 외에는 기본적으로 넣지 않는다.

사업자 정보(상호·대표자·사업자등록번호·사업장 주소·업태/종목·`hello@soluna.so`)는
예외가 아니라 **목적**이다. 전자상거래법상 표기 의무이자 D&B·스토어 심사의 확인 근거라
공개되어야 정상이다.

### 넣지 말 것
- 자격증명 일체 — API 키, 토큰, 비밀번호, 인증서, `.env`
- **서버·인프라 정보** — 설정 파일 경로, 설정 내용 인용, 서버 IP, 인스턴스 ID, 배포 절차 상세
- **내부 운영 메모** — 검토 근거, 의사결정 기록, 장애 이력, 비용, 고객·리드 정보
- 미공개 사업 정보 — 매출, 계약, 출시 일정
- 개인 연락처 — 개인 휴대폰·자택 주소 등 (업무용 `hello@soluna.so`는 허용)

이런 내용은 **비공개 저장소인 `oneulclass`의 `docs/`로** 보낸다.
여기에는 결론만 남기고 "상세는 운영 문서 참조"로 넘긴다.

> 실제 사례(2026-08-08): README 배포 섹션에 oneulclass의 nginx 설정 파일 경로와
> 설정 내용을 인용했다가 push 직전에 발견해 제거했다. 자격증명은 아니었지만
> 내부 구조를 공개 저장소에 적을 이유가 없었다. **커밋 메시지에도 같은 내용이 있어
> amend까지 필요했다** — 본문만 고치면 히스토리에 남는다는 점을 기억할 것.

### push 전 점검

```bash
# 자격증명 패턴
grep -rniE "api[_-]?key|secret|token|password|private[_-]?key|\.env" . --exclude-dir=.git
# 인프라·내부 정보
grep -rniE "nginx|certbot|letsencrypt|ec2|docker|[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" . --exclude-dir=.git
# 커밋 메시지까지 확인 (본문만 고치면 히스토리에 남는다)
git log --format=%B | grep -niE "nginx|certbot|ec2|secret|key"
```

아직 push 전이면 `git commit --amend`로 히스토리까지 정리할 수 있다.
이미 push했다면 지워도 남는다고 보고, 노출된 자격증명은 **회전(rotate)**시킨다.

## 구조

```
soluna-site/
├── index.html     랜딩
├── privacy.html   개인정보처리방침  → /privacy
├── support.html   앱 지원           → /support
├── styles.css     공통 스타일
└── favicon.svg    솔루나 마크
```

- 빌드가 없어 **공통 컴포넌트가 없다.** 사업자 정보는 3개 파일 푸터에 중복 기재되어 있으니
  값을 바꿀 때는 **세 파일 모두** 고칠 것.
- 배포는 Cloudflare Pages(빌드 커맨드 없음). 자세한 절차는 `README.md`.
- `/privacy`·`/support` 확장자 생략은 Cloudflare Pages 기본 동작이라
  로컬 `python3 -m http.server`에서는 404가 정상이다.

## 앱 출시 전 TODO (페이지에 앰버 블록으로 표시됨)

`privacy.html`의 수집 항목·처리위탁 표는 **Google Play 데이터 보안 섹션** 및
**App Store 앱 개인정보 항목**과 100% 일치해야 한다. 불일치는 심사 반려 사유다.
SDK를 추가할 때마다 이 문서를 **먼저** 고칠 것.

## 디자인

오늘수업(oneulclass) 랜딩과 같은 가족으로 보이되 위계가 느껴지게 — 더 조용하고 밀도가 낮게.
승계: Pretendard 스택 · gray 스케일 · 760px 본문 폭 · **굵기 상한 600**.
변경: 색은 솔루나 마크(잉크 `#0b1322` + 태양 `#f5b840`), 따뜻한 종이톤 배경.
기억에 남는 요소는 마크 하나뿐 — 과한 애니메이션 금지, `prefers-reduced-motion` 존중.
