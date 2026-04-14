# ⚛️ 원소 카드 주기율표 웹앱

중학교 2학년 과학 수행평가 — **학급별 원소 카드 만들기** 프로젝트를 위한 웹앱입니다.  
학생들이 캔바에서 제작한 원소 카드를 업로드하면 주기율표 형태로 실시간 완성됩니다.

---

## 📁 파일 구성

```
/
├── index.html          # 주기율표 메인 페이지 (학생/교사 로그인 + 카드 조회)
├── upload.html         # 원소 카드 업로드 페이지 (학생용)
├── converter.html      # 교사용 통합 관리 도구
└── firebase-config.js  # Firebase 연결 설정 (직접 작성 필요)
```

> 학생 명단은 개인정보 보호를 위해 GitHub에 포함하지 않습니다.  
> 모든 학생 데이터는 Firebase Firestore에 저장됩니다.

---

## 🚀 배포 URL

| 대상 | URL |
|---|---|
| 주기율표 — 학생 (반별) | `https://[아이디].github.io/[레포]/?class=2-1` |
| 주기율표 — 교사 (전체) | `https://[아이디].github.io/[레포]/` (파라미터 없이) |
| 업로드 페이지 | 주기율표 로그인 후 헤더의 [내 카드 업로드] 버튼 |
| 교사 관리 도구 | `https://[아이디].github.io/[레포]/converter.html` |

반 번호는 `?class=2-1` ~ `?class=2-10` 형태로 변경합니다.

---

## ⚙️ 초기 설정

### 1. Firebase 프로젝트 준비

1. [Firebase Console](https://console.firebase.google.com)에서 프로젝트 생성
2. Firestore Database 활성화 (위치: `asia-northeast3`)
3. Storage 활성화 (위치: `asia-northeast3`)
4. Authentication 활성화 → 이메일/비밀번호 로그인 사용 설정
5. Authentication → 사용자 탭 → 교사 계정 추가
6. 웹 앱 등록 후 `firebaseConfig` 복사

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

    // 학생 카드 업로드 경로
    match /classes/{cls}/{studentId}/{fileName} {
      allow read: if true;
      allow write: if request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*')
        && fileName.matches('[0-9]+_(front|back)');
    }

    // 교사 직접 업로드 경로
    match /teacher/{fileName} {
      allow read: if true;
      allow write: if request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*');
    }

  }
}
```

### 5. Storage CORS 설정 (ZIP 다운로드용)

Google Cloud Shell에서 한 번만 실행:

```bash
echo '[{"origin": ["*"], "method": ["GET"], "maxAgeSeconds": 3600}]' > cors.json
gsutil cors set cors.json gs://[storageBucket_주소]
```

### 6. GitHub Pages 활성화

레포 → Settings → Pages → Branch: `main` / Folder: `/(root)` → Save

---

## 📋 사용 방법

### 교사

#### 원소 배정 — `converter.html`

총 4개 탭으로 구성됩니다.

**📊 원소 배정 탭**
1. 나이스 명렬표 엑셀 업로드 (열 순서: `학년 / 반 / 번호 / 이름 / 성별`)
2. 반별 [🎲 이 반 배정] 또는 [🎲 전체 반 원소 배정] 클릭
   - 학생 수 × 2 범위 안에서 중복 없이 랜덤 배정
   - 작은 번호 → 원소1, 큰 번호 → 원소2
3. Firebase 자동 저장 + 저장 시간 표시
4. 엑셀 없이 재접속해도 Firebase에서 자동 복원

**🔍 학생 조회 탭**
- 반 선택 → 이름 선택 → 담당 원소 확인
- 반별 전체 목록 조회
- 학생이 태블릿으로 접속해서 본인 원소 확인 가능

**🖼️ 카드 다운로드 탭**
- 반 선택 → [📦 ZIP 다운로드]
- 파일명 형식: `2-1반_01번_김민준_앞면.png`
- 업로드된 카드만 포함, 미업로드 학생 자동 건너뜀

**👩‍🏫 교사 업로드 탭**
- 배정 범위 밖 원소(예시 카드 등)를 교사가 직접 업로드
- 원소 번호 입력 → 반 선택 (전체 선택 가능) → 앞면·뒷면 업로드
- 선택한 모든 반에 동시 반영

#### 주기율표 조회 — `index.html` (파라미터 없이)

1. `https://[아이디].github.io/[레포]/` 접속
2. [👩‍🏫 교사 로그인] 버튼 → 이메일 + 비밀번호 입력
3. 로그인 후 헤더의 반 선택 드롭다운으로 모든 반 전환 가능
4. 기본으로 1반 주기율표가 표시됨

---

### 학생

#### 원소 확인
`converter.html` → 학생 조회 탭 → 반 + 이름 선택

#### 카드 업로드
1. 담임선생님이 공유한 `?class=2-X` URL로 접속
2. 학번 + 이름으로 로그인
3. 헤더의 [내 카드 업로드] 클릭
4. 캔바에서 PNG로 내보낸 앞면·뒷면 각각 업로드 (5MB 이하)
5. 주기율표에 실시간 반영 확인

---

## 📊 완성도 계산 기준

완성도는 **학생 배정 범위 안의 원소만** 포함합니다.

```
학생 27명 × 2 = 배정 범위 1~54번

완성도 = 54번 이하 원소 중 앞면이 업로드된 카드 수 / 54
```

교사가 직접 업로드한 예시 카드(예: 우라늄 92번)는 완성도 계산에서 제외됩니다.

---

## 🗄️ Firebase 데이터 구조

```
Firestore
├── converter/
│   └── assignments              # 원소 배정 정보 + 학생 명단
│       ├── assignments: { 학번: { name, cls, clsKey, elem1, elem2 ... } }
│       ├── classList: ["2-1", "2-2", ...]
│       ├── savedAt: "2026-04-..."
│       └── totalStudents: 270
│
├── students/
│   └── 20101 → { name, cls, elements: [1, 2] }   # 로그인 검증용
│
└── classes/
    └── 2-1/
        └── elements/
            ├── 1  → { studentId, studentName, cls, elementNo, frontUrl, backUrl, updatedAt }
            └── 92 → { studentId: "teacher", ... }  # 교사 업로드

Storage
├── classes/
│   └── 2-1/
│       └── [학번]/
│           ├── 1_front    # 앞면 이미지
│           └── 1_back     # 뒷면 이미지
│
└── teacher/
    ├── 92_front            # 교사 직접 업로드 앞면
    └── 92_back             # 교사 직접 업로드 뒷면
```

---

## 🔒 보안

| 항목 | 방식 |
|---|---|
| 학생 명단 | GitHub 미포함, Firebase에만 저장 |
| 학생 로그인 | 학번+이름 → Firestore 검증, URL 학급과 일치해야 허용 |
| 교사 로그인 | Firebase Authentication (이메일+비밀번호) |
| 반 접근 제한 | `?class=` 파라미터 없으면 학생 접근 차단, 교사만 가능 |
| 이미지 업로드 | 5MB 이하 이미지 파일만 허용 |
| 교사 업로드 경로 | `teacher/` 경로 별도 규칙으로 분리 |

---

## 🛠️ 기술 스택

| 항목 | 내용 |
|---|---|
| 프론트엔드 | HTML / CSS / Vanilla JS |
| 백엔드 | Firebase Firestore + Storage + Authentication |
| 배포 | GitHub Pages |
| 라이브러리 | SheetJS (엑셀 파싱), JSZip (ZIP 생성) |
| 폰트 | Noto Sans KR, Space Mono |

---

## 📌 학번 형식

```
학년(1자리) + 반(2자리) + 번호(2자리)

예) 2학년 1반 1번   → 20101
    2학년 10반 26번 → 21026
```

---

## ❓ 문제 해결

| 증상 | 원인 | 해결 |
|---|---|---|
| 학생 로그인 불가 | converter에서 배정·저장 미완료 | converter → 배정 → 저장 확인 |
| 교사 로그인 불가 | Firebase Auth 계정 미생성 | Authentication → 사용자 추가 |
| 반 목록 안 보임 | Firebase 데이터 미로드 | 페이지 새로고침 |
| 카드 업로드 오류 | Storage 규칙 문제 | Storage 보안 규칙 확인 |
| ZIP 다운로드 CORS 오류 | Storage CORS 미설정 | gsutil cors set 명령 실행 |
| 교사 업로드 권한 오류 | teacher/ 경로 규칙 없음 | Storage 규칙에 teacher 경로 추가 |
| 완성도가 100% 초과 | 예시 카드가 포함됨 | updateProgress 함수 maxElemNo 조건 확인 |
| 주기율표 카드 미반영 | Firestore 연결 끊김 | 페이지 새로고침 |
