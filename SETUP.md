# 협업(실시간 공유) 켜는 법 — 5분

지금 상태로도 사이트는 잘 돌아갑니다. 다만 **체크 내용이 각자 기기에만** 저장돼요.
아래대로 Supabase(무료)에 연결하면 **모두가 같이 체크하고, 실시간으로 함께 반영**됩니다.

---

## 1. Supabase 프로젝트 만들기
1. https://supabase.com 접속 → **Start your project** → 깃허브/구글로 무료 가입
2. **New project** 클릭
   - Name: `retreat` (아무거나)
   - Database Password: 적당히 설정 (메모해두기)
   - Region: `Northeast Asia (Seoul)` 추천
3. 생성까지 1~2분 기다립니다.

## 2. 테이블 만들기 (SQL 한 번 실행)
왼쪽 메뉴 **SQL Editor** → **New query** 에 아래를 통째로 붙여넣고 **Run**.

```sql
create table checklist_items (
  id bigint generated always as identity primary key,
  team text not null,
  text text not null,
  note text default '',
  done boolean not null default false,
  sort double precision not null default 0,
  created_at timestamptz default now()
);

-- 누구나 읽고 쓸 수 있게 (소규모 비공개 팀용)
alter table checklist_items enable row level security;
create policy "public access" on checklist_items
  for all using (true) with check (true);

-- 실시간 동기화 켜기
alter publication supabase_realtime add table checklist_items;
```

> 기본 항목(또는 이미 입력해둔 내용)은 **앱이 처음 연결될 때 자동으로 채워요.** SQL로 따로 넣지 않아도 됩니다.

<details><summary>참고: SQL로 직접 기본 항목을 넣고 싶을 때 (선택)</summary>

```sql
insert into checklist_items (team, text, note, sort) values
  ('exec','팀 내 전참인원 확인 및 역할 분배','',1),
  ('exec','최종 참여자 조사','6/14',2),
  ('exec','인/아웃 차량 조사','6/28',3),
  ('exec','회비 걷기 · 재정 관리','',4),
  ('exec','전체 · 리더 회의 진행','',5),
  ('exec','마무리 점검 · 패킹 총괄','7/26',6),
  ('promo','팀 내 전참인원 확인 및 역할 분배','',7),
  ('promo','홍보 포스터 완료 및 개시','6/13',8),
  ('promo','현수막 완료 및 주문','6/14',9),
  ('promo','홍보용 전단지 완료 및 주문','6/21',10),
  ('promo','전체/성경학교/수련회용 타임테이블 완료','7/19',11),
  ('promo','핸드북 제작','7/19',12),
  ('worship','팀 내 전참인원 확인 및 역할 분배','',13),
  ('worship','예배 찬양 콘티 확정','개회·집회·기도회',14),
  ('worship','저녁 집회 찬양 플레이리스트 공유','6/28',15),
  ('worship','특송 준비','',16),
  ('worship','악기교실 1~3 커리큘럼','',17),
  ('worship','악기 점검 · 관리','',18),
  ('worship','달빛 마을 콘서트 준비','7/29 수요예배',19),
  ('media','팀 내 전참인원 확인 및 역할 분배','',20),
  ('media','음향 장비 점검','',21),
  ('media','영상 장비 준비','',22),
  ('media','집회 · 콘서트 셋업 계획','',23),
  ('mission','팀 내 전참인원 확인 및 역할 분배','',24),
  ('mission','현지 교회 연락 · 협력','가은제일·농암제일·영순중앙',25),
  ('mission','마을 잔치 · 전도 프로그램 기획','',26),
  ('mission','전도 물품 준비','',27),
  ('mission','현지 사역 동선 점검','',28),
  ('mission','선교 보고 · 간증 정리','',29),
  ('lead','팀 내 전참인원 확인 및 역할 분배','',30),
  ('lead','박영만 목사님(가은제일교회) 사전 협의','일정·동선',31),
  ('lead','사역 전체 흐름 · 진행 관리','',32),
  ('lead','기도회 인도 준비','',33),
  ('lead','팀별 사역계획서 취합','6/14',34),
  ('lead','핸드북 내용 총괄','',35),
  ('bible','팀 내 전참인원 확인 및 역할 분배','',36),
  ('bible','최종 사역계획서 완료','6/14',37),
  ('bible','성경학교 1~5 커리큘럼 기획','유치·유년·초등부',38),
  ('bible','달란트 시장 준비','',39),
  ('bible','대상 인원 파악','가은제일·농암제일·영순중앙',40),
  ('bible','교재 · 교구 준비','',41),
  ('bible','물품 구입 및 준비','7/19',42),
  ('camp','팀 내 전참인원 확인 및 역할 분배','',43),
  ('camp','최종 사역계획서 완료','6/14',44),
  ('camp','청소년 수련회 1·2 프로그램 기획','',45),
  ('camp','대상 인원 확인','중등6·고등6·대학생3',46),
  ('camp','집회 · 조모임 준비','',47),
  ('camp','물품 구입 및 준비','7/19',48),
  ('pastor','집회 말씀 준비','1~3집회',49),
  ('pastor','새벽 묵상 · 경건회 인도','',50),
  ('pastor','파송 예배 · 축도','',51),
  ('pastor','현지 교회 목사님과 협의','',52),
  ('pastor','사역 방향 · 영적 점검','',53);
```

</details>

## 3. 키 2개 복사
왼쪽 메뉴 **Project Settings**(톱니) → **API**
- **Project URL** 복사  (예: `https://abcd1234.supabase.co`)
- **Project API keys** 의 **anon / public** 키 복사 (`eyJ...` 로 시작하는 긴 문자열)

> anon 키는 공개되어도 되는 키예요. (서비스 role 키는 절대 넣지 마세요.)

## 4. index.html 에 붙여넣기
`index.html` 을 텍스트 편집기로 열어 맨 위쪽 두 줄을 채웁니다.

```js
const SUPABASE_URL      = "https://abcd1234.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIs...";
```

저장 후 새로고침 → 체크리스트 상단에 **“실시간 공유 중”** 배너가 뜨면 성공!

---

## 5. 다 같이 쓰게 인터넷에 올리기 (Netlify, 무료)
1. https://app.netlify.com/drop 접속
2. `retreat` **폴더를 통째로 드래그&드롭**
3. 몇 초 뒤 나오는 주소(`https://...netlify.app`)를 단톡방에 공유
4. 폰에서 열고 → 공유 버튼 → **홈 화면에 추가** 하면 앱처럼 써요

내용을 수정했을 땐 같은 화면에 폴더를 다시 드래그하면 갱신됩니다.

---

### 참고
- **일정**과 **타임테이블**은 아직 코드(`index.html` 안의 `SCHEDULE`, `TIMETABLE`)에서 수정해요.
  이 두 가지도 화면에서 직접 편집하게 만들고 싶으면 말씀해주세요 — 같은 방식으로 추가할 수 있어요.
- 비용: Supabase·Netlify 모두 이 정도 규모는 **무료 범위** 안입니다.
