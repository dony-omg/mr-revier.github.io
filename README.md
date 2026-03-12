# 🤖 MR Reviewer — AI Code Review cho GitLab

Tool tự động review Merge Request bằng **Claude Pro** hoặc **Codex**, post comment trực tiếp lên GitLab (cả inline comment trên từng dòng code).

---

## 🧠 Tại sao dùng AI để review MR?

### Vấn đề thực tế
- **Review thủ công tốn thời gian**: Reviewer mất 15–30 phút mỗi MR chỉ để đọc diff, trong khi còn phải làm việc khác.
- **Dễ bỏ sót lỗi**: Sau nhiều MR liên tiếp, sự tập trung giảm — bug nhỏ, security issue, hay logic sai lặt vặt hay bị bỏ qua.
- **Feedback không nhất quán**: Mỗi người review theo style khác nhau, gây khó khăn cho người nhận.
- **Bottleneck khi team lớn**: Ít senior reviewer phải duyệt nhiều MR → MR tồn đọng, release chậm.

### Claude và Codex giải quyết điều đó như thế nào?

| | Claude (claude-opus-4-6) | Codex |
|---|---|---|
| **Điểm mạnh** | Hiểu ngữ cảnh sâu, phân tích logic phức tạp, comment chi tiết | Nhanh, miễn phí, phù hợp MR nhỏ |
| **Inline comment** | ✅ Có (chỉ đúng dòng code có vấn đề) | ❌ Chỉ tổng quan |
| **Phân loại severity** | ✅ Critical / Warning / Suggestion | ✅ |
| **Yêu cầu** | Claude Pro subscription | Cài Codex (miễn phí) |

### Lợi ích cụ thể
- **Tiết kiệm thời gian**: AI đọc toàn bộ diff trong vài giây, reviewer chỉ cần xem lại comment và quyết định.
- **Không bỏ sót**: Phát hiện các lỗi thường gặp — missing error handling, potential NPE, security risk (XSS, SQL injection), logic sai điều kiện.
- **Nhất quán**: Mọi MR đều được review theo cùng tiêu chí, không phụ thuộc vào mood của reviewer.
- **Comment thẳng vào dòng code**: Với Claude, comment được post trực tiếp lên đúng dòng diff — reviewer và author dễ theo dõi hơn so với comment chung chung.

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
curl -s -H "PRIVATE-TOKEN: <your-gitlab-token>" \
  "https://gitlab.kyanon.digital/api/v4/projects/shiva%2Ftool%2Fmr-reviewer/repository/files/setup.sh/raw?ref=main" \
  -o /tmp/mr-setup.sh && bash /tmp/mr-setup.sh
```

**Windows** — mở **PowerShell**:

```powershell
$token="<your-gitlab-token>"; Invoke-WebRequest -Headers @{"PRIVATE-TOKEN"=$token} "https://gitlab.kyanon.digital/api/v4/projects/shiva%2Ftool%2Fmr-reviewer/repository/files/setup.ps1/raw?ref=main" -OutFile "$env:TEMP\setup.ps1"; & "$env:TEMP\setup.ps1"
```

> ⚠️ Lưu ý: phải dùng URL dạng `/api/v4/projects/...` (không dùng URL web `/-/raw/main/...`)

Script sẽ hỏi token vừa tạo, rồi tự động:
- Cấu hình npm registry `@shiva` trong `~/.npmrc`
- Cài đặt `mr-review` command
- Lưu token vào `~/.mr-reviewer/.env`

> Không cần clone source hay build gì cả.

---

### Bước 3 — (Tùy chọn) Cài AI provider

**Claude Pro** (review chuyên sâu hơn):
1. Tải Claude Code: **https://claude.ai/download**
2. Đăng nhập bằng tài khoản Claude Pro
3. Tool sẽ tự nhận diện và cho phép chọn Claude khi review

**Codex** (miễn phí):
- Windows: **https://developers.openai.com/codex/app/windows/**

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
