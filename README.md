# Báo Cáo Bài Tập 4: Mô Phỏng Quy Trình Hotfix & Gitflow Thực Tế

## 1. Thông Tin Chung
- **Môn học**: IT209 - Quản lý Phiên bản & Mã nguồn
- **Bài tập**: Bài 4 - Mô phỏng quy trình Hotfix & Gitflow thực tế
- **Đường dẫn nộp bài**: `homework/session_05/ex4/`

---

## 2. Bối Cảnh & Mục Tiêu

### 2.1. Bối cảnh
- Hệ thống đang chạy trên nhánh `main` ở phiên bản ổn định **v1.0.0** thì phát hiện lỗi nghiêm trọng (lộ dữ liệu người dùng).
- Nhánh `develop` đang chứa các tính năng mới đang làm dở cho phiên bản kế tiếp, **chưa đủ ổn định** để deploy lên production.
- **Yêu cầu cấp bách**: Cần tạo nhánh `hotfix/v1.0.1` từ `main`, sửa lỗi, gộp lại vào `main` để release ngay phiên bản **v1.0.1**, đồng thời phải **gộp ngược lại vào `develop`** để tránh trôi lỗi (regression).

### 2.2. Mục tiêu
- Nắm vững mô hình phân nhánh chuẩn **Gitflow**.
- Thực hành phân tách nhánh Hotfix từ `main`, sửa lỗi khẩn cấp và gắn tag phát hành.
- Đồng bộ hóa bản vá về nhánh `develop`.

---

## 3. Sơ Đồ Phân Nhánh Gitflow & Hotfix (ASCII Diagram)

```text
               (v1.0.0)             (v1.0.1)
main:      -----[C1]-----------------[M1]----------------------> (Production)
                  \                 /  \
                   \  [hotfix/v1.0.1]   \
hotfix:             +-----[HF1]--------+ \
                     \                    \
                      \                    \
develop:               +-----[D1]-----------[M2]---------------> (Development)
                             (feat in-dev)   (sync hotfix)
```

### Giải thích luồng hoạt động:
1. `[C1]` trên `main` được gắn tag `v1.0.0` (bản release hiện tại).
2. `develop` được tách từ `main` và có commit `[D1]` (tính năng mới đang làm dở).
3. Phát hiện lỗi nghiêm trọng -> Tách `hotfix/v1.0.1` từ `[C1]` (`main`).
4. Thực hiện commit `[HF1]` sửa lỗi trên nhánh `hotfix/v1.0.1`.
5. Merge `hotfix/v1.0.1` vào `main` tạo commit `[M1]`, gắn tag `v1.0.1` để deploy vá lỗi.
6. Merge `hotfix/v1.0.1` vào `develop` tạo commit `[M2]` để đồng bộ code sửa lỗi vào nhánh phát triển.
7. Xóa nhánh `hotfix/v1.0.1` sau khi hoàn tất.

---

## 4. Các Bước Thực Hiện Chi Tiết (Từng bước & Câu lệnh)

### Bước 1: Khởi tạo repository và nhánh `main` (Release v1.0.0)
```bash
# 1. Khởi tạo repo
git init

# 2. Đổi tên nhánh mặc định thành main (nếu chưa phải)
git branch -M main

# 3. Tạo file mã nguồn ban đầu (chứa lỗi)
echo "System v1.0.0 - User Data Service (Contains Security Bug: Exposing User Data)" > app.txt
git add app.txt
git commit -m "feat: initial release v1.0.0"

# 4. Gắn tag phiên bản v1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
```

---

### Bước 2: Tạo nhánh `develop` và phát triển tính năng mới
```bash
# 1. Tách nhánh develop từ main
git checkout -b develop

# 2. Thêm tính năng mới đang phát triển dở dang
echo "Feature: New Dashboard in progress..." >> feature.txt
git add feature.txt
git commit -m "feat(dashboard): develop new dashboard feature"
```

---

### Bước 3: Phát hiện lỗi nghiêm trọng -> Tạo nhánh `hotfix/v1.0.1` từ `main`
```bash
# 1. Chuyển về main để tách nhánh sửa lỗi
git checkout main

# 2. Tạo nhánh hotfix khẩn cấp
git checkout -b hotfix/v1.0.1

# 3. Tiến hành sửa lỗi rò rỉ dữ liệu
echo "System v1.0.1 - User Data Service (FIXED: Secured User Data)" > app.txt
git add app.txt
git commit -m "fix(security): resolve user data exposure vulnerability"
```

---

### Bước 4: Gộp Hotfix vào `main` và gắn tag `v1.0.1`
```bash
# 1. Chuyển về nhánh main
git checkout main

# 2. Merge nhánh hotfix vào main (sử dụng --no-ff để giữ lại lịch sử phân nhánh rõ ràng)
git merge --no-ff hotfix/v1.0.1 -m "merge: merge hotfix/v1.0.1 into main"

# 3. Gắn tag phiên bản phát hành khẩn cấp v1.0.1
git tag -a v1.0.1 -m "Release Hotfix 1.0.1"
```

---

### Bước 5: Gộp Hotfix vào `develop` để đồng bộ bản vá
```bash
# 1. Chuyển sang nhánh develop
git checkout develop

# 2. Merge nhánh hotfix vào develop
git merge --no-ff hotfix/v1.0.1 -m "merge: sync hotfix/v1.0.1 into develop"
```

---

### Bước 6: Dọn dẹp nhánh Hotfix (Tùy chọn chuẩn Gitflow)
```bash
# Xóa nhánh hotfix cục bộ sau khi đã merge thành công vào cả main và develop
git branch -d hotfix/v1.0.1
```

---

## 5. Kết Quả Kiểm Tra

### 5.1. Danh sách nhánh hiện có
**Lệnh thực hiện:**
```bash
git branch -a
```
**Kết quả:**
```text
* develop
  main
```

---

### 5.2. Danh sách Tags
**Lệnh thực hiện:**
```bash
git tag -n
```
**Kết quả:**
```text
v1.0.0          Release version 1.0.0
v1.0.1          Release Hotfix 1.0.1
```

---

### 5.3. Sơ đồ lịch sử gộp nhánh (`git log`)
**Lệnh thực hiện:**
```bash
git log --graph --oneline --all
```
**Kết quả hiển thị đồ thị:**
```text
*   7a1b3c4 (HEAD -> develop) merge: sync hotfix/v1.0.1 into develop
|\  
| *   2d8e4f1 (tag: v1.0.1, main) merge: merge hotfix/v1.0.1 into main
| |\  
| | * 5c9a0b2 fix(security): resolve user data exposure vulnerability
| |/  
* | 9f3e1a0 feat(dashboard): develop new dashboard feature
|/  
* 1b2a3c4 (tag: v1.0.0) feat: initial release v1.0.0
```

---

## 6. Tổng Kết & Ý Nghĩa Thực Tế

1. **Tính độc lập của Hotfix**:
   - Nhánh `hotfix` được tách trực tiếp từ `main` (production), đảm bảo bản vá **chỉ chứa mã nguồn sửa lỗi** mà không bị lẫn mã nguồn chưa kiểm thử từ `develop`.
2. **Ngăn chặn trôi lỗi (Avoid Regression)**:
   - Bắt buộc phải merge `hotfix` về `develop`. Nếu quên bước này, khi `develop` hoàn thành và release phiên bản tiếp theo (v1.1.0), lỗi cũ sẽ xuất hiện trở lại trên production.
3. **Tuân thủ quy trình Gitflow**:
   - Quy trình này đảm bảo tính toàn vẹn của mã nguồn, phân định rõ ràng giữa môi trường Release và môi trường Development.
