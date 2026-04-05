# Git & GitHub: hướng dẫn làm việc nhóm

Tài liệu này mô tả thao tác branch, fetch/pull, và các tình huống thường gặp khi nhiều người cùng làm việc trên một repository GitHub — đặc biệt là cách **đồng bộ với remote mà không làm mất code local**.

---

## 1. Khái niệm nhanh

| Thuật ngữ | Ý nghĩa |
|-----------|---------|
| **Working tree** | Thư mục dự án trên máy bạn; file đang sửa nằm ở đây. |
| **Staging (`git add`)** | Đánh dấu thay đổi sẽ đi vào commit tiếp theo. |
| **Commit** | Snapshot đã ghi lại trong lịch sử local; có hash, message. |
| **Branch (nhánh)** | Con trỏ di chuyển theo commit; cho phép phát triển song song. |
| **Remote** | Máy chủ (thường là GitHub): `origin` là tên mặc định. |
| **`origin/main` (hoặc `origin/master`)** | Nhánh mặc định **trên GitHub** — nguồn chung của cả nhóm. |

Luồng cơ bản: **sửa file → `add` → `commit` → `push`** lên nhánh của bạn (hoặc nhánh feature), sau đó **Pull Request (PR)** để gộp vào `main` sau khi review.

---

## 2. Branch: tạo, chuyển, đặt tên, xóa

### Tạo và chuyển nhánh

```bash
# Tạo nhánh mới từ commit hiện tại và chuyển sang
git switch -c feature/ten-tinh-nang

# Hoặc (cú pháp cũ vẫn dùng được)
git checkout -b feature/ten-tinh-nang
```

### Chỉ chuyển nhánh (đã tồn tại)

```bash
git switch ten-nhanh
# hoặc: git checkout ten-nhanh
```

### Đẩy nhánh mới lên GitHub lần đầu

```bash
git push -u origin feature/ten-tinh-nang
```

`-u` (upstream) giúp lần sau chỉ cần `git push` / `git pull` trên nhánh này.

### Xóa nhánh (local / remote)

```bash
# Xóa local sau khi đã merge xong
git branch -d feature/ten-tinh-nang

# Bắt buộc xóa nếu chưa merge (cẩn thận)
git branch -D feature/ten-tinh-nang

# Xóa trên GitHub
git push origin --delete feature/ten-tinh-nang
```

**Quy ước đặt tên (gợi ý):** `feature/...`, `fix/...`, `chore/...` — giúp PR và lịch sử dễ đọc.

---

## 3. `fetch` và `pull`: khác nhau thế nào?

### `git fetch`

- Tải **commit, nhánh, tag mới** từ `origin` về máy bạn.
- **Không** tự gộp vào nhánh đang đứng; **không** sửa working tree của bạn.
- Sau `fetch`, bạn có thể xem `origin/main` đã đi tới đâu so với `main` local.

```bash
git fetch origin
```

Hữu ích để **xem trước** thay đổi trên GitHub trước khi merge/rebase.

### `git pull`

- Thường là: **`fetch` + `merge`** (hoặc `fetch` + `rebase` nếu cấu hình như vậy) vào nhánh hiện tại.
- **Có thể** tạo merge commit hoặc conflict cần xử lý ngay.

```bash
git pull origin main
```

**Khuyến nghị khi làm việc nhóm:** hiểu rõ `fetch` trước; sau đó chủ động `merge` hoặc `rebase` để kiểm soát từng bước (mục 5).

---

## 4. Nhiều người cùng dự án: mô hình thường dùng

1. **`main` (hoặc `master`)** luôn **ổn định**, có thể deploy; **không push trực tiếp** (nếu team quy định vậy) — mọi thứ vào qua PR.
2. Mỗi tính năng / sửa lỗi: **nhánh riêng** từ `main` mới nhất.
3. Làm xong: **PR → review → merge** (thường merge trên GitHub).
4. Sau khi merge: trên máy local, **cập nhật `main`** và có thể xóa nhánh feature.

```bash
git switch main
git pull origin main
git branch -d feature/ten-tinh-nang   # nếu đã merge
```

Điều này giảm xung đột vì mỗi người làm trên “bản sao” thời điểm khác nhau của `main`, rồi gộp có kiểm soát.

---

## 5. Đồng bộ với GitHub **không mất code local**

Nguyên tắc: **Git không “xóa” commit đã tạo** trừ khi bạn chủ động reset/reflog một cách nguy hiểm. Code “mất” thường do: **ghi đè nhánh**, **pull không commit trước**, hoặc **force push** sai.

### 5.1 Luôn biết trạng thái trước khi pull

```bash
git status
```

- Nếu có thay đổi **chưa commit**: xem mục 5.2–5.3.
- Nếu **đã commit** trên nhánh feature: an toàn hơn khi merge/rebase với `main`.

### 5.2 Tạm cất thay đổi chưa commit (`stash`)

Khi cần `git pull` hoặc đổi nhánh nhưng **chưa muốn commit**:

```bash
git stash push -m "mô tả ngắn"
git pull origin main
git stash pop    # lấy lại thay đổi; có thể phải sửa conflict
```

`stash pop` có thể báo conflict — giải quyết như conflict merge bình thường.

### 5.3 Commit trước, rồi mới đồng bộ (khuyến nghị)

Cách an toàn nhất cho công việc đang làm:

```bash
git add -A
git commit -m "WIP: mô tả"
git pull origin ten-nhanh-dang-lam   # hoặc merge main vào nhánh feature
```

Sau đó xử lý conflict nếu có. **Commit là điểm khôi phục** trong lịch sử.

### 5.4 Cập nhật nhánh feature với `main` mới nhất

Sau khi đồng đội merge PR vào `main`:

**Cách A — Merge (đơn giản, lịch sử có merge commit):**

```bash
git switch feature/ten-tinh-nang
git fetch origin
git merge origin/main
```

**Cách B — Rebase (lịch sử thẳng hơn; cần quen):**

```bash
git switch feature/ten-tinh-nang
git fetch origin
git rebase origin/main
```

Nếu conflict: sửa file → `git add` → `git rebase --continue`. Hủy rebase: `git rebase --abort`.

**Lưu ý:** Nếu nhánh đã **push** và rebase làm đổi lịch sử, lần push sau cần:

```bash
git push --force-with-lease origin feature/ten-tinh-nang
```

Chỉ dùng trên **nhánh cá nhân/feature**, không force lên `main` trừ khi quy trình repo cho phép.

---

## 6. Các tình huống thường gặp

### 6.1 “Remote có commit mới, mình cũng có commit local chưa push”

```bash
git fetch origin
git status
# Trên cùng một nhánh:
git pull origin ten-nhanh
# Hoặc: git merge origin/ten-nhanh
```

Nếu không conflict: Git tạo merge commit hoặc fast-forward. Nếu conflict: mở file có marker `<<<<<<<`, sửa → `git add` → `git commit` (hoặc tiếp tục merge).

### 6.2 Merge conflict khi pull/merge

1. Mở từng file báo conflict.
2. Tìm `<<<<<<<`, `=======`, `>>>>>>>` — giữ đúng nội dung cần thiết, xóa marker.
3. `git add <file>`
4. `git commit` (nếu đang merge) hoặc `git rebase --continue` (nếu đang rebase).

### 6.3 “Tôi đã sửa nhầm trên `main`, chưa push”

```bash
git stash
git switch -c feature/sua-lai
git stash pop
git add -A && git commit -m "..."
git push -u origin feature/sua-lai
```

Giữ `main` sạch bằng cách chuyển work sang nhánh mới.

### 6.4 “Ai đó đã sửa cùng file — PR bị conflict”

Trên nhánh feature của bạn:

```bash
git fetch origin
git merge origin/main
# hoặc: git rebase origin/main
```

Giải conflict, commit/rebase xong, push nhánh (có thể `--force-with-lease` nếu đã rebase và từng push).

### 6.5 “Lỡ `git reset --hard` / xóa nhánh — có lấy lại được không?”

Trong thời gian ngắn, commit thường còn trong **reflog**:

```bash
git reflog
git checkout -b khoi-phuc <hash-commit>
```

Không đảm bảo mãi mãi — **push lên remote** hoặc **branch backup** là cách an toàn nhất.

### 6.6 Force push: khi nào được phép?

| Tình huống | Ghi chú |
|------------|---------|
| `git push --force-with-lease` lên **nhánh feature của bạn** sau rebase | An toàn hơn `--force` vì từ chối nếu remote có commit mới của người khác. |
| Force push lên **`main`** | Thường **cấm**; làm mất lịch sử của cả nhóm. |

### 6.7 Hai người cùng tạo nhánh tên giống nhau

Tránh trùng tên; nếu đã trùng: đổi tên local `git branch -m ten-moi` và push nhánh mới.

### 6.8 PR đã merge trên GitHub — local vẫn cũ

```bash
git switch main
git pull origin main
```

Xóa nhánh feature local nếu không cần: `git branch -d feature/...`.

---

## 7. Checklist ngắn trước khi bắt đầu ngày làm việc

1. `git fetch origin`
2. `git switch main && git pull origin main`
3. `git switch feature/...` và merge/rebase `main` nếu nhánh sống lâu.

Trước khi push:

1. `git status` — không còn file quan trọng sót chưa add (trừ khi cố ý ignore).
2. Chạy test/lint theo quy ước dự án.
3. Push nhánh → mở/cập nhật PR.

---

## 8. Tài liệu tham khảo chính thức

- [Git Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [GitHub: Collaborating with pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)

---

*Tài liệu này nhằm chuẩn hóa thao tác trong nhóm; quy trình cụ thể (bảo vệ nhánh `main`, bắt buộc PR, CI) nên được team thống nhất thêm trong quy định nội bộ.*
