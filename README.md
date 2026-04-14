# ⚛️ 원소 카드 주기율표 웹앱

중학교 2학년 과학 수행평가 — **학급별 원소 카드 만들기** 프로젝트를 위한 웹앱입니다.  
학생들이 캔바에서 제작한 원소 카드를 업로드하면 주기율표 형태로 실시간 완성됩니다.

---

## 📁 파일 구성

```
/
├── index.html          # 주기율표 메인 페이지 (학생 로그인 + 카드 조회)
├── upload.html         # 원소 카드 업로드 페이지
├── converter.html      # 교사용 관리 도구 (원소 배정 + 학생 조회 + ZIP 다운로드)
└── firebase-config.js  # Firebase 연결 설정 (직접 작성 필요)
```

> `students.json`은 개인정보 보호를 위해 GitHub에 포함하지 않습니다.  
> 학생 명단은 Firebase Firestore에 저장됩니다.

---

## 🚀 배포 URL

| 대상 | URL |
|---|---|
| 주기율표 (반별) | `https://[아이디].github.io/period_table_assignment/?class=2-1` |
| 업로드 페이지 | 로그인 후 헤더의 [내 카드 업로드] 버튼 |
| 교사 관리 도구 | `https://[아이디].github.io/period_table_assignment/converter.html` |

반 번호는 `?class=2-1` ~ `?class=2-10` 형태로 변경합니다.

---

## ⚙️ 초기 설정

### 1. Firebase 프로젝트 준비

1. [Firebase Console](https://console.firebase.google.com)에서 프로젝트 생성
2. Firestore Database 활성화 (위치: `asia-northeast3`)
3. Storage 활성화 (위치: `asia-northeast3`)
4. 웹 앱 등록 후 `firebaseConfig` 복사

### 2. `firebase-config.js` 작성

```javascript
const firebaseConfig = {
  apiKey: "여기에_붙여넣기",
  authDomain: "여기에_붙여넣기",
  projectId: "여기에_붙여넣기",
  storageBucket: "여기에_붙여넣기",
  messagingSenderId: "여기에_붙여넣기",
  appId: "여기에_붙여넣기"
};

export default firebaseConfig;
```

### 3. Firestore 보안 규칙

Firebase Console → Firestore → 규칙 탭에서 설정:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /classes/{cls}/elements/{elementNo} {
      allow read: if true;
      allow write: if request.resource.data.keys()
        .hasAll(['studentId','studentName','cls','elementNo','frontUrl','backUrl','updatedAt'])
        && request.resource.data.cls == cls
        && request.resource.data.elementNo == int(elementNo);
    }
    match /students/{studentId} {
      allow read: if true;
      allow write: if true;
    }
    match /converter/{docId} {
      allow read: if true;
      allow write: if true;
    }
  }
}
```

### 4. Storage 보안 규칙

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /classes/{cls}/{studentId}/{fileName} {
      allow read: if true;
      allow write: if request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*')
        && fileName.matches('[0-9]+_(front|back)');
    }
  }
}
```

### 5. Storage CORS 설정 (ZIP 다운로드용)

Google Cloud Shell에서 실행:

```bash
echo '[{"origin": ["*"], "method": ["GET"], "maxAgeSeconds": 3600}]' > cors.json
gsutil cors set cors.json gs://[storageBucket_주소]
```

### 6. GitHub Pages 활성화

레포 → Settings → Pages → Branch: `main` / Folder: `/(root)` → Save

---

## 📋 사용 방법

### 교사

#### 원소 배정
1. `converter.html` 접속
2. **원소 배정 탭** → 나이스 명렬표 엑셀 업로드
   - 엑셀 열 순서: `학년 / 반 / 번호 / 이름 / 성별`
3. 반별 [🎲 이 반 배정] 또는 [🎲 전체 반 원소 배정] 클릭
4. Firebase 자동 저장 + 저장 시간 표시
5. **JSON 다운로드 탭** → `students.json` 다운로드 (참고용)

#### 카드 수거
1. **카드 다운로드 탭** → 반 선택 → [📦 ZIP 다운로드]
2. 파일명 형식: `2-1반_01번_김민준_앞면.png`

### 학생

#### 원소 확인
- `converter.html` → **학생 조회 탭** → 반 + 이름 선택

#### 카드 업로드
1. `?class=2-1` URL로 접속 → 학번 + 이름으로 로그인
2. 헤더의 [내 카드 업로드] 클릭
3. 캔바에서 PNG로 내보낸 앞면·뒷면 각각 업로드
4. 주기율표에 실시간 반영 확인

---

## 🗄️ Firebase 데이터 구조

```
Firestore
├── converter/
│   └── assignments         # 원소 배정 정보 + 학생 명단
│       ├── assignments: { 학번: { name, cls, elem1, elem2, ... } }
│       ├── classList: ["2-1", "2-2", ...]
│       └── savedAt: "2026-04-..."
│
└── classes/
    └── 2-1/
        └── elements/
            ├── 1           # 원소번호 1번 카드
            │   ├── studentId, studentName, cls
            │   ├── elementNo, frontUrl, backUrl
            │   └── updatedAt
            └── 2 ...

Storage
└── classes/
    └── 2-1/
        └── [학번]/
            ├── 1_front     # 앞면 이미지
            └── 1_back      # 뒷면 이미지
```

---

## 🔒 보안

- 학생 명단은 GitHub에 포함하지 않고 Firebase에만 저장합니다
- 주기율표 URL에 `?class=` 파라미터가 없으면 접근이 차단됩니다
- 학번의 학급 정보와 URL의 학급이 일치해야만 로그인이 허용됩니다
- Firebase Storage는 이미지 파일만, 5MB 이하만 업로드 가능합니다

---

## 🛠️ 기술 스택

| 항목 | 내용 |
|---|---|
| 프론트엔드 | HTML / CSS / Vanilla JS |
| 백엔드 | Firebase Firestore + Storage |
| 배포 | GitHub Pages |
| 라이브러리 | SheetJS (엑셀 파싱), JSZip (ZIP 생성) |
| 디자인 | 다크 테마, Noto Sans KR + Space Mono |

---

## 📌 학번 형식

```
학년(1자리) + 반(2자리) + 번호(2자리)

예) 2학년 1반 1번  → 20101
    2학년 10반 26번 → 21026
```

---

## ❓ 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| 로그인 불가 | converter에서 배정 저장 안 됨 | converter → 배정 → 저장 확인 |
| 카드 업로드 오류 | Firebase Storage 규칙 문제 | Storage 보안 규칙 확인 |
| ZIP 다운로드 CORS 오류 | Storage CORS 미설정 | gsutil cors set 명령 실행 |
| 반 목록 안 보임 | Firebase 데이터 미로드 | 페이지 새로고침 후 재시도 |
| 주기율표 카드 미반영 | Firestore 실시간 연결 끊김 | 페이지 새로고침 |
