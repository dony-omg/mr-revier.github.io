# 🤖 MR Reviewer — AI Code Review cho GitLab

Tool tự động review Merge Request bằng **Claude Pro** hoặc **Codex**, post comment trực tiếp lên GitLab (cả inline comment trên từng dòng code).

---

## ⚡ Cài đặt nhanh (1 lần duy nhất)

### Bước 1 — Tạo GitLab Personal Access Token

Vào: **https://gitlab.kyanon.digital/-/user_settings/personal_access_tokens**

| Field | Giá trị |
|---|---|
| Token name | `mr-reviewer` |
| Expiration | để trống hoặc 1 năm |
| Scopes | ✅ `api` |

Copy token (dạng `glpat-xxxxxxxxxxxx`) — **chỉ hiện 1 lần**.

---

### Bước 2 — Cấu hình npm registry

Cần thực hiện **1 lần duy nhất** để npm biết tải package từ GitLab thay vì npmjs.org.

**macOS / Linux** — mở Terminal:

```bash
bash <(curl -s https://gitlab.kyanon.digital/shiva/tool/mr-reviewer/-/raw/main/setup.sh)
```

**Windows** — mở **PowerShell**:

```powershell
powershell -ExecutionPolicy Bypass -Command "Invoke-WebRequest https://gitlab.kyanon.digital/shiva/tool/mr-reviewer/-/raw/main/setup.ps1 -OutFile $env:TEMP\setup.ps1; & $env:TEMP\setup.ps1"
```

Script sẽ hỏi token vừa tạo, rồi tự động:
- Cấu hình npm registry `@shiva` trong `~/.npmrc`
- Cài đặt `mr-review` command
- Lưu token vào `~/.mr-reviewer/.env`

> Không cần clone source hay build gì cả.

---

### Bước 3 — (Tùy chọn) Cài Claude Code để dùng Claude Pro

Nếu bạn có **Claude Pro subscription**:

1. Tải Claude Code: **https://claude.ai/download**
2. Đăng nhập bằng tài khoản Claude Pro
3. Tool sẽ tự nhận diện và cho phép chọn Claude khi review

> Không có Claude Pro? Dùng **Codex** (miễn phí) vẫn hoạt động tốt.

---

## 🚀 Sử dụng hằng ngày

```bash
mr-review <gitlab-mr-url>
```

Hoặc dùng `npx` (không cần cài đặt trước):

```bash
npx @shiva/mr-reviewer <gitlab-mr-url>
```

**Ví dụ:**
```bash
mr-review https://gitlab.kyanon.digital/RX3/FE/amaze-fe/-/merge_requests/5260
```

> Lần đầu chạy chưa có config, tool sẽ tự hỏi GitLab URL và token rồi lưu lại — không cần setup thủ công.

---

### Chọn AI và chế độ comment

```
Chọn AI reviewer:
❯ Claude (claude-opus-4-6)       ← cần Claude Pro
  Codex (default model)          ← miễn phí

Chọn chế độ comment:
  📝 Tổng quan  — 1 comment tổng hợp toàn bộ MR
❯ 📌 Inline + Tổng quan  — comment trên từng dòng code + tổng hợp
```

Dùng phím **↑ ↓** để chọn, **Enter** để xác nhận.

---

### Kết quả

Tool sẽ tự động post comment lên GitLab MR:

```
✅ Review completed!
📝 Posting summary comment...
   ✅ Summary posted.
📌 Posting 4 inline comment(s)...
   ✅ [1/4] src/components/Store.tsx:221
   ✅ [2/4] src/components/Store.tsx:345
✅ Review posted successfully!
🔗 https://gitlab.kyanon.digital/RX3/FE/amaze-fe/-/merge_requests/5260
```

---

## 🔄 Cập nhật lên version mới

Nếu dùng `mr-review` (global install):
```bash
npm install -g @shiva/mr-reviewer@latest
```

Nếu dùng `npx`, tool **tự động dùng version mới nhất** mỗi lần chạy. Để force xóa cache:
```bash
npx --yes @shiva/mr-reviewer@latest <mr-url>
```

---

## ❓ Troubleshooting

### Lỗi "npm ERR! 404 Not Found — @shiva/mr-reviewer"
→ Registry chưa được cấu hình. Chạy lại setup script (Bước 2).

### Lỗi "401 Unauthorized" khi install hoặc post comment
→ Token hết hạn hoặc thiếu quyền `api`. Tạo token mới rồi chạy lại setup script.

### Lỗi "Claude Code CLI not found"
→ Cài Claude Code tại https://claude.ai/download, hoặc chọn Codex thay thế.

### Lỗi "Claude Code process exited with code 1"
→ Thêm vào `~/.mr-reviewer/.env`:
```
CLAUDE_CODE_PATH=/Users/<username>/.local/bin/claude
```
Tìm đường dẫn: `which claude` hoặc `find ~/.local -name claude`

### Windows: Claude Code path
→ Thêm vào `%USERPROFILE%\.mr-reviewer\.env`:
```
CLAUDE_CODE_PATH=C:\Users\<username>\AppData\Local\AnthropicClaude\claude.exe
```

### Windows: Lỗi "execution of scripts is disabled"
→ Mở PowerShell as Administrator, chạy lệnh sau rồi thử lại:
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## 🛠️ Dành cho Team Lead — Publish version mới

```bash
cd mr-reviewer
./publish.sh
```

Chọn loại version bump (patch / minor / major) → script tự build và publish.
