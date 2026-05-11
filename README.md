# 계란 이마 챌린지 - 공용 랭킹 버전

업로드된 리소스를 사용해 만든 정적 웹 게임입니다.  
게임 화면은 `index.html` 하나로 실행되며, 랭킹은 Firebase Firestore를 연결하면 여러 사용자가 같은 TOP 10을 공유합니다.

## 폴더 구조

```text
egg-forehead-game/
├─ index.html
├─ firebase-config.js
├─ firestore.rules
├─ README.md
├─ .nojekyll
└─ assets/
   ├─ 기본자세.png
   ├─ 계란 맞는 모습.gif
   ├─ 계란 안깐거 - 아래부분.png
   ├─ 계란 안깐거 - 윗부분.png
   └─ 깐계란.gif
```

## 동작 방식

- 게임 시작 후 30초 타이머가 시작됩니다.
- 캐릭터를 클릭하면 통계란이 이마로 날아갑니다.
- 이마에 맞으면 `계란 맞는 모습.gif`가 재생됩니다.
- 계란 껍질 상단/하단 PNG와 `깐계란.gif`가 랜덤한 방향으로 튕겨 떨어집니다.
- 종료 후 이름을 입력하면 Firestore의 공용 랭킹에 기록됩니다.
- Firebase 설정 전에는 자동으로 브라우저 로컬 테스트 랭킹으로 동작합니다.

## 1. 로컬 테스트

1. 압축을 풉니다.
2. `index.html`을 브라우저에서 엽니다.
3. 캐릭터를 클릭해 테스트합니다.

Firebase 설정 전에는 `localStorage` 기반 테스트 랭킹이 표시됩니다.  
여러 기기에서 같은 랭킹을 보려면 아래 Firebase 설정을 진행해야 합니다.

## 2. Firebase 공용 랭킹 설정

### 2-1. Firebase 프로젝트 만들기

1. Firebase Console에 접속합니다.
2. 새 프로젝트를 만듭니다.
3. 프로젝트에서 웹 앱을 추가합니다.
4. Firebase SDK 설정 중 `firebaseConfig` 객체를 복사합니다.

### 2-2. firebase-config.js 수정

`firebase-config.js` 파일을 열고 아래 값을 Firebase Console에서 복사한 값으로 바꿉니다.

```js
window.EGG_GAME_FIREBASE_CONFIG = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.firebasestorage.app",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

`apiKey`, `projectId`가 `YOUR_` 상태로 남아 있으면 공용 랭킹이 아니라 로컬 테스트 랭킹으로 동작합니다.

### 2-3. Firestore Database 만들기

1. Firebase Console → Firestore Database로 이동합니다.
2. 데이터베이스를 생성합니다.
3. 위치는 사용자와 가까운 지역을 선택합니다.
4. 보안 규칙은 아래 `firestore.rules` 내용을 복사해 게시합니다.

### 2-4. Firestore 보안 규칙

이 패키지의 `firestore.rules` 파일 내용을 Firebase Console → Firestore Database → Rules에 붙여 넣고 게시합니다.

```txt
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /leaderboards/egg-forehead-game-v1/scores/{scoreId} {
      allow read: if true;

      allow create: if
        request.resource.data.keys().hasOnly(['name', 'score', 'durationSec', 'createdAt']) &&
        request.resource.data.name is string &&
        request.resource.data.name.size() > 0 &&
        request.resource.data.name.size() <= 12 &&
        request.resource.data.score is int &&
        request.resource.data.score >= 0 &&
        request.resource.data.score <= 300 &&
        request.resource.data.durationSec == 30 &&
        request.resource.data.createdAt == request.time;

      allow update, delete: if false;
    }
  }
}
```

## 3. 무료 배포 추천

가장 단순한 무료 배포는 GitHub Pages입니다.

1. GitHub에 새 저장소를 만듭니다.
2. 이 폴더의 파일을 그대로 업로드합니다.
3. 저장소의 **Settings → Pages**로 이동합니다.
4. Source를 **Deploy from a branch**, Branch를 `main`, Folder를 `/root`로 선택합니다.
5. 생성된 `github.io` 주소로 접속합니다.

## 4. 랭킹 데이터 구조

Firestore에는 아래 경로로 점수가 저장됩니다.

```text
leaderboards / egg-forehead-game-v1 / scores / 자동문서ID
```

문서 예시:

```json
{
  "name": "홍길동",
  "score": 37,
  "durationSec": 30,
  "createdAt": "server timestamp"
}
```

게임은 `score` 내림차순으로 최대 30개를 가져온 뒤, 화면에는 TOP 10만 보여줍니다.

## 5. 주의사항

현재 구조는 무료 정적 웹 게임에 적합한 간단한 공용 랭킹입니다.  
점수는 클라이언트에서 전송되므로 개발자 도구를 악용한 점수 조작을 완벽히 막을 수는 없습니다.

랭킹 조작 방지가 중요해지면 다음 단계가 필요합니다.

- Firebase Authentication 익명 로그인 추가
- Cloud Functions 또는 별도 서버에서 점수 검증
- 1인 1기록 제한
- 비정상 점수 자동 필터링
