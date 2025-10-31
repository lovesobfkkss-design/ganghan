# Firebase Security Rules 설정 가이드

## ⚠️ 중요: 즉시 적용 필요

현재 데이터베이스는 **인증 없이 누구나 접근 가능**한 상태입니다.
아래 Security Rules를 Firebase Console에 즉시 적용해주세요.

---

## 🔐 Firebase Security Rules 적용 방법

### 1단계: Firebase Console 접속
1. https://console.firebase.google.com/ 접속
2. **ganghanlucy** 프로젝트 선택

### 2단계: Realtime Database 규칙 설정
1. 좌측 메뉴에서 **"Realtime Database"** 클릭
2. 상단 탭에서 **"규칙(Rules)"** 클릭
3. 아래 규칙을 복사하여 붙여넣기
4. **"게시(Publish)"** 버튼 클릭

---

## 📋 적용할 Security Rules

### 기본 규칙 (로그인한 사용자만 접근)

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null",

    "students": {
      ".read": "auth != null",
      ".write": "auth != null",
      ".indexOn": ["name", "grade"]
    },

    "daily": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "korean": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "hsMgmt": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "typing": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "tutoringSessions": {
      ".read": "auth != null",
      ".write": "auth != null",
      ".indexOn": ["date", "subject", "name"]
    },

    "retest": {
      ".read": "auth != null",
      ".write": "auth != null",
      ".indexOn": ["date"]
    },

    "classcard": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "vocabAccum": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "vocab": {
      ".read": "auth != null",
      ".write": "auth != null"
    },

    "settings": {
      ".read": "auth != null",
      ".write": "auth != null"
    }
  }
}
```

---

## 👥 Firebase Authentication 설정

### 1단계: Authentication 활성화
1. Firebase Console 좌측 메뉴에서 **"Authentication"** 클릭
2. **"시작하기"** 버튼 클릭 (처음 사용하는 경우)

### 2단계: 로그인 방법 설정

#### 옵션 1: Google 로그인 (권장)
1. **"Sign-in method"** 탭 클릭
2. **"Google"** 선택
3. **"사용 설정"** 토글 ON
4. 프로젝트 지원 이메일 선택
5. **"저장"** 클릭

#### 옵션 2: 이메일/비밀번호 로그인
1. **"Sign-in method"** 탭 클릭
2. **"이메일/비밀번호"** 선택
3. **"사용 설정"** 토글 ON
4. **"저장"** 클릭

### 3단계: 사용자 추가
1. **"Users"** 탭 클릭
2. **"사용자 추가"** 버튼 클릭
3. 이메일과 비밀번호 입력
4. **"사용자 추가"** 클릭

**주의:** Google 로그인을 사용하는 경우, 사용자가 처음 로그인하면 자동으로 계정이 생성됩니다.

---

## 🔒 고급 Security Rules (역할 기반 접근 제어)

특정 사용자만 데이터를 수정하고, 나머지는 읽기만 가능하게 하려면:

```json
{
  "rules": {
    "students": {
      ".read": "auth != null",
      ".write": "auth.token.email === 'admin@example.com' || auth.token.email === 'teacher@example.com'",
      ".indexOn": ["name", "grade"]
    },

    "settings": {
      ".read": "auth != null",
      ".write": "auth.token.email === 'admin@example.com'"
    }
  }
}
```

**admin@example.com**을 실제 관리자 이메일로 변경하세요.

---

## 🛡️ 추가 보안 조치

### 1. 입력 검증 규칙 추가

```json
{
  "rules": {
    "students": {
      ".read": "auth != null",
      ".write": "auth != null",
      "$studentId": {
        ".validate": "newData.hasChildren(['name', 'grade'])",
        "name": {
          ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 50"
        },
        "grade": {
          ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 10"
        },
        "studentPhone": {
          ".validate": "!newData.exists() || (newData.isString() && newData.val().length <= 20)"
        },
        "parentPhone": {
          ".validate": "!newData.exists() || (newData.isString() && newData.val().length <= 20)"
        },
        "$other": {
          ".validate": false
        }
      }
    }
  }
}
```

### 2. Firebase API 키 권한 제한

1. **Google Cloud Console** 접속: https://console.cloud.google.com/
2. **ganghanlucy** 프로젝트 선택
3. 좌측 메뉴에서 **"API 및 서비스"** → **"사용자 인증 정보"** 클릭
4. API 키 찾기 (AIzaSyCEPzrduTPO9_07SV3-2prXgWI-zQRsUA8)
5. **"API 키 제한"** 탭에서:
   - **"HTTP 리퍼러(웹사이트)"** 선택
   - 허용할 도메인 추가:
     - `https://iridescent-sunshine-6dfc58.netlify.app/*`
     - `http://localhost:*` (개발용)
6. **"저장"** 클릭

### 3. App Check 설정 (선택사항, 추가 보안)

Firebase App Check를 사용하면 승인된 앱에서만 Firebase에 접근할 수 있습니다.

1. Firebase Console → **"App Check"** 클릭
2. **"시작하기"** 클릭
3. 웹 앱용 **reCAPTCHA v3** 선택
4. 사이트 키 등록
5. **"저장"** 클릭

---

## ✅ 확인 체크리스트

적용 후 다음 사항을 확인하세요:

- [ ] Firebase Security Rules 게시됨
- [ ] Firebase Authentication 활성화됨
- [ ] 로그인 방법 설정됨 (Google 또는 이메일/비밀번호)
- [ ] 최소 1명의 사용자 추가됨
- [ ] 웹사이트에서 로그인 테스트 완료
- [ ] 로그인 후 데이터 읽기/쓰기 작동 확인
- [ ] 로그아웃 후 데이터 접근 차단 확인
- [ ] API 키 권한 제한 적용 (선택사항)

---

## 🆘 문제 해결

### 문제: 로그인 후에도 데이터를 읽을 수 없음

**해결:**
1. Firebase Console → Realtime Database → 규칙 확인
2. 규칙이 올바르게 게시되었는지 확인
3. 브라우저 콘솔(F12)에서 오류 메시지 확인
4. `auth != null` 규칙이 제대로 적용되었는지 확인

### 문제: Google 로그인 팝업이 차단됨

**해결:**
1. 브라우저 팝업 차단 해제
2. 다시 시도

### 문제: "auth/unauthorized-domain" 오류

**해결:**
1. Firebase Console → Authentication → Settings → Authorized domains
2. 도메인 추가:
   - `iridescent-sunshine-6dfc58.netlify.app`
   - `localhost`

---

## 📞 지원

문제가 계속되면:
- Firebase 문서: https://firebase.google.com/docs/database/security
- Firebase 지원: https://firebase.google.com/support

---

**마지막 업데이트:** 2025-01-31
