---
name: nextjs-form
description: Form trong Next.js + React — required↔asterisk↔rule phải chung MỘT nguồn, validate ở BIÊN bằng validator dùng chung (email/phone/password, không tự viết regex), upload file validate type+size ngay lúc pick VÀ drop (không tin `accept=`), chặn submit khi form rỗng, và create/edit/onboarding cùng entity dùng chung một validator. Dùng khi thêm/sửa form, field, modal, hoặc input upload/phone/email trong app Next.js, hoặc khi QC báo "thiếu dấu *, nhận phone/email sai, submit được form trống, upload không chặn type/size, ảnh không preview". Đi cặp với [[nextjs-convention]] và [[nextjs-data]] (validate ownership/tiền ở server, không chỉ ở form).
---

# Next.js Form — validate ở biên, required↔asterisk↔rule một nguồn

Lỗi form web hay lọt qua `tsc` + unit test vì chúng là **hành vi thiếu**, không phải
type sai: field required không có dấu `*`, upload tin mỗi `accept=`, submit chạy trên
form rỗng. Đây là lớp bug review được bằng mắt, không cần thiết bị QC.

## 1. required → asterisk → rule: khai MỘT nguồn, đừng để lệch
- **AntD `<Form>`** (khuyến nghị cho form phức tạp): khai `rules:[{ required: true, message }]`
  → dấu `*` và cổng submit là **structural** (AntD tự render `*`, `onFinish` chỉ chạy khi
  mọi rule pass). Đừng gắn `*` thủ công trên label — để rule sinh ra nó.
```tsx
<Form form={form} layout="vertical" onFinish={onSubmit}>
  <Form.Item name="name" label="Full name" rules={[{ required: true, message: 'Bắt buộc' }]}>
    <Input />
  </Form.Item>
</Form>
// submit: <Button onClick={() => form.submit()} />  // onFinish không chạy nếu rule fail
```
- **Form thủ công** (`useState` + `errors`): required-ness, dấu `*`, và validator sống ở
  ba chỗ khác nhau nên **drift**. Buộc chúng về một nguồn: field nào có trong `validateX()`
  thì label đọc `*` từ đúng nguồn đó; sửa một chỗ, grep tên field ở cả JSX lẫn validator.

## 2. Validate ở BIÊN bằng validator dùng chung — đừng tự viết regex
```ts
// lib/validation.ts — dùng lại ở MỌI flow (sign-up, reset, change, edit-profile)
import { isValidPhoneNumber } from 'libphonenumber-js';
export const isEmail = (s: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s) && s.length <= 254;
export const isPhone = (e164: string) => isValidPhoneNumber(e164); // số quá ngắn cũng fail
```
- Email/phone/password đi qua helper chung; `includes('@')` là nhận rác.
- **Country code mặc định đúng thị trường** (vd `+33`), đừng để lib mặc định `+1`; picker
  cho tìm kiếm.
- Password rules **giống nhau** ở Sign-up / Reset / Change — khác nhau nghĩa là bạn đã
  inline thay vì import chung.

## 3. Upload file: validate type + size lúc pick VÀ drop
`accept=` chỉ là gợi ý — kéo-thả (drag-drop) bỏ qua nó hoàn toàn.
```ts
const MAX = 10 * 1024 * 1024;
const OK = ['image/jpeg', 'image/png', 'image/webp'];
export function validateUpload(f: File): string | null {
  if (!OK.includes(f.type)) return 'Định dạng không hỗ trợ';
  if (f.size > MAX) return 'File quá lớn';
  return null;
}
// gọi ở CẢ nút pick lẫn onDrop, TRƯỚC khi preview; preview bản vừa chọn & giữ tới persist
```

## 4. Gate submit bằng một `validateX()` thuần — chặn form rỗng
```ts
function validateProduct(f: ProductForm): Errors {
  return { name: f.name.trim() ? undefined : 'Bắt buộc', price: f.price > 0 ? undefined : 'Bắt buộc' };
}
const errors = submitted ? validateProduct(form) : {};
const isValid = Object.values(errors).every(v => !v);
function handleSave() { setSubmitted(true); if (!isValid) return; persist(); } // click đầu chặn form rỗng
// <Button disabled={(submitted && !isValid) || saving} />
```
Một hàm thuần phủ **mọi** field required (dễ unit-test); handler chạy nó và dừng nếu có
lỗi. Đường "lưu nháp" có thể bỏ qua validate — nhưng phải là lựa chọn có chủ đích.

## 5. create ↔ edit ↔ onboarding cùng entity → chung MỘT validator
Cùng một entity (store profile, product) được nhập ở wizard onboarding, form tạo, modal
sửa. Ba bản rule sẽ drift; edit nhận cái create chặn → QC log bản đi sau. Tách validator
một lần, gọi từ cả ba (giống nguyên tắc [[nextjs-convention]]: một nguồn, tái sử dụng).

## Checklist
- [ ] Dấu `*` và validate của một field cùng MỘT nguồn (AntD: `rules:[{required}]`).
- [ ] Email/phone/password dùng validator chung, không regex mới; phone mặc định đúng nước.
- [ ] Mọi lối upload (pick **và** drop) validate type+size trước preview; preview & giữ tới persist.
- [ ] Submit chạy một `validateX()` phủ hết field required; form rỗng KHÔNG persist được.
- [ ] create/edit/onboarding cùng entity dùng chung một validator (không fork).
- [ ] Tiền/quyền vẫn validate server-side ([[nextjs-data]]), không chỉ ở form.
