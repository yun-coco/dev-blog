# Dev Blog

Fuwari 기반 개발 블로그입니다. 화면 스타일은 Astro Theme Pure의 색상과 여백을 참고했습니다.

## 로컬 실행

Node.js 22.12 이상과 Corepack 또는 pnpm 9.14.4가 필요합니다.

```sh
pnpm install --frozen-lockfile
pnpm dev
```

기본 주소는 http://localhost:4321 입니다. Pagefind 검색은 빌드한 뒤 미리보기에서 확인할 수 있습니다.

```sh
pnpm build
pnpm preview
```

글은 `src/content/posts/`에 Markdown 또는 MDX로 추가합니다. `hello-blog.md`는 발행과 검색을 확인하는 예시입니다.

## Cloudflare Pages 연결

GitHub 저장소를 연결한 뒤 프로덕션 브랜치는 `main`, 빌드 명령은 `pnpm build`, 출력 디렉터리는 `dist`로 설정합니다. Node.js 22.12 이상을 사용하세요. PR 미리보기와 main 자동 발행은 연결 후 실제 배포에서 확인합니다.

정식 사이트 주소를 고정하려면 Cloudflare Pages 환경 변수에 `SITE_URL`을 설정하세요. 기본값은 Cloudflare가 제공하는 `CF_PAGES_URL`입니다.

[Fuwari 원본](https://github.com/saicaca/fuwari) · [스타일 참고 Astro Theme Pure](https://github.com/cworld1/astro-theme-pure)
