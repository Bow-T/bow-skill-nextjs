---
name: nextjs-data
description: Lớp data Supabase cho Next.js App Router — đọc dữ liệu trong Server Components / Route Handlers bằng server client, mutation qua Server Actions, KHÔNG lộ service_role ra client, tôn trọng RLS và không tin dữ liệu tiền/quyền từ client. Dùng khi thêm/sửa truy vấn hoặc mutation Supabase trong app Next.js. Đi cặp với [[nextjs-convention]] và [[nextjs-realtime]]. Bản Next.js của lớp data Flutter/RN.
---

# Next.js Supabase Data — server-first, RLS-aware, không lộ secret

Mục tiêu: mọi truy cập Supabase đặt đúng tầng (server vs client), không rò
`service_role` ra browser, và không tin dữ liệu nhạy cảm từ client.

## 1. Đọc trong Server Component (mặc định)
```ts
// app/deliveries/[id]/page.tsx — Server Component
import { createServerClient } from '@/lib/supabase/server';

export default async function Page({ params }: { params: { id: string } }) {
  const supabase = createServerClient();          // client server-side, dùng cookie/RLS
  const { data, error } = await supabase
    .from('deliveries')
    .select('*')
    .eq('booking_id', params.id)                  // filter khớp subscription realtime
    .single();
  if (error) throw error;
  return <DeliveryView delivery={mapDelivery(data)} />;
}
```

## 2. Mutation qua Server Action
```ts
'use server';
export async function updateDelivery(id: string, patch: DeliveryPatch) {
  const supabase = createServerClient();
  // Ownership + số tiền: xác thực SERVER-SIDE (RLS WITH CHECK / RPC), không tin client.
  const { error } = await supabase.from('deliveries').update(patch).eq('id', id);
  if (error) throw error;
  revalidatePath(`/deliveries/${id}`);
}
```

## 3. Quy tắc tuyệt đối
- **KHÔNG** dùng `service_role` key trong code chạy ở client. Chỉ trong server
  (Route Handler / Server Action / Server Component), qua biến môi trường server.
- Đọc → ưu tiên Server Component; mutation → Server Action / Route Handler.
- Client Component chỉ dùng anon client cho realtime/UI, mọi thứ ràng buộc RLS.
- Số tiền/giảm giá/ownership: xác thực server-side, không ghi thẳng từ client.

## 4. Checklist
- [ ] Không có `service_role` trong bundle client.
- [ ] Đọc ở Server Component, mutation ở Server Action/Route Handler.
- [ ] Dữ liệu nhạy cảm xác thực server-side (RLS/RPC).
- [ ] Row map sang model trước khi render.
