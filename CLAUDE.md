# tubechecker-studio

## 무엇인가
- `index.html`: Script Studio(유튜브 대본 검수기). 빌드 없는 단일 HTML, 한국어 UI. OpenAI/Gemini/Claude API를 브라우저에서 직접 호출. 접속 코드 로그인, 레퍼런스 톤, 파트 분할 검수, 히스토리(localStorage).
- `docs/overseas-youtube-playbook.md`: 해외 유튜브 시장 공략집. 시장 분석(일본 1순위, 대만 2순위, 베트남·태국은 레퍼런스 수집용), "상품 녹이기" 광고 기획 엔진 설계, 스토리보드·소스 조달 설계, 쇼츠레이더 국가별 수집 사양. **이 프로젝트 맥락이 필요하면 먼저 읽을 것.**

## 진행 상태 (2026-09-30)
- 엔진 만들 순서: ① 레퍼런스 탭 ② 상품 녹이기 ③ 스토리보드·소스 ④ 검수. 모두 `index.html`에 탭으로 추가.
- **① 레퍼런스 탭, ② 상품 녹이기 탭 완료.** 다음은 ③ 스토리보드·소스(컷별 소스 조달, Pexels/Pixabay 검색, SRT, 밋밋 감지).
- 쇼츠레이더는 별도 도구(사용자 PC, 이 저장소에 없음). 수집 국가: KR, JP, US, VN, TH, TW.

## 레퍼런스 탭 구조 (`index.html` 두 번째 `<script>`)
- 흐름: 링크 → (선택) YouTube Data API로 조회수·구독자·댓글 → Gemini `generateContent`에 `fileData.fileUri`로 유튜브 링크 전달 → JSON 포맷 카드 → `normCard()`로 정리 → 라이브러리.
- 쇼츠(또는 3분 이하)는 `videoMetadata.fps=2`로 요청하고, 400이 나면 fps 없이 한 번 재시도.
- 카드 스키마는 `REF_PROMPT` 안의 JSON 형식이 기준. 필드를 바꾸면 `normCard`·`refCardHTML`·`refMd`를 같이 고칠 것.
- localStorage: `fss-ref`(라이브러리, 최대 150개) · `fss-yk`(YouTube 키) · `fss-rm`(분석 모델) · `fss-rmx`(직접 입력한 모델) · `fss-rc`(나라) · `fss-tab`(마지막 탭). Gemini 키는 기존 `fss-k.gemini`를 대본 검수와 같이 씀.
- 새 기능은 CORS 프록시(`fetchWithProxy`)를 쓰지 않는다. Google API는 브라우저 직접 호출이 되고, 프록시를 쓰면 키가 제3자에게 간다.
- 모델 출력·YouTube 메타데이터는 전부 `esc()`/`escA()`로 이스케이프해서 넣는다.

## 상품 녹이기 탭 구조 (`index.html` 세 번째 `<script>`)
- 흐름: 레퍼런스(라이브러리 카드 또는 붙여넣기) + 상품 + 타깃 → 1차 호출: 긴장·상품 역할·포맷 점수·컨셉 3개 → 컨셉 선택 → 2차 호출: 타임코드 대본·업로드 세트·2편·AI 자체 점검.
- 엔진은 OpenAI·Gemini·Claude 중 선택(`APIS`의 url/hdr/ext 재사용, 시스템 프롬프트는 `MELT_SYS`). JSON 모드(OpenAI `response_format`, Gemini `responseMimeType`)가 400이면 JSON 모드 없이 한 번 재시도.
- 레퍼런스는 `meltRefBrief()`로 구조만 넘기고 대사는 빼서 보낸다(복사 방지).
- 광고 표시 문구(`MELT_C[나라].disc/pin`), Amazon·쿠팡 제휴 고지, `{LINK}` 치환은 AI에 맡기지 않고 `meltPost()`가 코드로 붙인다.
- `meltAutoChecks()`: 상품 등장 2~3번, 첫 3초 상품 없음, 반전 위치 65~88%, 영상 안 구매·가격·링크 표현(`MELT_BUY_RX`), 4초 넘는 비트, 촬영 비트 3개 이하, 상품명 언급.
- localStorage: `fss-melt`(기획 히스토리, 최대 30) · `fss-mform`(입력값) · `fss-prod`(저장한 상품) · `fss-mp`(엔진) · `fss-mm`(엔진별 모델) · `fss-mmx`(직접 입력 모델). 키는 `fss-k` 공유.
- AI 출력 정리는 공용 도우미 `nS/nN/nT/nA/nO`와 `normConcepts`·`normScript`.

## 테스트
- `NODE_PATH=$(npm root -g) node tests/e2e.js` — Playwright로 Gemini·YouTube 응답을 가짜로 바꿔 레퍼런스 탭, 상품 녹이기 탭(세 엔진), 기존 대본 검수 흐름을 확인한다(과금 없음). 녹이기 가짜 응답(`MELT_CONCEPTS`·`MELT_SCRIPT`)은 상어 레퍼런스 + 원터치 우산 일본 타깃 예시. 수정 후 커밋 전에 돌릴 것.

## 작업 규칙
- 모든 기능이 `index.html` 하나에 들어 있다. 새 탭/모드도 같은 파일에 추가.
- API 키를 코드·커밋에 넣지 않는다. 접속 코드가 `index.html`에 하드코딩되어 있고 저장소가 공개 상태라는 점 유의.
- 다른 사람 영상의 자막은 YouTube Data API로 못 받는다. 영상 분석은 Gemini의 YouTube URL 입력으로 한다.
