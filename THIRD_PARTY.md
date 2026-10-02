# Nguồn bên ngoài

Skill chép từ repo khác được ghi ở đây: lấy từ đâu, tại commit nào, giấy phép gì, và đã sửa gì.
Bản chép là **ảnh chụp tại một commit** — bow-agent không tự kéo bản mới của nguồn. Muốn cập nhật
thì làm lại các bước ở cuối file, đọc lại nội dung, rồi mới gắn tag.

| Skill | Nguồn | Commit | Giấy phép | Đã sửa |
| --- | --- | --- | --- | --- |
| `vercel-react-best-practices` | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) `skills/react-best-practices` (v1.0.0) | `063bee94c3f4df8453406c830b0a7df0f2860278` (2026-08-28) | MIT — theo frontmatter của skill | Đổi tên thư mục cho trùng `name`; bỏ `README.md`, `rules/_template.md`, `AGENTS.md` và mục "Full Compiled Document" cuối `SKILL.md` trỏ tới nó; nội dung còn lại nguyên văn |
| `vercel-composition-patterns` | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) `skills/composition-patterns` (v1.0.0) | `063bee94c3f4df8453406c830b0a7df0f2860278` (2026-08-28) | MIT — theo frontmatter của skill | Đổi tên thư mục cho trùng `name`; bỏ `README.md`, `rules/_template.md`, `AGENTS.md` và mục "Full Compiled Document" cuối `SKILL.md` trỏ tới nó; nội dung còn lại nguyên văn |

Phiên bản trong ngoặc là `metadata.version` ở frontmatter của từng `SKILL.md`.

## Giấy phép

Repo nguồn **không có file `LICENSE`** (tại commit trên không có file nào tên `LICENSE*` / `COPYING*` /
`NOTICE*`, `package.json` cũng không có trường `license`). Căn cứ duy nhất là:

- frontmatter của từng `SKILL.md` ghi `license: MIT`;
- `README.md` ở gốc repo nguồn có mục `## License` với đúng một chữ `MIT`.

Vì nguồn không phát hành văn bản giấy phép hay dòng bản quyền nào, repo này **không có** thư mục
`LICENSES/` — không tự viết ra văn bản giấy phép thay cho nguồn. Giấy phép của hai skill là
"giấy phép theo frontmatter của skill". `README.md` của `react-best-practices` (đã bỏ, xem dưới) ghi
công: "Originally created by @shuding at Vercel".

## Vì sao bỏ những phần đó

- **`README.md` của từng skill** là hướng dẫn cho người soạn skill ở nguồn: cách thêm quy tắc mới và
  dựng lại `AGENTS.md` bằng `pnpm install` / `pnpm build` / `pnpm validate` — các lệnh đó cần thư mục
  `packages/react-best-practices-build` của repo nguồn, không có ở đây. `SKILL.md` không trỏ tới file này.
- **`rules/_template.md`** là khuôn để viết quy tắc mới, chỉ dành cho người soạn skill.
- **Đổi tên thư mục**: ở nguồn thư mục là `react-best-practices` / `composition-patterns` còn `name`
  trong frontmatter là `vercel-react-best-practices` / `vercel-composition-patterns`. bow-agent trải skill
  theo tên thư mục nên đặt thư mục trùng `name`. Không sửa chữ nào bên trong.

- **`AGENTS.md`** là bản gộp sinh tự động từ `rules/` (108 KB và 23 KB — riêng bản React khoảng 27k
  token). `SKILL.md` trỏ tới nó như "tài liệu đầy đủ", agent làm theo sẽ nạp cả file vào ngữ cảnh trong
  khi từng quy tắc đã có sẵn ở `rules/` để đọc đúng cái cần. Đã đối chiếu từng dòng: nó không chứa gì
  ngoài nội dung `rules/` + mục lục. Bỏ file và cắt 3 dòng mục "Full Compiled Document" ở cuối `SKILL.md`.

Giữ lại: `SKILL.md`, `metadata.json`, `rules/_sections.md` và toàn bộ file quy tắc (70 + 8).

## Đã soát gì trước khi chép (2026-10-02)

- Chỉ có Markdown và mỗi skill một `metadata.json` — không script, không file thực thi, không symlink.
- Không có ký tự vô hình (tag Unicode, zero-width, bidi). Ký tự ngoài ASCII chỉ là gạch dài, mũi tên,
  dấu `×`, nháy cong và emoji (⚠️ ✅ ❌ 😬).
- Không có chỉ dẫn tải nội dung từ URL lúc chạy, không có mục bảo agent gửi phản hồi / mở issue / gửi
  dữ liệu ra dịch vụ ngoài.
- `AGENTS.md` (đã bỏ — xem trên) đã đối chiếu từng dòng với `rules/`: phần thân trùng nguyên văn; phần khác chỉ là tiêu đề,
  tóm tắt và danh sách tham khảo lấy từ `metadata.json`, mục lục, số thứ tự, dòng `Impact` dựng từ
  frontmatter của quy tắc, mô tả mục lấy từ `rules/_sections.md`, nhãn "Incorrect / Correct" đổi dấu ngoặc
  thành dấu hai chấm, và dòng `Reference` đổi chữ hiển thị thành chính URL.
- Lệnh shell duy nhất: `npx svgo --precision=1 --multipass icon.svg` trong
  `rules/rendering-svg-precision.md` — ví dụ tối ưu SVG; chạy thật thì `npx`
  tải gói `svgo` từ npm.
- Đường link trỏ tới `react.dev`, `nextjs.org`, `vercel.com`, `swr.vercel.app`, và ngoài miền của hãng:
  `github.com/shuding/better-all`, `github.com/isaacs/node-lru-cache`,
  `gist.github.com/paulirish/5d52fb081b3570c81e3a`, `csstriggers.com`, `jsfiddle.net/qw1jabsx/4/`,
  `developer.mozilla.org`, `vite.dev`, `esbuild.github.io`, `webpack.js.org`,
  `npmjs.com/package/@rollup/plugin-dynamic-import-vars`. `example.com` chỉ xuất hiện trong code mẫu.
- Vài quy tắc khuyên dùng thư viện ngoài (`better-all`, `lru-cache`, `swr`) — là gợi ý trong ví dụ,
  không phải lệnh cài đặt.

## Cập nhật từ nguồn

```bash
git clone --depth 1 https://github.com/vercel-labs/agent-skills /tmp/up && cd /tmp/up && git rev-parse HEAD
# so từng skill với bản ở đây, ĐỌC phần khác biệt (đặc biệt lệnh / link / file mới), rồi chép đè
diff -r -x README.md -x _template.md /tmp/up/skills/react-best-practices skills/vercel-react-best-practices
diff -r -x README.md -x _template.md /tmp/up/skills/composition-patterns skills/vercel-composition-patterns
ls /tmp/up/LICENSE* 2>/dev/null   # nguồn có file giấy phép rồi thì chép vào LICENSES/ và sửa mục "Giấy phép"
```
Sửa bảng ở trên (commit + ngày), tăng `version` trong `bow-skill.json`, gắn tag mới.
