# LUỒNG USER CHI TIẾT – StudyHub

Tài liệu mô tả chi tiết 6 luồng chính của người dùng trên nền tảng StudyHub.

---

## 1. LUỒNG CỘNG ĐỒNG (Community)

### 1.1. Xem feed cộng đồng

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | User vào `/community` |
| 2 | Lấy dữ liệu | API `GET /api/community` — bài viết không thuộc nhóm cụ thể |
| 3 | Lọc | Theo nhóm (`groupId`), tag, thời gian (all / today / week) |
| 4 | Tab | Tất cả / Theo nhóm / Đã lưu / Đang theo dõi |

### 1.2. Tương tác với bài viết

| Hành động | API | Ghi chú |
|-----------|-----|---------|
| Like | `POST /api/community/[postId]/like` | Optimistic update trên UI |
| Lưu bài | (trong code) | Trạng thái saved_by_user |
| Bình luận | `POST /api/community/[postId]/comments` | Hỗ trợ reply lồng nhau |
| Xem chi tiết | `/community/[postId]` | `GET /api/community/[postId]` |

### 1.3. Đăng bài mới

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/community/create` hoặc `/community/create?groupId=xxx` (đăng trong nhóm) |
| 2 | Nhập nội dung | Tiêu đề (tùy chọn), nội dung, tag (tối đa 5), ảnh, file đính kèm |
| 3 | Tùy chọn | Ẩn danh, gắn nhóm |
| 4 | Gửi | `POST /api/community` — tạo bài viết mới |

### 1.4. Sơ đồ luồng Cộng đồng

```
/community → Xem feed → Lọc (nhóm, tag, thời gian)
    ├── Like bài
    ├── Lưu bài
    ├── Bình luận (có reply)
    ├── Click bài → /community/[postId] → Xem chi tiết, like, comment
    └── Đăng bài → /community/create → Nhập nội dung, tag, ảnh, file → POST
```

---

## 2. LUỒNG NHÓM HỌC (Groups)

### 2.1. Xem danh sách nhóm

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/groups` |
| 2 | Lấy dữ liệu | `GET /api/groups` — kèm trạng thái membership (member / pending / none) nếu đã đăng nhập |
| 3 | Sắp xếp | Phổ biến / Mới nhất / Gợi ý |
| 4 | Lọc | Công khai / Riêng tư, tìm theo tên/môn |

### 2.2. Tham gia nhóm

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Vào chi tiết nhóm | `/groups/[groupId]` |
| 2 | Bấm "Tham gia" | Gọi `POST /api/groups/[groupId]/join` |
| 3 | Kết quả | **Nhóm công khai**: status = `active` → Tham gia ngay |
| 4 | Kết quả | **Nhóm riêng tư**: status = `pending` → Chờ admin/mentor duyệt |

### 2.3. Chi tiết nhóm (sau khi tham gia)

| Tab | Nội dung | API / Hành động |
|-----|----------|-----------------|
| Bài viết | Bài trong nhóm | Lấy từ community (filter groupId) |
| Thành viên | Danh sách member | Admin: duyệt/từ chối pending, xóa member |
| Tài liệu | File đính kèm bài viết | Chỉ VIP/Mentor/Admin xem được |
| Sự kiện | (Placeholder) | Chức năng đang phát triển |

### 2.4. Tạo nhóm

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/groups/create` |
| 2 | Điều kiện | **VIP** hoặc **Mentor/Admin/Staff** mới tạo được |
| 3 | Nhập thông tin | Tên, mô tả, tag môn học, công khai/riêng tư |
| 4 | Riêng tư | Chỉ Mentor/Admin tạo được nhóm riêng tư |
| 5 | Mời thành viên | Nhập email (tùy chọn) |
| 6 | Gửi | `POST /api/groups` |

### 2.5. Rời nhóm

- Gọi `POST /api/groups/[groupId]/leave` — xóa membership, giảm `membersCount`.

### 2.6. Sơ đồ luồng Nhóm học

```
/groups → Xem danh sách → Lọc, sắp xếp
    → /groups/[groupId] → Xem chi tiết
        ├── Tham gia (public: active ngay / private: pending)
        ├── Tab Bài viết: đọc, tạo bài (nếu member)
        ├── Tab Thành viên: xem (admin: duyệt pending)
        ├── Tab Tài liệu: xem (VIP/Mentor/Admin)
        └── Rời nhóm

/groups/create → Chỉ VIP/Mentor/Admin → Nhập thông tin → POST
```

---

## 3. LUỒNG POMODORO

### 3.1. Sử dụng timer

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/pomodoro` |
| 2 | Chọn chế độ | **Tập trung** (25 phút) / **Nghỉ ngắn** (5 phút) / **Nghỉ dài** (15 phút) |
| 3 | Chạy timer | Bấm Play → đếm ngược từng giây |
| 4 | Hoàn thành Focus | Phát âm thanh, tăng `completedSessions` |
| 5 | Lưu phiên | Nếu đã đăng nhập → `POST /api/pomodoro/session` (lưu duration, subject) |

### 3.2. Thống kê (chỉ VIP)

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Kiểm tra VIP | `GET /api/upgrade` |
| 2 | Lấy thống kê | `GET /api/pomodoro/stats` |
| 3 | Hiển thị | Tổng phiên, tổng phút, streak, biểu đồ theo tuần/tháng, phân bố môn học |

### 3.3. Sơ đồ luồng Pomodoro

```
/pomodoro → Chọn chế độ (Focus / Short Break / Long Break)
    → Bấm Play → Đếm ngược
    → Hết Focus → Lưu phiên (nếu đăng nhập) → POST /api/pomodoro/session
    → VIP: Xem thống kê (GET /api/pomodoro/stats)
```

---

## 4. LUỒNG MENTOR

### 4.1. Xem danh sách mentor

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/mentors` |
| 2 | Lấy dữ liệu | `GET /api/mentors?limit=30&search=&subject=` |
| 3 | Lọc | Theo môn học, ngày rảnh, khung giờ (Sáng/Chiều/Tối) |
| 4 | Sắp xếp | Đánh giá, giá tăng/giảm, số buổi |

### 4.2. Xem chi tiết mentor

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Click mentor | `/mentors/[id]` |
| 2 | Lấy dữ liệu | `GET /api/mentors/[id]` — trả về profile + reviews + courses (1 request) |
| 3 | Xem | Hồ sơ, khóa học, đánh giá, lịch rảnh |

### 4.3. Gửi yêu cầu đặt lịch (luồng đầy đủ)

| Bước | Hành động | API / Trạng thái |
|------|-----------|------------------|
| 1 | Mở form | Bấm "Nói chuyện với Mentor" |
| 2 | Chọn slot | Khung giờ rảnh (VD: Thứ 3 - Sáng) |
| 3 | Nhập chủ đề | Topic, loại (Học cùng / Tư vấn), ghi chú |
| 4 | Gửi | `POST /api/mentor-bookings` → status = `pending` |
| 5 | Mentor duyệt | Mentor chấp nhận → status = `confirmed` |
| 6 | Thanh toán | User bấm "Thanh toán ngay" → `POST /api/mentor-bookings/[id]/pay` → redirect PayOS |
| 7 | PayOS | User thanh toán → redirect `/payment/success` |
| 8 | Xác nhận | `POST /api/payments/confirm` → status = `paid` |
| 9 | Mentor thêm link | Mentor nhập link Meet/Zoom → `PATCH /api/mentor-bookings/[id]/meeting-link` |
| 10 | User vào học | `/schedule` → nút "Vào link học" (chỉ khi paid + có link) |
| 11 | Hoàn thành | Mentor đánh dấu → status = `completed` |
| 12 | Đánh giá | User đánh giá (buổi tư vấn) → `POST /api/mentor-reviews` |

### 4.4. Xem đơn yêu cầu của mình

- Trên trang mentor: bấm "Đơn yêu cầu" → modal hiển thị đơn với mentor đó.
- Lấy từ `GET /api/mentor-bookings?role=student`, lọc theo `mentorId`.

### 4.5. Sơ đồ luồng Mentor

```
/mentors → Xem danh sách → Lọc, sắp xếp
    → /mentors/[id] → Xem chi tiết (profile, courses, reviews)
        ├── Gửi yêu cầu → pending
        │   → Mentor duyệt → confirmed
        │   → User thanh toán PayOS → paid
        │   → Mentor thêm link
        │   → User vào /schedule → "Vào link học"
        │   → Mentor hoàn thành → completed
        │   → User đánh giá (tư vấn)
        └── Xem "Đơn yêu cầu" (modal)
```

---

## 5. LUỒNG LỊCH HỌC (Schedule)

### 5.1. Xem lịch theo tuần

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/schedule` (yêu cầu đăng nhập) |
| 2 | Lấy dữ liệu | `GET /api/mentor-bookings?role=student&from=&to=` (theo tuần) |
| 3 | Hiển thị | Bảng lịch theo ngày + giờ (7h–21h) |
| 4 | Điều hướng | Tuần trước / Tuần hiện tại / Tuần sau |

### 5.2. Trạng thái đơn trên lịch

| Status | Màu | Ý nghĩa |
|--------|-----|---------|
| pending | Vàng | Chờ thanh toán |
| confirmed | Xanh dương | Đã xác nhận |
| paid | Xanh lá | Đã thanh toán |
| completed | Xám | Hoàn thành |
| cancelled | Xám gạch | Đã hủy |

### 5.3. Link học

- Chỉ hiển thị nút **"Vào link học"** khi: `status = paid` hoặc `completed` **và** có `meetingLink`.

### 5.4. Đánh giá

- Buổi **tư vấn** đã `completed` → User có thể đánh giá (sao + nhận xét) qua modal.

### 5.5. Sơ đồ luồng Lịch học

```
/schedule → Đăng nhập bắt buộc
    → Chọn tuần → Bảng lịch (ngày × giờ)
    → Mỗi ô: topic, mentor, status
    → paid/completed + meetingLink → "Vào link học"
    → completed (tư vấn) → "Đánh giá"
```

---

## 6. LUỒNG VIP (Nâng cấp)

### 6.1. Xem gói và quyền lợi

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Truy cập | `/upgrade` |
| 2 | Xem gói | Hàng tháng / 3 tháng / 1 năm (giá khác nhau) |
| 3 | Quyền lợi | 2 buổi mentor miễn phí/tháng, badge VIP, nhóm premium, thống kê Pomodoro |

### 6.2. Thanh toán

| Bước | Hành động | Chi tiết |
|------|-----------|----------|
| 1 | Chọn gói | Bấm chọn plan |
| 2 | Bấm nâng cấp | Yêu cầu đăng nhập nếu chưa |
| 3 | Tạo thanh toán | `POST /api/upgrade` với `planId` → nhận `checkoutUrl` |
| 4 | Redirect | Chuyển đến PayOS |
| 5 | Thanh toán | User thanh toán trên PayOS |
| 6 | Success | Redirect `/payment/success?type=vip_upgrade&planId=xxx` |
| 7 | Xác nhận | `POST /api/payments/confirm` → cập nhật `vipPlan`, `vipExpiresAt` |
| 8 | Kích hoạt | VIP ngay lập tức |

### 6.3. Sơ đồ luồng VIP

```
/upgrade → Xem gói (monthly / quarterly / yearly)
    → Chọn gói → Bấm nâng cấp
    → Đăng nhập (nếu cần)
    → POST /api/upgrade → checkoutUrl
    → PayOS thanh toán
    → /payment/success → confirm → Kích hoạt VIP
```

---

## TỔNG HỢP ĐƯỜNG DẪN USER

| Luồng | Trang chính | Trang phụ |
|-------|-------------|-----------|
| Cộng đồng | `/community` | `/community/create`, `/community/[postId]` |
| Nhóm học | `/groups` | `/groups/[groupId]`, `/groups/create` |
| Pomodoro | `/pomodoro` | — |
| Mentor | `/mentors` | `/mentors/[id]` |
| Lịch học | `/schedule` | — |
| VIP | `/upgrade` | `/payment/success`, `/payment/cancel` |

---

*Tài liệu mô tả chi tiết luồng User cho dự án StudyHub.*
