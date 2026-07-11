---
name: nextjs-convention
description: Quy ước code Next.js App Router của team Bow — ranh giới Server vs Client Component ('use client' chỉ khi cần), cấu trúc thư mục app/, đặt tên route/component, và tách UI khỏi lớp data. Dùng khi tạo page/layout/route/component mới hoặc review xem code có theo quy ước không. Đi cặp với [[nextjs-data]] và [[nextjs-realtime]].
---

# Next.js Convention — Server-first, ranh giới rõ ràng

Mục tiêu: mọi route Next.js theo cùng khung App Router, mặc định Server Component,
chỉ 'use client' khi thật sự cần tương tác.

## 1. Cấu trúc thư mục (App Router)
```
app/
  <route>/
    page.tsx        # Server Component mặc định
    layout.tsx      # layout dùng chung
    loading.tsx     # skeleton
    actions.ts      # 'use server' — Server Actions của route
components/          # component tái dùng (server hoặc client)
lib/
  supabase/         # server.ts + client.ts (2 loại client, xem nextjs-data)
```

## 2. Ranh giới Server ⟷ Client (quan trọng nhất)
- **Mặc định Server Component.** Chỉ thêm `'use client'` khi cần state/effect/
  event handler/realtime. Đừng 'use client' cả cây chỉ vì 1 nút bấm.
- Fetch dữ liệu ở Server Component (xem [[nextjs-data]]); truyền model đã map
  xuống Client Component qua props.
- Realtime là việc của client — tách vào hook riêng, xem [[nextjs-realtime]].

## 3. Đặt tên
- Component: `PascalCase`. Hook: `use` + camelCase. Route folder: kebab-case.
- Server Action: động từ, trong `actions.ts` của route (`updateDelivery`).

## 4. Checklist review
- [ ] Component không cần tương tác thì KHÔNG có `'use client'`.
- [ ] Fetch ở Server Component, không fetch trong `useEffect` nếu render được ở server.
- [ ] Không prop-drill quá 2 tầng — tách context/store khi cần.
