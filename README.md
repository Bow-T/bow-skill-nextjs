# bow-skill-nextjs

Bộ **skill dùng chung** cho stack **Next.js (App Router) + Supabase** của team Bow,
dùng với [bow-agent](https://github.com/Bow-T/bow-agent). Đồng bộ Supabase với
[bow-skill-flutter](https://github.com/Bow-T/bow-skill-flutter) và
[bow-skill-react-native](https://github.com/Bow-T/bow-skill-react-native).

## Repo này chứa gì

```
bow-skill-nextjs/
├── bow-skill.json                  # manifest: id stack, version, danh sách skill, targets
├── skills/
│   ├── nextjs-data/SKILL.md        # lớp data server-first, RLS, không lộ service_role
│   ├── nextjs-convention/SKILL.md  # ranh giới Server/Client Component, cấu trúc app/
│   └── nextjs-realtime/SKILL.md    # realtime client-only, LUÔN dọn channel
└── README.md
```

bow-agent tải repo này (ghim theo tag `v1.0.0`), rồi **trải** các folder trong
`skills/` vào `.claude/skills/` của repo đích để Claude dùng qua tool `Skill`.

## Phát hành phiên bản mới

Sửa skill → tăng `version` trong `bow-skill.json` → gắn tag:
```bash
git tag v1.0.1 && git push --tags
```
Rồi báo admin cập nhật `ref` trong registry của bow-agent (bow luôn tải theo tag).
