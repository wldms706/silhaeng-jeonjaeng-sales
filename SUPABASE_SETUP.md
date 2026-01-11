# Supabase 설정 가이드

## 1. Supabase 프로젝트 생성

1. [Supabase](https://supabase.com)에 접속하여 로그인
2. "New Project" 클릭
3. 프로젝트 이름 입력 (예: silhaeng-jeonjaeng)
4. 데이터베이스 비밀번호 설정
5. Region 선택 (Northeast Asia - ap-northeast-1 권장)

## 2. 테이블 생성

Supabase 대시보드에서 SQL Editor로 이동 후 아래 SQL 실행:

```sql
-- 클릭 기록 테이블
CREATE TABLE clicks (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 신청 기록 테이블
CREATE TABLE applications (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    status TEXT DEFAULT 'initiated' CHECK (status IN ('initiated', 'submitted', 'paid'))
);

-- RLS (Row Level Security) 활성화
ALTER TABLE clicks ENABLE ROW LEVEL SECURITY;
ALTER TABLE applications ENABLE ROW LEVEL SECURITY;

-- 누구나 읽기 가능 정책
CREATE POLICY "Allow public read" ON clicks FOR SELECT USING (true);
CREATE POLICY "Allow public read" ON applications FOR SELECT USING (true);

-- 누구나 클릭 삽입 가능 정책
CREATE POLICY "Allow public insert" ON clicks FOR INSERT WITH CHECK (true);

-- 실시간 기능 활성화 (선택)
ALTER PUBLICATION supabase_realtime ADD TABLE clicks;
ALTER PUBLICATION supabase_realtime ADD TABLE applications;
```

## 3. API 키 확인

1. Supabase 대시보드 → Settings → API
2. 아래 정보 복사:
   - **Project URL**: `https://xxxxxxxx.supabase.co`
   - **anon public key**: `eyJhbGciOiJIUzI1NiIs...`

## 4. index.html에 적용

`index.html` 파일에서 아래 부분을 찾아 실제 값으로 교체:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';      // ← Project URL 입력
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';  // ← anon key 입력
```

예시:
```javascript
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

## 5. 신청자 관리

### 신청자 추가 (수동)
Supabase 대시보드 → Table Editor → applications에서 직접 추가하거나:

```sql
INSERT INTO applications (status) VALUES ('submitted');
```

### 상태 변경
```sql
UPDATE applications SET status = 'paid' WHERE id = 'uuid-here';
```

### 현재 신청 현황 확인
```sql
SELECT COUNT(*) FROM applications WHERE status IN ('submitted', 'paid');
```

## 6. 보안 참고사항

- `anon key`는 공개되어도 안전합니다 (RLS로 보호됨)
- 신청자 추가/수정은 Supabase 대시보드에서만 가능
- 민감한 작업은 `service_role` 키로 서버에서 처리

## 테스트

Supabase 설정 후 페이지를 새로고침하면:
1. 클릭 수와 신청 수가 DB에서 로드됨
2. 신청 버튼 클릭 시 클릭 수 +1
3. 신청자 4명 이상 시 버튼 자동 비활성화
