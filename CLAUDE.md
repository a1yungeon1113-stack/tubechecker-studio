# tubechecker-studio 작업 메모

사용자와는 한국어로, 쉬운 말로 짧게 이야기한다. 화면 문구도 한국어 존댓말(~해요)로 쓴다.

## 사이트
- 빌드 도구·패키지 없는 정적 사이트. 페이지마다 HTML 파일 하나에 CSS·JS가 다 들어 있다.
- Netlify(프로젝트 `tubechecker1-studio`)로 배포: main에 합치면 자동 배포, PR마다 `deploy-preview-<번호>--tubechecker1-studio.netlify.app` 미리보기가 생긴다. 기본 주소는 https://tubechecker1-studio.netlify.app
- `index.html` — 유튜브 대본 검수기. localStorage: `fss-k`(API 키), `fss-h`(히스토리), `fss-access`(접속 코드 확인).
- `storage.html` — 이미지·영상 저장창고. 접속 확인은 `fss-access`를 같이 쓰고, 없으면 `index.html?next=storage`로 보내 로그인 뒤 돌아오게 한다.
- 두 페이지는 같은 다크 테마 색 변수(`--bg`, `--c1`, `--ac` 등)와 Pretendard 폰트를 쓴다.

## 저장창고 (storage.html)
- 서버 없음. 파일은 브라우저 IndexedDB `fss-media`에 저장: `items`(정보+썸네일), `files`(원본), `meta`(`cats`, `lastBackup`). 화면 설정은 localStorage `fss-media-ui`.
- 그래서 기기·브라우저·주소마다 따로 저장되고, 사이트 데이터를 지우면 사라진다. PC↔휴대폰 동기화는 없다. 원하면 클라우드 저장소(Firebase 등, 가입 필요) 연결을 제안해 둔 상태.
- 기본 카테고리: 동물, 물고기, 공룡, 식물(꽃·나무·풀·잎), 계절(봄·여름·가을·겨울). 카테고리는 2단계까지이고, 파일 하나가 여러 카테고리에 들어갈 수 있다.
- 백업: 카테고리별 폴더와 `저장창고-백업정보.json`이 든 zip. zip은 외부 라이브러리 없이 직접 만들고(압축 없음, 2GB마다 나눔) 읽는다(deflate, 윈도우 한글 이름 포함). 빈 저장창고에 복원하면 백업의 카테고리 이름·순서를 그대로 따른다.

## 사용자에게 안내한 것
- 바탕화면 아이콘은 자동으로 생기지 않는다. PC는 저장창고를 연 뒤 주소창 맨 왼쪽 아이콘을 바탕화면으로 끌어다 놓기, 아이폰은 사파리 공유 → 홈 화면에 추가, 안드로이드는 크롬 ⋮ → 홈 화면에 추가.
- 올린 사진·영상은 바탕화면에 파일로 생기지 않는다. 실제 파일이 필요하면 백업 zip을 풀면 카테고리별 폴더로 나온다.
- 저장창고는 늘 같은 주소로만 쓴다. PR 미리보기 주소에 올린 파일은 본 사이트에 안 보인다.

## 작업할 때
- 저장소에 테스트 코드는 없다. 확인은 `python3 -m http.server`로 띄우고 전역 Playwright(`/opt/node22/lib/node_modules/playwright`)로 Chromium을 돌려서 한다.
- 한글 다운로드 파일 이름을 확인하려면 브라우저를 `LANG=C.UTF-8`로 띄운다. 기본 로케일이면 이름이 `download`로 바뀐다(사이트 문제 아님).
- 클라우드 작업 환경의 네트워크 정책이 netlify.app 접속을 막아서, 배포된 사이트는 여기서 직접 열어 볼 수 없다. 환경 설정의 허용 도메인에 추가하면 된다.
