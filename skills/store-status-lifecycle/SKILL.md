---
name: store-status-lifecycle
description: Vòng đời status của một entity (store/business: active / paused / suspended / reactivated / removed) khi state KHÔNG lưu thành enum mà được SUY RA từ nhiều cột timestamp — phải suy ra GIỐNG NHAU và re-sync trên MỌI surface đọc nó: RLS visibility (DB), admin list/filter/detail/audit, banner phía chủ (partner), và luồng fulfilment. Dùng khi thêm/đổi một giá trị status hoặc thứ tự ưu tiên, một path suspend/pause/reactivate/remove, một surface hiển thị "tạm ngưng/không khả dụng", hoặc khi QC báo "status sai/cũ ở surface này", "không refresh sau reactivate", "mất lịch sử suspend". Đi cặp với [[nextjs-data]] (đọc đúng nguồn sự thật), [[nextjs-realtime]] (re-sync khi state đổi) và [[nextjs-convention]].
---

# Status lifecycle — một state, nhiều surface, MỘT cách suy ra

State "đang active/paused/suspended" của một entity thường **không** là một cột enum
grep-được. Nó **suy ra** từ các cột timestamp (`paused_at`, `suspended_at`,
`suspension_lifted_at`, `removed_at`, `is_active`), và mỗi surface tự suy ra bằng ngôn
ngữ của nó → drift. Đổi vòng đời mà không sweep hết, `tsc`/test vẫn xanh còn QC log
từng surface một.

## 1. "Đang suspended?" là MỘT predicate — gọi chung, đừng null-check thô
`suspended_at IS NOT NULL` là **sai**: một lần suspend đã gỡ vẫn giữ `suspended_at` để
lưu lịch sử, nên phép thử "đang suspended" phải so với `suspension_lifted_at`.
```sql
-- một predicate duy nhất, mọi reader SQL/view gọi nó
create function store_suspension_in_force(suspended_at timestamptz, lifted_at timestamptz)
returns boolean language sql immutable as $$
  select suspended_at is not null and (lifted_at is null or lifted_at < suspended_at)
$$;
```
- Thứ tự ưu tiên `removed > suspended > paused > active` khai **một chỗ** (view/CASE),
  các nơi khác đọc lại — đừng chép logic precedence.
- Mọi mirror (admin TS, partner TS) phải khớp predicate này, không tự viết `!= null`.

## 2. Một status flip phải tới MỌI reader
Cùng một state được đọc ở nhiều surface; đổi vòng đời mà không sweep hết là rơi ở
surface bạn không mở. Coi đây là thay đổi cross-cutting: grep một peer đã wired để liệt
kê mọi nơi.
- **RLS/DB** — khách chỉ thấy khi `is_active AND removed_at IS NULL AND NOT store_suspension_in_force(...)`.
- **Admin** — vocabulary + option filter (đủ cả "paused") + tag status + drawer audit.
- **Partner/owner** — banner suy ra state, load lại đúng lúc (login, đổi tab).

## 3. Visibility ≠ fulfilment — suspend đừng chặn đơn ĐÃ đặt
Ẩn store khỏi khách (browse) là câu hỏi khác với chặn luồng của một đơn **đã tạo**.
- Đặt filter status ở nhánh **khách duyệt** (RLS, list, tạo đơn mới), **không** ở nhánh
  xử lý một đơn đang chạy (dispatch/nhận đơn). Suspend một store không được làm kẹt đơn
  đã ở trạng thái chờ giao.
- Khi thêm một check suspend, hỏi: "khách duyệt, hay đơn đang chạy?" — chỉ cái đầu filter.

## 4. Re-sync khi state đổi live
Một row đọc-một-lần rồi cache sẽ cũ. Surface nào gate hành động theo availability phải
đọc lại state khi nó có thể đổi dưới chân người dùng — subscribe row (lọc đúng như query
server) hoặc re-fetch khi focus. Xem [[nextjs-realtime]] (client-only, LUÔN dọn channel).

## 5. Reactivate PHẢI giữ lịch sử — đừng null cột đã set
Chiều ngược không phải "xóa timestamp".
- Thêm cột *lifted/reactivated* (vd `suspension_lifted_at`, `reactivated_at`) thay vì
  null cột *entered* (`suspended_at`); giữ `last_paused_at`/`last_pause_reason` để admin
  xem lịch sử.
- Surface hiển thị lịch sử đọc đúng cột preserved, không đọc cờ live. Backfill đủ cặp cột
  (có `reviewed_by` thì có `reviewed_at`).

## Checklist
- [ ] "Đang suspended?" dùng chung một predicate ở mọi surface — không `suspended_at IS NOT NULL` thô.
- [ ] State/precedence mới sweep đủ: RLS/DB, admin vocab+filter+detail, banner partner.
- [ ] Filter status chỉ ở nhánh khách-duyệt, không ở nhánh đơn đang chạy.
- [ ] Surface gate theo availability re-sync khi flip live ([[nextjs-realtime]]).
- [ ] Reactivate giữ cột lịch sử; audit đọc cột preserved; backfill đủ cặp cột.
- [ ] Verify runtime end-to-end (suspend → khách ẩn, đơn cũ vẫn chạy, admin/partner phản ánh,
      reactivate khôi phục + giữ lịch sử), không chỉ tsc/test xanh.
