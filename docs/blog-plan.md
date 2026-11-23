# Dev Blog 구축 계획과 스펙

## 목표

개발하면서 배운 내용을 Markdown으로 기록하고, GitHub에서 글을 검토한 뒤 `main`에 머지하면 자동으로 발행한다. 첫 공개 배포를 빠르게 끝내고, 이후 주제별 탐색과 댓글을 필요에 따라 확장한다.

## 기술 선택 과정

### Astro를 고른 이유

처음에는 Astro, Next.js, Hugo, Jekyll을 비교했다. 블로그를 빠르게 만들고 배포하는 게 이번 목표였고, 핵심 기능은 Markdown 글을 관리하고 발행하는 것이었다.

| 후보 | 대화에서 비교한 내용 |
| --- | --- |
| Astro | Markdown/MDX와 콘텐츠 컬렉션으로 글과 메타데이터를 관리하고 정적 HTML로 발행할 수 있다. 필요한 부분에 React 컴포넌트도 사용할 수 있다. |
| Next.js | 이미 익숙한 React를 사용하고, 로그인이나 대시보드 등으로 확장하기 좋다. 글 발행이 중심인 첫 버전에서는 범용 앱 기능을 위한 설정이 추가될 수 있다. |
| Hugo | 정적 빌드가 빠르고 블로그 테마가 다양하다. |
| Jekyll | GitHub Pages와 기본적으로 통합되어 GitHub 기반 정적 블로그를 시작하기 쉽다. |

Astro를 선택한 직접적인 이유는 **콘텐츠 관리부터 정적 발행까지 블로그에 필요한 기능이 갖춰져 있어 첫 공개까지 구현할 양을 줄일 수 있기 때문**이다. 빌드 시 만든 HTML에 본문이 들어 있어 검색 엔진이 읽기 쉽다는 점도 고려했다. Hugo와 Jekyll은 후보로 비교했으며, 각각을 제외할 별도 조건은 정하지 않았다.

#### 참고 실험 · AI 개발 토큰 비용

Nolan Lawson은 Claude Code에 같은 포켓몬 도감 명세, 요청, 테스트를 주고 12개 프레임워크 조합으로 각각 5회 구현을 시도했다. 아래는 블로그 운영비와 별도로, 에이전트가 앱을 구현하고 검증하는 데 사용한 토큰을 API 단가로 환산한 1회 비용과 소요 시간의 중앙값이다.

| 조합 | 환산 비용 | 토큰 | 소요 시간 |
| --- | ---: | ---: | ---: |
| Hono + htmx | $2.67 | 426만 | 8.6분 |
| Astro | $2.76 | 413만 | 8.2분 |
| Next.js | $3.14 | 499만 | 10.9분 |
| React Router | $3.81 | 758만 | 12.5분 |

Astro의 비용 중앙값은 Next.js보다 약 12% 낮았다. 생성된 소스도 Astro 1,461줄, Next.js 1,539줄로 Astro 쪽이 짧았다. 이처럼 페이지 중심 앱에서 구현할 코드가 적으면 AI가 읽고 쓰는 양도 줄어들 수 있다. 저자는 낯선 프레임워크의 문서와 소스를 추가로 읽는 작업도 비용을 높일 수 있다고 관찰했다. 실험은 이런 요인별 효과를 따로 측정하지 않았고, 두 프레임워크의 실행별 비용 범위도 겹쳤다. 따라서 이 수치는 단일 앱에서 관찰한 결과로 참고한다.

- [Nolan Lawson의 실험 원문](https://nolanlawson.com/2026/10/28/the-ai-cost-of-javascript-frameworks/)
- [실험을 소개한 유튜브 쇼츠](https://youtube.com/shorts/IWj5tabIqp4?si=alLrPSUf7kSNfc7C)

### Cloudflare Pages를 고른 이유

GitHub Pages와 Vercel도 후보였다. GitHub Pages는 GitHub와 함께 정적 사이트를 호스팅하기 쉽고, Vercel도 Git 연동과 PR 미리보기를 지원한다. 이번에는 **Astro의 정적 빌드 결과를 배포하고, PR에서 바뀐 글과 UI를 미리 확인하고, `main` 머지 후 자동 발행하는 흐름**을 Cloudflare Pages의 Git 연동으로 구성하기로 했다. 이 흐름에는 별도의 GitHub Actions 배포 워크플로가 필요 없다.

기본 디자인을 확인한 뒤 전환을 결정했다. Fuwari는 사이드바와 글 목록, Pagefind 검색과 단일 카테고리를 기본 제공한다. 화면은 Astro Theme Pure를 참고해 색상, 타이포그래피, 여백을 조정했다. 상위·하위 카테고리는 이후에 추가한다.

## 구성과 세부 스펙


| 항목      | 선택                           | 이유                                                                                                                       |
| ------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| 프레임워크   | Astro                        | 글을 빌드 시 정적 HTML로 생성한다. 검색 엔진이 본문을 읽기 쉽고, 검색처럼 상호작용이 필요한 곳에 JavaScript를 사용할 수 있다.                                         |
| 블로그 템플릿 | Fuwari와 Pure 스타일 참고         | Fuwari의 글 목록, 사이드바, 검색과 단일 카테고리를 쓰고 Pure의 색상과 여백을 참고한다.                                    |
| 글 관리    | Markdown/MDX와 GitHub         | 글 파일과 변경 이력을 저장소에 보관하고 PR에서 글과 UI 변경을 함께 검토할 수 있다. Astro Content Collections가 제목, 날짜 같은 글 메타데이터를 검증한다.                   |
| 검색      | Pagefind                     | Fuwari에 포함되어 있고, 빌드 결과에서 검색 인덱스를 만든다. 제목과 본문을 검색하는 데 별도 검색 서버가 필요 없다.                                                |
| 배포      | Cloudflare Pages의 GitHub 연동  | 정적 빌드 결과를 호스팅하면서 PR 미리보기와 `main` 배포를 같은 서비스에서 처리할 수 있다. 머지 전에 실제로 렌더링된 글과 화면을 확인할 수 있는 구성을 선택했다.                         |
| 카테고리    | Markdown 메타데이터 기반의 최대 2단계 분류 | 네이버 블로그처럼 상위 주제에서 하위 주제로 탐색할 수 있다. 파일 경로와 분류를 분리하면 카테고리를 개편할 때 글 파일을 옮길 필요가 줄어든다. 태그는 글에 걸쳐 공통 키워드를 붙이는 데 사용한다.          |
| 댓글 후보   | giscus                       | 개발 블로그 독자가 GitHub 계정을 사용한다면 붙이기 쉽다. 댓글을 공개 저장소의 GitHub Discussions에 저장하므로 별도 댓글 서버와 DB가 필요 없다. 댓글 작성에는 GitHub 로그인이 필요하다. |

## 진행 순서와 완료 기준

1. **첫 배포 준비** · 공개할 글과 남길 예시 글을 정하고 GitHub 저장소를 연결한다.
2. **프로젝트 문서와 코드 컨벤션 정리** · 코드 컨벤션 파일, `AGENTS.md`, `CLAUDE.md`를 만들고 이미 있는 `README.md`를 보강한다. 저장소에 네 문서가 있고, 실행·검증 방법과 글·코드 작성 규칙을 문서만 보고 따라 할 수 있는지 확인한다.
3. **Cloudflare Pages 연결** · 프로덕션 브랜치를 `main`, 빌드 명령을 `pnpm build`, 출력 경로를 `dist`로 설정한다. 공개 URL에서 홈, 글 상세, 제목·본문 검색을 확인한다.
4. **PR 미리보기와 자동 발행 확인** · Markdown 글을 추가한 PR의 미리보기 URL에서 글을 확인한다. `main` 머지 후 같은 글이 프로덕션에서 열리고 검색되는지 확인한다.
5. **계층형 카테고리 추가** · 상위·하위 카테고리를 탐색할 메뉴와 카테고리별 글 목록을 구현한다. 상위 주제에서 하위 주제로 들어가 해당 글을 열 수 있는지 확인한다.
6. **필요 시 댓글 추가** · GitHub 로그인 조건이 독자에게 적합한지 확인하고 giscus를 연결한다. 글별 댓글 작성과 Discussions 저장을 확인한다.

## 참고

- [구축 방향을 정한 ChatGPT 대화](https://chatgpt.com/share/6afd6c03-4e38-83e8-9f16-8dda49c05f06)
- [로컬 실행 및 배포 설정](../README.md)
- [AstroPaper v6](https://github.com/satnaing/astro-paper/tree/v6.0.0) · [Cloudflare Pages Git 연동](https://developers.cloudflare.com/pages/configuration/git-integration/) · [giscus](https://giscus.app/ko)

- [Fuwari](https://github.com/saicaca/fuwari) · [Astro Theme Pure](https://github.com/cworld1/astro-theme-pure)
