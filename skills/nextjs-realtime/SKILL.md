---
name: nextjs-realtime
description: Wire Supabase realtime trong Next.js phía client — subscribe trong một custom hook của Client Component, giữ channel trong ref, LUÔN removeChannel khi unmount, áp CÙNG filter mà server query dùng, và re-render khi row đổi. Dùng khi thêm/sửa realtime subscription, live badge/count, hay UI phải cập nhật khi DB đổi trong app Next.js. Đi cặp với [[nextjs-data]] và [[nextjs-convention]]. Bản Next.js của [[flutter-realtime]]/[[rn-realtime]].
---

# Next.js Realtime — client-only hook, LUÔN dọn dẹp

Mục tiêu: realtime cập nhật live mà không leak channel, không badge drift. Realtime
**chỉ chạy ở client** — phải là Client Component (`'use client'`), subscribe trong
hook, dọn khi unmount.

## 1. Subscription chuẩn — hook trong Client Component
```tsx
'use client';
import { useEffect, useRef, useState } from 'react';
import { createBrowserClient } from '@/lib/supabase/client';

export function useDeliveryRealtime(bookingId: string, initial: Delivery) {
  const [row, setRow] = useState(initial);
  const channelRef = useRef<RealtimeChannel | null>(null);

  useEffect(() => {
    const supabase = createBrowserClient();          // anon client, ràng buộc RLS
    const channel = supabase
      .channel(`delivery_${bookingId}`)              // tên unique, scoped
      .on('postgres_changes',
        { event: 'UPDATE', schema: 'public', table: 'deliveries',
          filter: `booking_id=eq.${bookingId}` },    // CÙNG filter với query server
        (payload) => setRow(payload.new as Delivery))
      .subscribe();
    channelRef.current = channel;

    return () => { supabase.removeChannel(channel); }; // LUÔN dọn khi unmount
  }, [bookingId]);

  return row;
}
```

## 2. Bug class phải tránh
1. **Quên `'use client'`** — realtime không chạy trên server, subscription câm lặng.
2. **Leak channel** — thiếu `removeChannel` trong cleanup.
3. **Filter lệch** — subscription khác filter server query → nhận nhầm row.
4. **Bỏ qua initial từ server** — nên nhận `initial` (fetch ở Server Component) rồi
   realtime chỉ cập nhật delta, tránh chớp nháy loading.

## 3. Checklist
- [ ] Hook nằm trong Client Component (`'use client'`).
- [ ] `removeChannel` trong return của `useEffect`.
- [ ] `filter` khớp query server (xem [[nextjs-data]]).
- [ ] Nhận initial từ server, realtime chỉ cập nhật delta.
