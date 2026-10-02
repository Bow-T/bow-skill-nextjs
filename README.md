# bow-skill-nextjs

Bộ skill cho công nghệ **Next.js (App Router) + React**, dùng với [bow-agent](https://github.com/Bow-T/bow-agent).
Theo hướng **mỗi công nghệ một repo**: bộ này chỉ lo Next.js / React; dự án có Supabase thì tick thêm bộ
[bow-skill-supabase](https://github.com/Bow-T/bow-skill-supabase) ở ô **Công nghệ**.

## Cấu trúc

```
bow-skill-nextjs/
├── bow-skill.json        # manifest: id "nextjs" + quy tắc tự nhận diện repo + `needs`
├── THIRD_PARTY.md        # skill nào chép từ đâu, commit nào, giấy phép gì, đã sửa gì
└── skills/
    ├── nextjs-convention/            # team — ranh giới Server ⟷ Client Component, cấu trúc app/
    ├── nextjs-form/                  # team — form: required ↔ dấu * ↔ rule một nguồn, validate ở biên
    ├── nextjs-data/                  # team — lớp data Supabase qua server      (cần bộ `supabase`)
    ├── nextjs-realtime/              # team — realtime Supabase phía client     (cần bộ `supabase`)
    ├── vercel-react-best-practices/  # ⬇ Vercel — 70 quy tắc hiệu năng React / Next.js
    └── vercel-composition-patterns/  # ⬇ Vercel — 8 quy tắc composition cho component React
```

| Skill | Dùng khi | Nguồn |
| --- | --- | --- |
| `nextjs-convention` | tạo page / layout / route / component mới; soát ranh giới Server ⟷ Client | team Bow |
| `nextjs-form` | thêm / sửa form, field, modal, input upload / phone / email | team Bow |
| `nextjs-data` | thêm / sửa truy vấn hoặc mutation Supabase (Server Component, Server Action) | team Bow · cần `supabase` |
| `nextjs-realtime` | thêm / sửa realtime subscription, live badge / count | team Bow · cần `supabase` |
| `vercel-react-best-practices` | viết / review / refactor code React, Next.js về hiệu năng: waterfall, bundle, server, re-render | Vercel (MIT theo frontmatter) |
| `vercel-composition-patterns` | refactor component nhiều boolean prop, thiết kế API component, compound component | Vercel (MIT theo frontmatter) |

Bốn skill của team giữ nguyên từ v1. Hai skill của Vercel chép nguyên văn từ
[vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) — commit, giấy phép và phần đã bỏ
ghi ở [`THIRD_PARTY.md`](THIRD_PARTY.md). Repo này không có thư mục `LICENSES/` vì nguồn không phát hành
file giấy phép (chỉ ghi `license: MIT` trong frontmatter).

Cố ý KHÔNG lấy các skill khác của nguồn: `deploy-to-vercel`, `vercel-cli-with-tokens`, `vercel-optimize`
(có script / thao tác triển khai), `react-native-skills` (thuộc bộ React Native), `react-view-transitions`,
`web-design-guidelines`, `writing-guidelines`.

Hai skill của Vercel là hướng dẫn chung cho React / Next.js, không viết riêng cho team: ví dụ của
`vercel-composition-patterns` dùng component kiểu React Native (`TextInput`, `onPress`) và có một mục chỉ
áp dụng cho React 19+; vài quy tắc của `vercel-react-best-practices` gợi ý thư viện ngoài (`better-all`,
`lru-cache`, `swr`). **Khi hai nguồn nói khác nhau, quy ước team thắng.**

## `needs` — skill chỉ trải khi có đủ bộ khác

```json
"needs": { "nextjs-data": ["supabase"], "nextjs-realtime": ["supabase"] }
```

Skill có tên trong `needs` chỉ được trải vào `.claude/skills/` khi dự án tick **CẢ** các bộ ghi ở đó.
Dự án chỉ tick `nextjs` nhận 4 skill (`nextjs-convention`, `nextjs-form` và 2 skill Vercel); tick thêm
`supabase` thì có thêm `nextjs-data` và `nextjs-realtime`. Nhờ vậy app Next.js gọi backend riêng không
bị nạp hướng dẫn Supabase.

## Tự nhận diện

bow-agent tự tick bộ này cho dự án chưa khai công nghệ khi repo khớp quy tắc ở `targets.detect`:

| Quy tắc | Nghĩa |
| --- | --- |
| `package.json` chứa `"next"` | app Next.js (tìm cả `package.json` trong thư mục con, vd `apps/web/package.json`) |

Chuỗi tìm là `"next"` **có cả dấu nháy kép**, tức đúng tên gói `next` trong `dependencies`; nhờ đó không
khớp nhầm `next-auth`, `next-themes`, `eslint-config-next`… Chỉ soi file git đã track.

## Đổi gì so với v1

- `id` cũ là `nextjs-supabase` ("Next.js + Supabase", v1.1.0). v2 đổi thành `nextjs` ("Next.js") — mục
  registry dùng id cũ phải thay bằng mục ở dưới.
- Bỏ `store-status-lifecycle` (skill riêng của một sản phẩm).
- Thêm 2 skill của Vercel: `vercel-react-best-practices`, `vercel-composition-patterns`.
- Skill Supabase dùng chung (migration, RLS, soát bảo mật…) nằm ở
  [bow-skill-supabase](https://github.com/Bow-T/bow-skill-supabase); ở đây `nextjs-data` và
  `nextjs-realtime` chỉ trải khi dự án tick cả `supabase` (xem `needs`).
- Quy tắc nhận diện chỉ còn `package.json` chứa `"next"` (v1: chuỗi `next` không nháy, hoặc có thư mục `app`).

## Đăng ký vào bow-agent

Thêm vào `stacks` của `~/.bow-agent/registry.json` (bow chỉ tải repo có trong danh sách này, đúng tag đã ghim):

```json
{ "id": "nextjs", "label": "Next.js", "repo": "github.com/Bow-T/bow-skill-nextjs", "ref": "v2.0.0" }
```

Muốn có `nextjs-data` / `nextjs-realtime` thì registry phải có cả mục `supabase` và dự án tick cả hai bộ.

Thử local trước khi đẩy GitHub: `"repo": "file:///đường/dẫn/tới/bow-skill-nextjs"`.

## Phát hành phiên bản mới

Sửa skill qua PR → tăng `version` trong `bow-skill.json` → gắn tag (`git tag v2.0.1 && git push --tags`)
→ cập nhật `ref` trong registry. bow tải theo tag và cache theo `<id>@<ref>`, nên sửa nội dung mà giữ
nguyên tag sẽ KHÔNG được tải lại. Cập nhật skill từ nguồn bên ngoài: xem cuối `THIRD_PARTY.md`.
