# VALIDATION REPORT — Codebase ↔ Thesis Cross-Check

**Огноо:** 2026-04-17
**Скоп:** `Diplom_bichver/main.tex` + `Chapters/Chapter1–4.tex` (canonical) ↔ `driver/` + `merchant_portal/` codebases
**Анхаарахгүй файлууд:** `Latex/`, `DIPLOMA_THESIS.md`, `Chapter5.tex` (main.tex-д include хийгдээгүй)
**Status шошго:** ✅ MATCH | ⚠️ PARTIAL | ❌ NOT FOUND | 🔴 OVERSTATED | 🆕 UNDOCUMENTED

---

## 0. Thesis include структурын ерөнхий тойм

**`main.tex`-д бодитоор include хийгдсэн файлууд:**

| Хэсэг | Файл | Төлөв |
|---|---|---|
| Front | `FrontBackMatter/TittlePage` | ✅ |
| Front | `FrontBackMatter/Abstract` | ✅ |
| Front | `FrontBackMatter/Abberaviation` | ✅ |
| Plan | `Chapters/plan` | ✅ |
| TOC | `FrontBackMatter/TableOfContent` | ✅ |
| Body | `Chapters/Chapter1` | ✅ |
| Body | `Chapters/Chapter2` | ✅ |
| Body | `Chapters/Chapter3` | ✅ |
| Body | `Chapters/Chapter4` | ✅ |
| Body | `Chapters/Chapter5` | ❌ **OGT INCLUDE ХИЙГДЭЭГҮЙ** (`\include` line байхгүй) |
| Body | `Chapters/DataLineageSystem` | 🔇 commented out (зөв — өөр сэдвийн draft) |
| Appendix | `Appendices/AppendixA` | 🔇 commented out — **3.7 KB content PDF-д ороогүй** |
| Bib | `\printbibliography` | 🔇 commented out — **bibliography PDF-д ороогүй** |

**Chapter6/7:** `\include` шугам main.tex-д огт байхгүй; `Chapter6.aux`, `Chapter7.aux` нь зөвхөн хуучин build-ийн үлдэгдэл — minor only.

**Chapter5-ийн агуулга** (commented out тул scope-оос гадуур): "Tdelivery" гэсэн өөр төслийн нэр, "Resend API-аар илгээгдэн" claim, hash-chained audit log future-work тайлбар. Энэ нь хуучин draft бөгөөд Chapter4 нь шинэ canonical conclusion.

**Chapter3 дотор `\iffalse...\fi` блокт хаагдсан subsection-ууд (PDF-д харагдахгүй):**

1. `\subsection{Функционал бус шаардлага}` (line 615–667) — 99.9% uptime, 150 хүргэгч SLA, accessibility — бүгд хасагдсан
2. `\subsubsection{Триггерийн архитектур}` (line 252–274) — `handle_new_user`, `auto_send_epod_on_delivered`, `auto_insert_courier_earnings` — бүгд хасагдсан
3. `\subsubsection{Бодит цагийн архитектур}` (line 352–370) — Realtime механизмын тайлбар хасагдсан
4. `\subsubsection{Хэрэглэгчийн эрхийн түвшин}` (line 440–460) — role table хасагдсан
5. `\subsubsection{Системийн нэгдсэн юзкейс диаграмм}` (line 829–888) — хасагдсан
6. `\subsubsection{Өгөгдлийн бүрэн бүтэн байдлын үндсэн дүрмүүд}` (line 1363–1381) — хасагдсан
7. `\subsubsection{available\_tasks VIEW}` (line 1383–1396) — хасагдсан
8. `\subsection{Үйлчилгээний тусгаарлалт ба blast radius}` (line 1399–1453, label `subsec:service_isolation`) — хасагдсан
9. `\subsection{Scalability архитектур ба чадамжийн хязгаар}` (line 1456–1559, label `subsec:scalability`) — **load test үр дүн, 150 хүргэгч claim, k6 setup, `tab:load_test` бүгд энд хасагдсан**
10. `\subsection{Архитектурын хязгаарлалт}` (line 1562–1636, label `subsec:limitations`) — Outbox pattern, BFF давхарга, schema drift discussion — хасагдсан

**Үр дагавар:** `\ref{subsec:scalability}` (Chapter1 line 27, Chapter3 line 641, line 1472), `\ref{subsec:limitations}` (Chapter3 line 193, line 1443), `\ref{subsubsec:realtime_architecture}` (Chapter3 line 595), `\ref{subsubsec:trigger_architecture}` (implicit) — **бүгд хоосон руу заасан → PDF-д `??` болж харагдана**.

---

## A. Cross-check Tables (8 category)

### A.1. RPC функцүүдийн match (Хүснэгт 4)

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| `claim_delivery_task` нь `FOR UPDATE SKIP LOCKED` ашигладаг | Chapter3 §3.2.5, Хүснэгт 4, line 237 | `driver/supabase/migrations/20260331000001_epod_completed_status.sql:131` ба `20260404000001_phase6_courier_app_compat.sql:122` — `FOR UPDATE SKIP LOCKED;` | ✅ MATCH | Hardware fact баталгаажлаа |
| KYC шалгалт + нэг идэвхтэй даалгавартай байх нөхцөл | Chapter3 line 237 | driver/CLAUDE.md "claim_delivery_task() RPC (KYC + one-active guard)" | ✅ MATCH | partial unique index-ээр баталгаажлаа |
| `update_task_status` (төлвийн шилжилт + auto timestamp) | Chapter3 line 239 | `driver/supabase/migrations/20260401000002_update_task_status_rpc.sql:7` | ✅ MATCH | |
| `get_available_tasks` RLS recursion bypass | Chapter3 line 241 | 5+ migration iteration: `20250301000001_fix_available_tasks_rls.sql`, `20260403000002`, `20260403000003`, `20260417000001_get_available_tasks_join_locations.sql` | ⚠️ PARTIAL | Recursion бодитой асуудал байсан, олон fix migration-аар solveolovsoн — schema-history эмх замбараагүй |
| `request_epod_otp` 6-оронтой OTP, bcrypt, 10 мин | Chapter3 line 243 (Хүснэгт 4) | **merchant-д** `request_epod_otp` (`20260401000002_epod_otps.sql:73`); **driver-т** ижил функц `generate_epod_otp` нэртэйгээр (`20260331000001_epod_completed_status.sql:166`, `20260403000001_add_receiver_email.sql:18`, `20260407000001_fix_epod_use_customer_email.sql:10`) | ⚠️ PARTIAL | **Нэр зөрсөн.** Хүснэгт 4 нь зөвхөн merchant-ыг тусгасан; driver-т `generate_epod_otp` нэртэй. Chapter3 ERD label (line 1359)-д хүртэл "generate_epod_otp" гэж бичсэн → thesis өөрөө дотроо inconsistent. |
| `verify_epod_otp` bcrypt compare, 10 мин, 5 attempt limit | Chapter3 line 245 | merchant: `crypt()` + `attempts >= 5` (`20260401000002_epod_otps.sql:176`); driver: `crypt()` бий, **5 attempt limit driver/migrations-д БАЙХГҮЙ** — зөвхөн `driver/security-tests/migrations/20260417000001_otp_lockout.sql`-д шинээр нэмсэн | ⚠️ PARTIAL | merchant хувилбарт хэрэгжсэн; driver runtime хувилбар lockout-гүй. Phase 2-д "5 attempt lockout" claim-ийг merchant-д хязгаарлах ёстой. |
| `is_approved_courier` RLS helper | Chapter3 line 247 | `driver/supabase/migrations/20250101000001_courier_auth_schema.sql:167` ба `20250101000005_fix_rls_recursion.sql:32` | ✅ MATCH | |
| Бүгд `SECURITY DEFINER` | Chapter3 line 226 | Bulk publish RPC `SECURITY DEFINER SET search_path = public` (`20260411000001_bulk_publish_rpc.sql:26`) болон бусад RPC хувилбарууд тодорхой | ✅ MATCH | `pgcrypto` extension-ы `search_path` асуудлыг `20260406000002_fix_pgcrypto_search_path.sql`-аар зассан — schema drift evidence |
| `resend_epod_otp` нэмэлт RPC | — (thesis-д дурдаагүй) | `driver/supabase/migrations/20260331000001_epod_completed_status.sql:297`, `20260403000001_add_receiver_email.sql:86`, `security-tests/migrations/20260417000001_otp_lockout.sql:119` | 🆕 UNDOCUMENTED | Хүснэгт 4-д нэмэх ёстой |
| `create_delivery_task` нэмэлт RPC | — (thesis-д дурдаагүй) | `merchant_portal/supabase/migrations/20260331000002_merchant_portal.sql:168`, `20260403000002_repair_create_delivery_task.sql:17`, `20260401000001_phase1_fixes.sql:25` | 🆕 UNDOCUMENTED | Daалгавар үүсгэх RPC |
| `publish_delivery_tasks_bulk` (bulk publish RPC) | — (thesis-д дурдаагүй) | `merchant_portal/supabase/migrations/20260411000001_bulk_publish_rpc.sql:23` | 🆕 UNDOCUMENTED | **Phase 2-ийн bench evidence-ийн гол RPC** — энэ функцийн throughput тоо `summary-2026-04-17T03-31-15.csv`-д бий |
| `current_org_id()` helper | — (thesis-д `get_user_org_id()` гэж дурдсан) | bulk_publish RPC body-д `public.current_org_id()` дуудсан | ⚠️ PARTIAL | **Helper-ийн нэр зөрсөн** (Хүснэгт 5-д `get_user_org_id` гэж нэрлэсэн, кодод `current_org_id`) |

### A.2. RLS policies-ийн match (Хүснэгт 5)

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| `profiles`: `auth.uid() = id` | Chapter3 §3.2.6, Хүснэгт 5, line 326 | `driver/supabase/migrations/20250101000005_fix_rls_recursion.sql:73` `profiles_select_own ... USING (auth.uid() = id)` (5+ migration-д давтагдсан) | ✅ MATCH | |
| `delivery_tasks` (courier read): `is_approved_courier()` AND `status='published'` OR `courier_id = auth.uid()` | Chapter3 line 328 | `driver/supabase/migrations/20260331000001_epod_completed_status.sql:62` `delivery_tasks_courier_select`; `20250301000001_fix_available_tasks_rls.sql:21` `dt_courier_select_published` | ✅ MATCH | Дулаанаар evolve хийсэн (5+ хувилбар) |
| `delivery_tasks` (merchant): `org_id = get_user_org_id()` | Chapter3 line 330 | `merchant_portal/supabase/migrations/20250101000002_delivery_tasks.sql:73,76,79` "Org users can ..."; `20260411000002_fix_merchant_update_recursion.sql:27` `merchant_update_tasks` | ⚠️ PARTIAL | Helper-ийн нэр кодод `current_org_id()` (Хүснэгт 5-д `get_user_org_id` гэж бичсэн) |
| `courier_earnings`: `courier_id = auth.uid()` | Chapter3 line 332 | `driver/supabase/migrations/20250101000001_courier_auth_schema.sql:318` `courier_earnings_own`; `merchant_portal/.../20260331000002_merchant_portal.sql:144` `courier_select_own_earnings` | ✅ MATCH | |
| `courier_kyc`: courier own + admin | Chapter3 line 334 | `driver/supabase/migrations/20250224000001_kyc_support.sql:59,64,68,73,83` (5 policy: view own, submit, update pending, admin view, admin review) | ✅ MATCH | |
| `delivery_epod_otps`: merchant own org | Chapter3 line 336 | `merchant_portal/supabase/migrations/20260401000002_epod_otps.sql:49` `merchant_select_epod_otps` | ✅ MATCH | |
| `locations` хүснэгтийн RLS | — (thesis-д дурдаагүй) | `merchant_portal/.../20250101000002_delivery_tasks.sql:70-71` "Anyone can insert locations" + "Anyone can view locations" — **public access!** | 🆕 UNDOCUMENTED + ⚠ риск | Locations нь практикт public-аар нээлттэй — security model-д bypass hole |
| `products`, `orders`, `order_items`, `payments`, `task_items`, `org_settings` RLS | — (thesis-д дурдаагүй) | 20+ нэмэлт policy: `orders_select_merchant`, `payments_select_customer`, `order_items_*`, `products_*`, `merchant_manage_own_settings` | 🆕 UNDOCUMENTED | Хүснэгт 5 нь зөвхөн 6 хүснэгт жагсаасан — нийт ~25 policy code-д бий. Thesis нь бодит scope-оос 4× бага дүрсэлсэн |
| Schema drift evidence | — | `profiles_select_own` policy 5+ хувилбараар олон migration-д давтагдан үүссэн (CREATE POLICY, DROP, CREATE again pattern) | ⚠️ | "Migration-аар л schema удирдана" claim-тай хэсэгчлэн зөрчилдөнө |

### A.3. Database schema vs ERD (Зураг 15)

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| ERD-д 9 хүснэгт | Chapter3 §3.8.1, Зураг 15 (line 1188) | Бодит хүснэгт жагсаалт: `auth.users`, `profiles`, `organizations`, `org_settings`, `locations`, `products`, `orders`, `order_items`, `payments`, `delivery_tasks`, `task_items`, `task_assignments`, `available_tasks` (VIEW), `courier_kyc`, `courier_earnings`, `delivery_epod_otps` = **14+ table + 1 view** | 🔴 UNDERSTATED | "9 үндсэн хүснэгт" claim бодитоос ~1.5× бага. Шинэ хүснэгт нэмэх ёстой ERD-д: `org_settings`, `products`, `order_items`, `task_items`, `courier_earnings`, `delivery_epod_otps` |
| `task_events` хүснэгтэд төлөв өөрчлөлт хадгалах | Chapter2 §2.2 line 278 | grep `task_events` ❌ NO MATCH in бүх `.sql` файл | ❌ NOT FOUND | Хүснэгт огт байхгүй |
| `status_history` хүснэгт | Chapter2 §2.2 line 278 | grep `status_history` ❌ NO MATCH | ❌ NOT FOUND | |
| "Event-driven audit log, immutable бичлэг" | Chapter2 Хүснэгт 2 (`tab:comparative_analysis` line 330), Chapter4 Хүснэгт `tab:unified_comparison` line 55, Chapter1 §1.1 | grep `audit_log`, `REVOKE.*UPDATE`, immutable trigger ❌ NO MATCH | 🔴 OVERSTATED | Audit trail дэд бүтэц **бүхэлдээ хийгдээгүй**. Chapter4-д "audit log нь цаашдын ажил" гэж хэлж бий — гэвч Chapter2-д "хэрэгжүүлсэн" гэж claim хийсэн → дотоод зөрчилдөөн |
| `delivery_epod_otps` хүснэгтийн нэр | Chapter3 Хүснэгт 5 line 336 | merchant_portal-ийн RLS policy `merchant_select_epod_otps`, table нэр `delivery_epod_otps` ✅ | ✅ MATCH | |
| Chapter3 Хүснэгтийн тайлбар list (line 1359) "epod_otps" | Chapter3 line 1359 | Бодит нэр `delivery_epod_otps` | ⚠ INCONSISTENT | Thesis дотроо нэр зөрсөн (Хүснэгт 5: `delivery_epod_otps`, тайлбар: `epod_otps`) |
| `available_tasks` нь "хүснэгт" эсвэл "VIEW" | Chapter3 §3.8.1 implicit table; commented section line 1383 ("VIEW юм" гэж зөв тайлбарласан) | `merchant_portal/.../20250224000001_delivery_publishing.sql:358` `CREATE VIEW available_tasks AS`; `20260403000003_repair_available_tasks.sql:27` `CREATE VIEW` | ⚠️ PARTIAL | Зөв тайлбар (`subsubsection{available_tasks VIEW}`) `\iffalse`-аар хаагдсан → PDF-д харагдахгүй |
| ERD-д `courier_earnings` | Зураг 15 figure | TikZ ERD figure-д **байхгүй** (зөвхөн hand-list-д дурдсан line 1358) | ⚠️ PARTIAL | ERD figure нь бүх 9 entity дүрсэлсэн ч `courier_earnings`, `epod_otps`, `task_items`, `order_items`, `org_settings`, `products` бүгд figure-д **байхгүй** |
| `task_assignments` хүснэгт | Зураг 15, Chapter3 line 1300 | grep `CREATE TABLE.*task_assignments` болон `task_assignments` ⚠️ Зөвхөн Зураг 15-д бий, бодит code-д migration ОЛДОХГҮЙ | ❌ NOT FOUND | ERD-д бий, кодод хүснэгт байхгүй магадлалтай. Phase 2-д баталгаажуулах |

### A.4. Client app-ийн claim validation

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| Courier app: `restoreSession()` app start-д | Chapter3 §3.2.4 line 217 | `driver/src/context/CourierAuthContext.tsx` (CLAUDE.md confirms) | ✅ MATCH | |
| Courier: "Төлөв 'pending' бол 60 секунд тутам автоматаар шинэчилнэ" | Chapter3 line 220 | `driver/src/context/CourierAuthContext.tsx:87` `statusRefreshInterval = 60000`; line 156 `setInterval(async () => ...)` | ✅ MATCH | |
| `CourierAuthContext` баталгаажуулалтын төлвийг удирдана | Chapter3 §3.2.4 line 216 | `driver/src/context/CourierAuthContext.tsx` (file бий, hook export-той) | ✅ MATCH | |
| AsyncStorage-д session хадгална | Chapter3 line 215 | driver/CLAUDE.md "AsyncStorage persistence", `driver/src/config/supabaseClient.ts` ашигладаг | ✅ MATCH | |
| Merchant: orders 3 таб ("Үйлдэл шаардлагатай / Идэвхтэй / Түүх") | Chapter3 §3.5 line 552–560, line 402 | `merchant_portal/app/(merchant)/orders/page.tsx:7` `Tabs` import; line 296 `<Tabs activeKey={activeTab}>` | ✅ MATCH | Кодод 3-tab структур баталгаажлаа |
| Merchant: Next.js middleware session шалгалт | Chapter3 §3.2.4 line 207 | `merchant_portal/middleware.ts`, `lib/supabase/middleware.ts` | ✅ MATCH | merchant CLAUDE.md confirm |
| Merchant: Recharts графикаар analytics | Chapter3 §3.3.1 line 404 | `merchant_portal/app/(merchant)/analytics/page.tsx:25` `from "recharts"`; `package.json: recharts ^3.8.1` | ✅ MATCH | |
| Merchant: KPI dashboard (нийт/идэвхтэй/дууссан, өнөөдөр vs өчигдөр) | Chapter3 §3.3.1 line 399 | `merchant_portal/app/(merchant)/dashboard/` бий — content шалгаагүй | ⚠️ PARTIAL | Folder бий, бодит KPI implementation дотроо шалгах хэрэгтэй |
| Merchant: Bulk publish UI + backend logic | Chapter3 §3.5 line 401 ("олноор нийтлэх (bulk publish)") | Backend: `publish_delivery_tasks_bulk` RPC ✅; UI: `merchant_portal/scripts/bench/bulk-publish/` нь bench-аар л дуудаж байгаа — frontend bulk-publish UI бодоор бий эсэх шалгаагүй | ⚠️ PARTIAL | Backend ✅; UI side кодоор баталгаажуулаагүй |

### A.5. Technology stack (Хүснэгт 6)

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| Expo 54 | Chapter3 Хүснэгт 6 line 684 | `driver/package.json: expo ~54.0.31` | ✅ MATCH | |
| TypeScript 5.9 | Chapter3 Хүснэгт 6 line 684 | `driver/package.json: typescript ~5.9.2`; `merchant_portal: typescript ^5` | ✅ MATCH | |
| React Native (default Expo) | line 684 | `driver/package.json: react-native 0.81.5` | ✅ MATCH | |
| React 19 | line 684, 686 | `driver: react 19.1.0`; `merchant: react 19.2.3` | ✅ MATCH | |
| React Hook Form + Zod (driver app) | line 684 | driver: `react-hook-form ^7.49.2`, `zod ^3.22.4`, `@hookform/resolvers ^3.3.2`; ашиглагдсан: `LoginScreen.tsx`, `RegisterScreen.tsx`, `CourierRegisterScreen.tsx` | ✅ MATCH | driver-д бий |
| **React Hook Form (merchant app — implied by Хүснэгт 6 общим)** | line 684, 686 | merchant package.json: **react-hook-form БАЙХГҮЙ**; merchant нь `Form.useForm` (antd) ашигладаг (`settings/page.tsx:43`, `tasks/new/page.tsx:53`, `products/ProductsClient.tsx:69`) | 🔴 OVERSTATED | "React Hook Form" зөвхөн driver-д үнэн; merchant-д ашиглаагүй |
| AsyncStorage | line 684 | `driver: @react-native-async-storage/async-storage ^2.1.0` | ✅ MATCH | |
| Reanimated | line 684 | `driver: react-native-reanimated ~4.1.1` | ✅ MATCH | |
| Next.js 16.1 | line 686 | `merchant: next 16.1.4` | ✅ MATCH | |
| Ant Design | line 686 | `merchant: antd ^6.2.1`, `@ant-design/icons ^6.1.0`, `@ant-design/nextjs-registry ^1.3.0` | ✅ MATCH | |
| Tailwind CSS | line 686 | `merchant: tailwindcss ^4` (v4); `driver: tailwindcss 3.3.2` (v3 NativeWind) | ⚠️ PARTIAL | merchant v4, driver v3 — version зөрсөн |
| PostgreSQL 15+ | line 688 | Кодоор шалгаагүй (Supabase Pro tier — verifiable but no doc evidence in repo) | ⚠️ PARTIAL | Phase 2-д "Supabase Pro tier defaults to PostgreSQL 15" гэх ишлэл нэмэх |
| **NativeWind (Tailwind for React Native)** | — | `driver: nativewind ^4.2.2` ашигласан, driver/CLAUDE.md confirm | 🆕 UNDOCUMENTED | Хүснэгт 6-д нэмэх ёстой |
| **Recharts** | — | `merchant: recharts ^3.8.1` ашигласан | 🆕 UNDOCUMENTED | Хүснэгт 6-д нэмэх ёстой (analytics-д ашигласан) |
| **Nodemailer (Gmail SMTP)** | — | `merchant: nodemailer ^8.0.4`; driver edge function: nodemailer + Gmail | 🆕 UNDOCUMENTED + ⚠ Chapter5 contradiction | Chapter5 (excluded) "Resend API" гэж бичсэн нь зөвхөн **уг код-той зөрчилдсөн** — Gmail SMTP via nodemailer ашиглагдсан |
| dayjs, lucide-react, @supabase/ssr, react-native-maps, expo-* extra packages | — | merchant: dayjs, lucide-react; driver: react-native-maps 1.20.1, lucide-react-native | 🆕 UNDOCUMENTED | Хүснэгт 6-д нэмэх боломжтой (optional) |
| `react-native-maps 1.20.1` бий (driver) | — | `driver/package.json: react-native-maps 1.20.1` | 🆕 UNDOCUMENTED + ⚠ | Chapter4 Хүснэгт line 129 "Бодит цагийн байршил мөшгөлт = ❌ хийж чадаагүй" — gehded library бий → "lib бий, integration алга" гэж тодруулах |

### A.6. Business logic claim validation

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| Task status flow: `draft → published → assigned → picked_up → delivered → completed` | Chapter3 §3.5 line 581–588 | merchant CLAUDE.md confirm; `trg_enforce_delivery_task_transition` trigger (`20250224000001_delivery_publishing.sql:158`) шилжилт enforce | ✅ MATCH | |
| Task status `failed` | — (thesis-д байхгүй) | merchant CLAUDE.md "also `cancelled`, `failed`" | 🆕 UNDOCUMENTED | Status enum-д `failed` бий, thesis-д тэмдэглэх ёстой |
| Order status flow: төлбөр хүлээгдэж буй → төлбөр хийгдсэн → бэлтгэж буй → ... | Chapter3 §3.4 line 538–543 | `merchant_portal/.../20250101000004_orders_system.sql:163` `trg_enforce_order_status` trigger; `20250101000004_orders_system.sql:338` `trg_payment_confirmed`, line 345 `trg_payment_confirmed_on_insert` | ✅ MATCH | |
| Төлбөрийн баталгаажуулалт trigger-ээр | Chapter3 line 546, 550 | `trg_payment_confirmed` (payment INSERT/UPDATE → orders.status auto-update) | ✅ MATCH | |
| ePOD амжилттай үед `courier_earnings` автомат | Chapter3 line 607, Chapter4 line 109 | `auto_insert_courier_earnings()` + trigger (`driver/.../20260404000001_phase6_courier_app_compat.sql:31, 57`); merchant `auto_send_epod_on_delivered()` trigger (`20260404000001_auto_epod_on_delivered.sql:24, 104`) | ✅ MATCH | |
| `courier_earnings` хүснэгт | Chapter4 line 111 | `driver/.../20250101000001_courier_auth_schema.sql:300` `CREATE TABLE`; `merchant_portal/.../20260331000002_merchant_portal.sql:132` бас CREATE | ✅ MATCH (duplicated definition — minor) | Хүснэгт хоёр project-д тус тусдаа CREATE TABLE — schema drift эрсдэл |
| Bulk publish single-statement UPDATE | — (thesis-д дурдаагүй) | `publish_delivery_tasks_bulk(p_task_ids UUID[])` нэг `UPDATE ... WHERE id = ANY(...) AND status='draft' AND org_id = current_org_id() RETURNING *` | 🆕 UNDOCUMENTED | Bench-ийн "bulk_rpc" стратегийн source. Phase 2-д бичих гол нотолгоо |
| **Payment provider жагсаалт: qpay, stripe, bank, cash** | Chapter3 §3.8.1 ERD line 1356 | Кодоос баталгаажуулаагүй (payments enum шалгах) | ⚠ TBD | Quick check needed |
| Status transition trigger immutability | — (claim тодорхой биш) | `trg_merchant_courier_immutable` (`20260411000002_fix_merchant_update_recursion.sql:51`) — merchant нь `courier_id`-г өөрчилж чадахгүй | 🆕 UNDOCUMENTED | "Immutable хэсэгчилсэн" хэлбэр — column-level immutability бий |

### A.7. "Overclaim/judder" detection (rubric-аар)

| Thesis claim | Thesis location | Code evidence | Status | Comment |
|---|---|---|---|---|
| **"Marketplace-based динамик хуваарилалт, алгоритмын түвшин"** ⋆⋆⋆ | Chapter2 Хүснэгт `tab:comparative_analysis` line 324, Chapter4 line 49 | Бодитоор: `claim_delivery_task` нь `WHERE status='published' AND id=p_task_id FOR UPDATE SKIP LOCKED` — **First-Come-First-Served claim** (no algorithm, no recommendation, no prioritization, no matching score) | 🔴 OVERSTATED | "FCFS claim with concurrency protection" гэж нэрлэх нь үнэн зөв. Marketplace algorithm нь зөвхөн liquidity (хэн хурдан claim хийсэн нь авах) — энэ нь "алгоритмын түвшний" хуваарилалт биш |
| **"Event-driven audit log, immutable бичлэг"** ⋆⋆⋆⋆ | Chapter2 Хүснэгт 2 line 330, Chapter4 line 67 | `task_events`, `status_history`, `audit_log`, `REVOKE UPDATE`, immutable trigger ❌ NONE EXIST | 🔴 OVERSTATED | Чадамж хийгдээгүй. Chapter4 line 55 өөрөө "бүрэн audit log нь цаашдын ажил" гэж зөв хүлээн зөвшөөрсөн → Хүснэгт 2-д ⋆⋆⋆⋆ үнэлгээ нийцэхгүй |
| **"Horizontal scalability дэмжлэг"** ⋆⋆⋆⋆ | Chapter2 Хүснэгт 2 line 336, Chapter4 line 67 | Single Supabase Pro instance, no read replica, no sharding, no caching layer, no queue. Bulkhead/connection pool разделение `subsec:service_isolation` `\iffalse`-аар хаагдсан | 🔴 OVERSTATED | "Vertical scaling within Supabase Pro tier; horizontal roadmap planned but unimplemented" гэх нь үнэн |
| **"150 concurrent хүргэгчийг тэсвэрлэх баталгаатай, load test-ээр шалгасан"** | Chapter1 §1.2 line 27, Chapter4 implicit | Driver: `benchmarks/load_test/claim_race.k6.js` script бэлэн боловч **run хийгдээгүй, result CSV/JSON алга** | ❌ NOT FOUND (evidence) | Code бэлэн байгаа ч empirical result бэлдээгүй. Phase 2-д "code is ready, run pending" гэж шударга бичих эсвэл local Supabase-д run хийх |
| **"Bulk publish" feature** | Chapter3 §3.5 line 401 | Backend RPC ✅ (`publish_delivery_tasks_bulk`); UI шалгаагүй; **bench data ✅** (raw + summary CSV бий, 2026-04-17 run хийгдсэн) | ⚠️ PARTIAL | Backend + benchmark бэлэн; UI integration evidence шалгах ёстой |
| **"Санхүүгийн тайлан, аналитик, Recharts"** | Chapter3 line 403–404 | `app/(merchant)/financials/`, `app/(merchant)/analytics/` folder бий; analytics page Recharts import ✅ | ✅ MATCH | Implementation бодитоор бий |
| **"Automatic TypeScript type generation (Supabase CLI gen types)"** | Chapter3 §3.2.3 line 184 | `merchant_portal/types/database.ts` (8.9 KB, бий) ✅; `driver/src/types/`-д **`database.ts` БАЙХГҮЙ** (зөвхөн hand-written cart/order/shop/user/index) | ⚠️ PARTIAL | merchant-д хэсэгчлэн match; driver-т claim үнэн биш |
| **"CI pipeline-д Supabase gen types шат"** | Chapter3 §3.2.3 line 186 | grep `.github/workflows/` ❌ NO directory in аль алинд | 🔴 OVERSTATED | CI workflow огт байхгүй. "Schema drift prevention 4-strategy"-ийн нэг шат бүхэлдээ хийгдээгүй |
| **"Migration-аар л schema удирдана, Supabase Studio гараар хориглосон"** | Chapter3 §3.2.3 line 190 | Migration файл олонтаа `FIX_*.sql`, `DEBUG_*.sql`, `QUERIES_ONLY_*.sql`, `nuclear_drop_all_triggers.sql`, `manual_apply.sql`, `remote_only.sql`, `reverted.sql` гэсэн хувилбарууд байгаа | ⚠️ PARTIAL | Migration discipline практикт сул — нэр оноох convention зөрчигдсөн. driver-д `supabase/migrations/supabase/` гэсэн nested folder бий |

### A.8. Тасалдсан reference-үүд (??)

| ?? location | Thesis location | Target label | Target status | Status |
|---|---|---|---|---|
| "3-р бүлгийн ??-т дэлгэрэнгүй" | Chapter1 line 27 | `subsec:scalability` | Chapter3 line 1456-д label бий, бүхэл subsection `\iffalse...\fi`-аар хаагдсан | ❌ BROKEN |
| "энэ тухайг ?? хэсэгт дэлгэрүүлнэ" | Chapter3 line 193 | `subsec:limitations` | Chapter3 line 1562-д label бий, `\iffalse`-аар хаагдсан | ❌ BROKEN |
| "(дэлгэрэнгүйг \\ref{subsubsec:realtime_architecture} хэсэгт)" | Chapter3 line 595 | `subsubsec:realtime_architecture` | Chapter3 line 354-д label commented out (`%`-аар) | ❌ BROKEN |
| Chapter3 line 1472 `(\ref{subsubsec:roadmap}` | Chapter3 line 1472 | `subsubsec:roadmap` | Same `\iffalse` block | ❌ BROKEN (commented section дотор internal ref) |
| Chapter3 line 1443 `(\ref{subsec:limitations})` | Chapter3 line 1443 (commented) | `subsec:limitations` | Same `\iffalse` block | ❌ BROKEN (commented section дотор internal ref) |
| Implicit: trigger architecture-аас хамаарал | Chapter3 line 482 `\ref{subsubsec:auth_architecture}-р хэсэгт тодорхойлсон` | `subsubsec:auth_architecture` | Chapter3 line 196 label бий ✅ | ✅ OK |
| `\ref{subsubsec:rpc_architecture}` | Chapter3 line 593, 610 | line 224 label бий | ✅ OK | ✅ OK |
| `\ref{subsubsec:security_architecture}` | Chapter3 line 508 | line 277 label бий | ✅ OK | ✅ OK |
| **Hidden ??-уудын тоо** | — | — | **5+ broken reference** (Chapter1, Chapter3-аас active section-ууд commented section руу заасан) | — |

---

## B. Critical Findings (red flags)

### 🔴 1. "Event-driven, immutable audit log" нь бүхэлдээ **fabricated**

- **Бичвэр:** Chapter2 §2.2 line 278 ("task_events ба status_history хүснэгтээр ... Бүх төлөв өөрчлөлтийг өөрчлөх боломжгүй (immutable) байдлаар хадгалах"); Хүснэгт `tab:comparative_analysis`-д ⋆⋆⋆⋆ үнэлгээ
- **Код:** `task_events`, `status_history`, `audit_log` хүснэгт **бүгд байхгүй**; `REVOKE UPDATE`, immutable-enforcing trigger **бүгд байхгүй**
- **Chapter4 line 55-д өөрөө "бүрэн audit log нь цаашдын ажил"** гэж зөв хүлээн зөвшөөрсөн — Chapter2 Хүснэгт 2 нь өөрөө thesis-ийн дотор contradict хийсэн
- **Эрсдэл:** Багшийн нүдэнд хамгийн илрүүлэхэд хялбар overclaim — асууж шалгасан тохиолдолд хариу мэдэгдэх боломжгүй

### 🔴 2. "150 concurrent хүргэгч + load test" нь evidence-гүй

- **Бичвэр:** Chapter1 §1.2 line 27 шууд "ойролцоогоор 150 concurrent хүргэгчийг тэсвэрлэх баталгаатай бөгөөд уг тоон хязгаарыг load test-ээр шалгасан"
- **Код:** `driver/benchmarks/load_test/claim_race.k6.js` script бий; үр дүн файл (`*.csv`, `*.json`, `*report*`) **бүгд байхгүй**
- **Үүн дээр нэмж:** Chapter3-ийн `subsec:scalability` section (k6 setup, `tab:load_test` хүснэгт, p95/p99 latency) **бүхэлдээ `\iffalse`-аар хаагдсан** → Chapter1-аас үүн рүү заасан `\ref` нь PDF-д `??` болж харагдана
- **Эрсдэл:** Бичвэрт SLA too дурдсан атлаа дэмжих empirical evidence үзүүлэхгүй — багш "150 нь хаанаас гарсан тоо вэ?" гэж асуувал хариу мэдэгдэх боломжгүй

### 🔴 3. "Marketplace-based динамик хуваарилалт, алгоритмын түвшин" нь FCFS

- **Бичвэр:** Chapter2 Хүснэгт 2 line 324, Chapter4 Хүснэгт line 49 ⋆⋆⋆ үнэлгээтэй
- **Код:** `claim_delivery_task` RPC нь зөвхөн `WHERE id = $1 AND status = 'published' FOR UPDATE SKIP LOCKED` — **алгоритм, оноо, ranking, recommendation байхгүй**. Энэ нь "First-Come-First-Served claim with concurrency protection"
- **Эрсдэл:** "Алгоритмын түвшин" гэх клейм нь хувилбар бүрд "matching score эсвэл prioritization функц байгаа юу?" гэсэн асуултад хариу олохгүй

### 🔴 4. "Schema drift prevention" дөрвөн стратеги — нэг шат огт хийгдээгүй

- **Бичвэр:** Chapter3 §3.2.3 line 181-191 — 4 шат: (1) Auto TS gen, (2) **CI pipeline integration**, (3) Compile-time enforcement, (4) Migration-only schema
- **Код:**
  - (1) `merchant_portal/types/database.ts` (8.9 KB) ✅; `driver/src/types/`-д **`database.ts` байхгүй** (manual types only)
  - (2) `.github/workflows/` directory **аль алинд байхгүй** → CI шат **fabricated**
  - (3) `createClient<Database>(...)` ашиглалт зөвхөн merchant-д
  - (4) Migration discipline сул: `FIX_*.sql`, `DEBUG_*.sql`, `QUERIES_ONLY_*.sql`, `nuclear_drop_all_triggers.sql`, `manual_apply.sql`, `reverted.sql`, `remote_only.sql` зэрэг migration root-д hanging — convention зөрчигдсөн
- **Эрсдэл:** §3.2.3 нь дэлгэрэнгүй strategy section боловч 4-ний 1 шат бодоор хийгдээгүй; bichver "schema drift prevention" claim-ыг merchant-д хязгаарлах ёстой

### 🔴 5. Chapter3-ийн **17%** content `\iffalse...\fi` блокт хаагдсан, ?? reference үүсгэсэн

- **Бичвэр:** Chapter3-ын 1636 мөрөөс ~10 subsection (load test, limitations, blast radius, realtime, triggers, available_tasks VIEW, etc.) `\iffalse`-аар хаагдсан
- **Үр дагавар:** 5+ broken `\ref` (deta detail A.8-д жагсаасан); load test result хүснэгт `tab:load_test` PDF-д харагдахгүй атал Chapter1 нь үүн рүү ишилсэн
- **Эрсдэл:** PDF-ийг compile хийхэд "??" placeholder харагдана — багш нэн даруй мэдэгдэх

### 🔴 6. Дутуу chapter-ууд: Chapter5 + Appendix + Bibliography PDF-д ороогүй

- **Бичвэр:** `main.tex` дотор Chapter5 `\include` line огт байхгүй; AppendixA + bibliography commented out
- **Код:** Chapter5.tex (7.8 KB), AppendixA.tex (3.7 KB), references.bib бэлэн файл байгаа
- **Үр дагавар:** Bibliography байхгүй академик ажил → `\cite{}` хийсэн ишлэлүүд бүгд "[?]" болно. (Дашрамд: Chapter1-д бараг ишлэл байхгүй гэж harav гэхдээ Chapter2-д ишлэл байх магадлалтай — өргөтгөсөн grep шаардлагатай)
- **Эрсдэл:** Bibliography байхгүй thesis нь академик стандартын хатуу нийцлээс гадуур

### 🔴 7. ePOD OTP функцийн нэр zörtsöv: `request_epod_otp` (merchant) vs `generate_epod_otp` (driver)

- **Бичвэр:** Chapter3 Хүснэгт 4 (line 243) `request_epod_otp` гэж нэрлэсэн; Chapter3 commented тайлбар list (line 1359) `generate_epod_otp` гэж нэрлэсэн (бичвэр дотроо inconsistent)
- **Код:** merchant `request_epod_otp(p_task_id)`; driver `generate_epod_otp(p_task_id)` — өөр өөр RPC дуудлагатай. driver Edge Function `send-otp-email` нь `generate_epod_otp` дуудна; merchant Edge Function `send-epod-otp` нь `request_epod_otp` дуудна
- **Эрсдэл:** Хүснэгт 4 нь системийг merchant хувилбараар л төлөөлсөн — driver-ын RPC table dieser-д ороогүй. Phase 2-д duplicate row нэмэх ёстой

### 🔴 8. Driver-т `5 attempt OTP lockout` claim бодит код-д шууд хэрэгжээгүй

- **Бичвэр:** Chapter3 §3.2.6 line 347 "Хамгийн ихдээ 5 удаа оруулах боломжтой, хэтэрвэл шинэ код шаардлагатай"; Хүснэгт 4 line 245 `verify_epod_otp` "5 оролдлогын хязгаар"
- **Код:**
  - merchant: ✅ `attempts >= 5` check `20260401000002_epod_otps.sql:176`
  - driver/supabase/migrations: **5 attempt limit БАЙХГҮЙ** — production runtime-д lockout логикгүй
  - Зөвхөн `driver/security-tests/migrations/20260417000001_otp_lockout.sql`-д шинээр нэмсэн (production-д apply хийсэн эсэх тодорхойгүй)
- **Эрсдэл:** Чичр security тестийн scenario бичсэн ч production driver schema-д lockout байхгүй. Brute-force тестийн result нь production-trace-той зөрчилдөж магадгүй

---

## C. Minor Inconsistencies

| # | Issue | Detail |
|---|---|---|
| C1 | `delivery_epod_otps` vs `epod_otps` нэрэн зөрүү | Chapter3 Хүснэгт 5 (line 336): `delivery_epod_otps`; Chapter3 ERD label list (line 1359): `epod_otps`. Бодит хүснэгт `delivery_epod_otps` |
| C2 | `get_user_org_id()` vs `current_org_id()` | Хүснэгт 5-д thesis нь `get_user_org_id()` гэж нэрлэсэн; bulk_publish RPC body-д `current_org_id()` ашиглагдсан. Helper function нэр зөрсөн |
| C3 | bcrypt cost зөрүү | merchant: cost 8 (`gen_salt('bf', 8)`); driver: cost 4 (`gen_salt('bf', 4)`); security-tests upgrade: cost 10. Cost 4 нь production grade-д сул |
| C4 | TittlePage.tex (typo) | "TittlePage" → ёсоор "TitlePage". Folder-д `TitlePage.aux` тус тусдаа бий — case-insensitive filesystem-д ажилладаг magado |
| C5 | "Abberaviation.tex" (typo) | "Abbreviation" → "Abberaviation". `Abbreviation.aux` өөр файл бий — өөр build-аас үлдэгдэл |
| C6 | `Chapter6.aux`, `Chapter7.aux` orphan | Chapter6.tex / Chapter7.tex файл байхгүй, `\include` шугам байхгүй; зөвхөн хуучин build artifact |
| C7 | "9 үндсэн хүснэгт" ERD claim | Бодит ~14+ хүснэгт + 1 view. ERD figure-д 9 entity дүрсэлсэн нь зөв, гэхдээ "үндсэн" гэсэн word-ыг clarify хийх хэрэгтэй |
| C8 | Tailwind version зөрүү | merchant: v4 (`^4`); driver: v3 (`3.3.2`) — бичвэрт version differentiate хийх ёстой |
| C9 | `available_tasks` хүснэгт vs VIEW | merchant CLAUDE.md "Core tables: ... available_tasks" гэж буруу; кодоор `CREATE VIEW available_tasks` — Phase 2-д Хүснэгтийн тайлбарт "VIEW" тэмдэг нэмэх |
| C10 | Driver-ын `supabase/fix_remote_now.sql`, `FIX_claim_delivery_task.sql`, `QUERIES_ONLY_available_tasks.sql`, `DEBUG_rls_available_tasks.sql` | Migration discipline-аас гадуур, root supabase folder-д hanging. `driver/supabase/migrations/supabase/` гэсэн nested folder бас бий |
| C11 | Driver-ын `nuclear_drop_all_triggers.sql` | "Nuclear drop" гэсэн нэртэй migration-ы дотор triggers DROP, дараа CREATE — emergency fix evidence |
| C12 | Хоёр project-д `courier_earnings` `CREATE TABLE` хоорондын зөрүү | driver `20250101000001_courier_auth_schema.sql:300` ба merchant `20260331000002_merchant_portal.sql:132` тус тусдаа CREATE — schema duplicate (linked Supabase project-д аль аль нь run хийгдвэл нэг нь fail болно) |
| C13 | "Resend API" (Chapter5, excluded) — кодод nodemailer/Gmail | Chapter5 нь PDF-д ороогүй тул PDF-д харагдахгүй ч hidden inconsistency |
| C14 | Driver-ын locations xусны "Anyone can view/insert" RLS бодлого | merchant `20250101000002_delivery_tasks.sql:70-71` нийтэд нээлттэй — security-аас санаа зовох цэг |

---

## D. Empirical Evidence Summary (Phase 2-д LaTeX хүснэгт болгон оруулах material)

**Эх үүсвэр:** `merchant_portal/bench-results/bulk-publish/summary-2026-04-17T03-31-15.csv` (canonical run, 5 iterations per scenario, N ∈ {10, 50, 100, 500})

### D.1. Bulk publish stratеги харьцуулалт (mean ms, доод нь сайн)

| N | sequential_rpc | parallel_rpc | batched_update | bulk_rpc |
|---|---:|---:|---:|---:|
| 10 | 31.05 | 11.20 | 5.49 | **3.49** |
| 50 | 111.8 | 81.09 | 7.99 | **5.81** |
| 100 | 275.2 | 96.01 | 12.22 | **8.48** |
| 500 | 1041.3 | 516.9 | 2.39 ⚠ | **23.05** ✅ |

### D.2. Tail latency (p95 / p99) хам gold dataset (N=500)

| Strategy | p50 ms | p95 ms | p99 ms | success | error |
|---|---:|---:|---:|---:|---:|
| **bulk_rpc** | 22.02 | 25.44 | **25.52** | 2500/2500 (100%) | 0 |
| sequential_rpc | 986.2 | 1175 | 1180 | 2500/2500 (100%) | 0 |
| parallel_rpc | 515.7 | 555.2 | 557.1 | **1407/2500 (56%)** | 1093 |
| batched_update | 2.21 | 4.17 | 4.46 | **0/2500 (0%)** | 2500 |

### D.3. Гол түүх (Phase 2 narrative-д ашиглах)

1. **`bulk_rpc` (нэг `UPDATE ... ANY(ids)` SECURITY DEFINER RPC) нь scaling-д хамгийн зохистой:** N=500 хүртэл linear sub-millisecond per-task (per_task_ms 0.046 at N=500), p99 25 ms тогтвортой, 100% success.
2. **`sequential_rpc` (N бие даасан HTTP round-trip) нь линейн өснө:** N=500 үед 1041 ms (45× удаан bulk-ээс), бүгд success.
3. **`parallel_rpc` (N зэрэг RPC) нь high concurrency-д тогтворгүй:** N=500 үед 44% failure rate (1093 error) — Supabase pooler concurrency limit рүү толгой цохив.
4. **`batched_update` (PostgREST `.in('id', ids)` bulk update) нь N=500 үед бүхэлдээ failed (2500/2500 error)** — REST endpoint-ын payload size limit/timeout магадгүй.

→ **Эмпирик дүгнэлт:** RPC-д суурилсан single-statement bulk операци нь PostgREST-ийн `.in()`-аас илүү зохистой, parallel client-side fanout-аас илүү тогтвортой. Энэ нь thesis-д "bulk publish архитектурын нэг сайн жишээ" гэж бичих empirical material.

### D.4. Driver benchmark / security-test status

| Test suite | Code ready? | Run хийгдсэн? | Result file? |
|---|:---:|:---:|---|
| `driver/benchmarks/benchmark_get_available_tasks.ts` (RLS overhead) | ✅ | ❌ | none |
| `driver/benchmarks/load_test/claim_race.k6.js` (concurrent claim) | ✅ | ❌ | none |
| `driver/security-tests/brute_force.ts` (OTP brute-force scenarios A/B/C) | ✅ | ❌ | none |
| `driver/analysis/analyze_results.py` (matplotlib reports) | ✅ | ❌ | none |

→ **Status:** Driver талын empirical evidence бэлэн биш. Phase 2-д сонголт:
  - **(a)** "Code is implemented; full-scale execution requires local Supabase setup, scheduled for evaluation post-defense" гэж шударга хүлээн зөвшөөрөх (Хязгаарлалт хэсэгт)
  - **(b)** Phase 1.5 хийж local Supabase дээр script-уудыг run хийсний дараа Phase 2-т empirical тоо нэмэх

---

## E. Action Items for Phase 2

### MUST FIX (хамгаалалтад халдах)

1. **Chapter3-ийн `\iffalse...\fi` блокийг сэргээх** — load test result subsection (`subsec:scalability`) болон blast-radius/limitations-ийг идэвхжүүлэх. Гэхдээ:
   - "150 concurrent" claim-ийг тоон evidence-гүй учир **зөөлрүүлж бичих** ("одоогийн архитектурын төлөвлөсөн capacity ceiling, full validation pending") эсвэл driver bench run хийсний дараа real тоог оруулах
   - `tab:load_test` фейк хүснэгтийг устгах эсвэл "(estimated)" footnote нэмэх
2. **Chapter1 line 27 "150 concurrent... load test-ээр шалгасан" claim-ийг** хатуу засах: эсвэл merchant bulk_publish bench result-аар орлуулах ("bulk publish throughput up to N=500 measured at p99 = 25 ms"), эсвэл "planned validation" гэж хэллэгдэх
3. **`task_events` / `status_history` / "immutable audit log" claim-уудыг** Хүснэгт 2 (Chapter2 `tab:comparative_analysis`) болон Хүснэгт `tab:unified_comparison` (Chapter4)-ээс **бүрэн устгах** эсвэл "status timestamp columns (`assigned_at`, `delivered_at`, `completed_at`) — full event-sourced audit log: future work" гэж тодруулах. Chapter4 line 55-д аль хэдийн зөв wording бий — Chapter2-д тэр wording-ыг ашиглах
4. **"Marketplace-based динамик хуваарилалт"-ыг "FCFS claim with concurrency protection (FOR UPDATE SKIP LOCKED)" болгож rename хийх** — Chapter2 Хүснэгт 2 line 324, Chapter4 Хүснэгт line 49 хоёрт
5. **Chapter5 эсвэл Appendix эсвэл Bibliography main.tex-д идэвхжүүлэх** — bibliography ялангуяа critical (academic standard). `\printbibliography` line uncomment хийх
6. **Broken `\ref` 5+-ыг засах** — Chapter1 §1.2 line 27, Chapter3 line 193, line 595 — эсвэл reference-ийг устгах, эсвэл target section-ыг нээх
7. **CI-аар schema drift prevention claim-ыг (Chapter3 §3.2.3) merchant-д л хязгаарлах** — driver-т `database.ts` бэлэн биш, GitHub Actions огт байхгүй

### SHOULD FIX (итгэмжлэгдэх байдал)

8. **Хүснэгт 4-д `generate_epod_otp` (driver), `resend_epod_otp`, `create_delivery_task`, `publish_delivery_tasks_bulk` нэмэх** — нэр зөрөлтийг тайлбарлах footnote
9. **Хүснэгт 5-д `current_org_id()` нэртэй helper-ыг ашиглах** (`get_user_org_id`-ыг update хийх)
10. **Хүснэгт 6 (Технологийн стек)-ыг expand хийх:**
    - "React Hook Form + Zod" нь зөвхөн driver — merchant-т `Form.useForm` (antd) ашигладаг гэж row нэмэх
    - NativeWind (driver), Recharts (merchant), Nodemailer (merchant + driver edge function), dayjs (merchant), react-native-maps (driver, library only) бүгдийг нэмэх
    - Tailwind version зөрүү (merchant v4, driver v3) тэмдэглэх
11. **Зураг 15 ERD-д `courier_earnings`, `delivery_epod_otps`, `task_items`, `order_items`, `org_settings`, `products` хүснэгтүүдийг нэмэх** (TikZ figure-д 6 entity нэмж дэлгэрүүлэх). "9 үндсэн" → "14+ хүснэгт + 1 view"
12. **`subsubsection{available_tasks VIEW}` (одоо commented)-ыг нээх** — view нь хүснэгт биш гэдгийг тодруулах
13. **Триггерийн архитектур subsection-ыг сэргээх** — `auto_send_epod_on_delivered`, `auto_insert_courier_earnings`, `trg_enforce_delivery_task_transition`, `trg_payment_confirmed`, `trg_merchant_courier_immutable`, `trg_delivery_fee_change`, `on_kyc_status_change` зэрэг 7+ trigger үнэхээр кодод бий — thesis-д тусгасан нь нийцтэй (Chapter3-д commented section-ыг нээх)
14. **Bench evidence-ийг 3-р бүлгийн шинэ subsection болгон оруулах** (`\subsection{Гүйцэтгэлийн туршилт ба bulk publish throughput}`):
    - Хүснэгт `tab:bulk_publish_summary` — D.1, D.2 хүснэгтүүд
    - 1-2 догол commentary
    - LaTeX command `\input{Diplom_bichver/results/bulk_publish_summary.tex}` хэлбэр зөвлөмөл
15. **driver/security-test, driver/load_test ажлын статусыг "Хязгаарлалт"-д бичих** — code ready, run pending due to environment setup

### NICE TO HAVE

16. **Chapter6.aux / Chapter7.aux устгах** (хуучин build artifact)
17. **`Latex/`, `DIPLOMA_THESIS.md`-ийг `.gitignore`-т оруулах** (canonical биш)
18. **`Diplom_bichver/CHANGES.md` файл үүсгэж Phase 2 өөрчлөлтийг логлох**
19. **`TittlePage.tex`, `Abberaviation.tex` typo-уудыг засах** (`TitlePage`, `Abbreviation`)
20. **Driver migration discipline сайжруулах** — `FIX_*.sql`, `DEBUG_*.sql`, `nuclear_*` файлуудыг archived folder руу шилжүүлэх (codebase task — thesis-д хамаарахгүй)
21. **"Resend API" wording (Chapter5 — excluded) → nodemailer Gmail SMTP** ирээдүйд canonical conclusion-аар Chapter5-ыг сэргээхэд

### Phase 1.5 (бүтээлд ашиглахын тулд optional)

22. **Driver bench/security script-уудыг local Supabase дээр run хийх** (60 минут орчим setup) — Phase 2-д extra empirical row нэмэх боломж
23. **Bench-ийн pip script болон D-table CSV-ээс LaTeX `\input{}`-ийг auto-generate хийх Python helper бичих**

---

## Дүгнэлт (Phase 1 summary)

- **Bench evidence бэлэн (merchant bulk publish):** Phase 2-д шууд оруулах боломжтой, түүхий тоо CSV-д бий
- **8 critical finding (B хэсэг)** — гол overclaim ба broken reference. Энэ нь хамгаалалтад уулзаж болзошгүй
- **14 minor inconsistency (C хэсэг)** — нэр зөрөлт, version mismatch, schema duplicate
- **~20 action item (E хэсэг)** — 7 MUST FIX, 8 SHOULD FIX, 6 NICE-TO-HAVE
- Driver-ын benchmark/security-test code бэлэн боловч empirical run хийгдээгүй — энэ нь Phase 2-д хязгаарлалт болон цаашдын ажил хэлбэрээр шударгаар зориулах боломжтой
- Chapter5, AppendixA, bibliography PDF-д ороогүй — энэ нь хамгаалалтын дагуу заавал засах ёстой
- Chapter3-ийн 17% (10 subsection) `\iffalse`-аар хаагдсан — nice content (architecture deep-dives, load test setup) нь thesis-ийг үнэлгээгүй болгосон

**Phase 2 руу шилжих зөвшөөрлийг хүлээж байна.**
