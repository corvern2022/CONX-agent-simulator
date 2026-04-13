# Foundation Agent Simulator

이 프로젝트는 정적 HTML/CSS/JS 웹앱으로 구성되어 있어, 서버 없이도 웹 호스팅 서비스에서 바로 배포할 수 있습니다.

## 라이브 링크 만들기

1. 이 저장소를 GitHub에 푸시하세요.
2. `main` 또는 `master` 브랜치에 푸시하면 GitHub Actions가 자동으로 배포를 시작합니다.
3. GitHub Pages 설정에서 `GitHub Actions` 빌드 결과를 소스로 선택하면 라이브 사이트가 활성화됩니다.

## 다른 배포 옵션

- Netlify / Vercel: 이 저장소를 연결하고 빌드 설정 없이 `index.html`을 그대로 배포할 수 있습니다.

## 준비 사항

- 현재 `index.html`은 단일 HTML 파일로 구성되어 있어 추가 빌드 도구가 필요 없습니다.
- 로컬 서버용 파일(`server.js`, `package.json`)은 제거되었습니다.
