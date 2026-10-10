# 수행평가 공지 사이트

GitHub에 코드를 보관하고 Render에서 실행하는 웹 프로젝트입니다. **GitHub Pages만으로는 이 프로젝트를 운영할 수 없습니다.** 이 사이트는 로그인 검증, 서버 API, 세션, PostgreSQL이 필요하므로 서버 호스팅이 필요합니다.

## 기능

- 학생: 학년·반 선택, 학급별 일정/달력, D-Day, 검색 및 과목 필터
- 교사: Google 로그인, 관리자 승인 후 수행평가 등록·수정·삭제
- 서버: Google ID 토큰 검증, PostgreSQL 저장, 관리자 계정 초기화

> 현재 버전은 MVP입니다. 실제 파일 업로드/다운로드 기능은 아직 구현되지 않았습니다. 학생들이 실제로 사용하기 전에 권한·보안·학급별 접근을 충분히 테스트하세요.

## 가장 쉬운 배포 방법: GitHub + Render + PostgreSQL

### 1. 프로젝트를 GitHub에 올리기

1. 아래 ZIP 파일을 내려받아 압축을 풉니다.
2. GitHub에서 **New repository**를 눌러 새 저장소를 만듭니다. 예: `assessment-notice-site`.
3. 저장소 화면에서 **Add file → Upload files**를 선택합니다.
4. 압축을 푼 `assessment-notice-site` 폴더 안의 파일과 `public`, `src` 폴더를 업로드합니다. `README.md`, `package.json`, `tsconfig.json`이 저장소 최상위에 보여야 합니다.
5. **Commit changes**를 눌러 저장합니다.

### 2. PostgreSQL 데이터베이스 만들기

1. [Neon](https://neon.tech/)에 가입하고 새 PostgreSQL 프로젝트를 만듭니다.
2. 대시보드에서 연결 문자열(Connection string)을 복사합니다. 보통 `postgresql://...` 형태입니다.
3. 이 문자열은 비밀 정보이므로 GitHub 코드에 넣지 마세요. 다음 단계의 Render 환경 변수에만 입력합니다.

### 3. Render에서 웹사이트 배포하기

1. [Render](https://render.com/)에 가입한 뒤 GitHub 계정을 연결합니다.
2. Render 대시보드에서 **New → Blueprint**를 선택합니다.
3. 방금 만든 GitHub 저장소를 선택하면 저장소의 `render.yaml` 설정을 읽습니다.
4. 환경 변수 입력 화면에서 아래 값을 설정합니다.
   - `DATABASE_URL`: Neon에서 복사한 PostgreSQL 연결 문자열
   - `GOOGLE_CLIENT_ID`: Google OAuth 웹 클라이언트 ID
   - `SESSION_SECRET`: 32자 이상의 무작위 비밀 문자열
   - `BOOTSTRAP_ADMIN_EMAIL`: 최초 관리자 권한을 받을 Google 계정 이메일
   - `COOKIE_SECURE`: `true`
5. 배포가 완료되면 Render가 제공하는 `https://...onrender.com` 주소를 엽니다.

### 4. Google 로그인 설정

1. [Google Cloud Console](https://console.cloud.google.com/)에서 프로젝트를 만들거나 선택합니다.
2. Google Auth Platform/OAuth 설정에서 동의 화면을 구성하고 **OAuth Client ID → Web application**을 만듭니다.
3. Render에서 받은 사이트 주소를 **Authorized JavaScript origins**에 추가합니다. 예: `https://your-site.onrender.com` (실제 주소로 바꾸세요).
4. 생성된 Client ID를 Render의 `GOOGLE_CLIENT_ID` 환경 변수에 입력합니다.
5. OAuth 동의 화면이 테스트 모드라면 로그인 가능한 테스트 사용자에 해당 Google 계정을 추가합니다.
6. Render에서 서비스를 다시 배포하거나 환경 변수를 저장한 후 재배포합니다.

### 5. 최초 관리자 계정

`BOOTSTRAP_ADMIN_EMAIL`에 지정한 이메일로 Google 로그인을 처음 하면 해당 계정이 최초 관리자로 등록되도록 설계되어 있습니다. 이메일을 정확히 입력하고 계정을 안전하게 관리하세요. 다른 교사는 가입 후 관리자의 승인을 받아야 합니다.

## 로컬 실행(선택 사항)

Node.js 20 이상과 PostgreSQL이 필요합니다.

```bash
npm install
cp .env.example .env
# .env에 DATABASE_URL, GOOGLE_CLIENT_ID, SESSION_SECRET, BOOTSTRAP_ADMIN_EMAIL 입력
npm run dev
```

브라우저에서 `http://localhost:3000`을 엽니다. Google OAuth의 로컬 테스트를 위해 `http://localhost:3000`을 Authorized JavaScript origins에 추가해야 할 수 있습니다.

## 필수 보안 주의

- `.env` 파일, DB 연결 문자열, 세션 비밀값을 GitHub에 올리지 마세요.
- 실제 운영 전 CSRF 보호, 로그인 속도 제한, 보안 점검, 백업, 접근성 및 모바일 테스트를 추가하세요.
- 첨부 파일 기능은 아직 작동하지 않습니다. 화면에 파일 안내가 있어도 실제 파일 저장소가 연결된 상태가 아닙니다.
