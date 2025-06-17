# Bài tập giữa kì: Thiết kế todo Web

# Họ và tên: Lê Trung Hiếu 

# Mã sinh viên: 22010482

## Mục lục

- [Giới thiệu](#giới-thiệu)
- [Sơ đồ cấu trúc](#sơ-đồ-cấu-trúc)
- [Sơ đồ thuật toán](#sơ-đồ-thuật-toán)
- [Giao diện thực tế](#giao-diện-thực-tế)
- [Code minh họa](#code-minh-họa)
- [Link Repo](#link-repo)
- [Link Deploy](#link-deploy)

## Giới thiệu

Dự án To-do App được xây dựng nhằm giúp người dùng quản lý công việc cá nhân một cách hiệu quả và khoa học. Ứng dụng cho phép tạo, chỉnh sửa, xóa và đánh dấu hoàn thành các nhiệm vụ hàng ngày. Ngoài ra, hệ thống hỗ trợ phân loại công việc theo trạng thái, ưu tiên, giúp người dùng dễ dàng theo dõi tiến độ và sắp xếp công việc hợp lý.

Phạm vi dự án bao gồm xây dựng một ứng dụng web sử dụng PHP và Laravel cho backend, kết hợp với giao diện thân thiện, dễ sử dụng. Ứng dụng hướng đến trải nghiệm mượt mà, bảo mật thông tin cá nhân và có khả năng mở rộng trong tương lai như tích hợp nhắc nhở, đồng bộ hóa đa thiết bị hoặc chia sẻ công việc với người khác. Dự án phù hợp cho cá nhân và nhóm nhỏ mong muốn tối ưu hóa quy trình làm việc hàng ngày.

## Sơ đồ cấu trúc

![Sơ đồ cấu trúc](Image-report/struct-diagram.png)

### Mô tả sơ đồ cơ sở dữ liệu

Dự án sử dụng cơ sở dữ liệu quan hệ gồm ba bảng chính: **Users** (Người dùng), **Lists** (Danh sách), và **Tasks** (Nhiệm vụ). Các bảng liên kết với nhau qua khóa chính (PK) và khóa ngoại (FK).

#### **Bảng Users**
- **Khóa chính:** `user_id` (string)
- **Các trường:** 
    - `name` (string)
    - `email` (string)
    - `password` (string)
- **Mô tả:** Lưu trữ thông tin người dùng như tên, email và mật khẩu.

#### **Bảng Lists**
- **Khóa chính:** `list_id`
- **Khóa ngoại:** `user_id` (tham chiếu đến `user_id` trong bảng Users)
- **Các trường:** 
    - `title` (string)
    - `description` (string)
- **Mô tả:** Lưu trữ các danh sách công việc của từng người dùng, liên kết với bảng Users qua `user_id`.

#### **Bảng Tasks**
- **Khóa chính:** `task_id`
- **Khóa ngoại:** `list_id` (tham chiếu đến `list_id` trong bảng Lists)
- **Các trường:** 
    - `title` (string)
    - `description` (string)
    - `is_completed` (boolean)
    - `due_date` (date)
- **Mô tả:** Lưu trữ các nhiệm vụ cụ thể trong từng danh sách, bao gồm tiêu đề, mô tả, trạng thái hoàn thành và ngày đến hạn.

#### **Quan hệ giữa các bảng**
- Một người dùng (**Users**) có thể sở hữu nhiều danh sách (**Lists**) thông qua `user_id`.
- Một danh sách (**Lists**) có thể chứa nhiều nhiệm vụ (**Tasks**) thông qua `list_id`.

## Sơ đồ thuật toán

### Thuật toán thống kê dữ liệu tổng quan
<p align="start">
    <img src="Image-report/dashboard-diagram.png" alt="Thuật toán thống kê dữ liệu tổng quan" width="300"/>
</p>

#### Sơ đồ hoạt động: Lấy Dữ liệu cho Trang Chủ

```mermaid
flowchart TD
    A[Bắt đầu] --> B{Người dùng đã đăng nhập?}
    B -- Không --> C[Chuyển hướng đến trang đăng nhập]
    B -- Có --> D[Lấy thông tin người dùng hiện tại]
    D --> E[Lấy danh sách (Lists) thuộc về người dùng]
    E --> F[Lấy nhiệm vụ (Tasks) thuộc về các danh sách]
    F --> G[Tính toán thống kê (Stats)]
    G --> H[Chuẩn bị dữ liệu để render]
    H --> I[Render trang chủ với Inertia]
    I --> J[Kết thúc]
```

**Chi tiết các bước:**

1. **Bắt đầu:**  
   Hệ thống khởi động chức năng `index` trong `DashboardController`.

2. **Kiểm tra người dùng đã đăng nhập:**  
   - Lấy thông tin người dùng hiện tại bằng `auth()->user()`.
   - Nếu không có người dùng, chuyển hướng đến trang đăng nhập (`redirect()->route('login')`).
   - Nếu có người dùng, tiếp tục quá trình.

3. **Lấy danh sách (Lists):**  
   Thực hiện truy vấn `TaskList::where('user_id', $user->id)->get()` để lấy tất cả danh sách thuộc về người dùng hiện tại.

4. **Lấy nhiệm vụ (Tasks):**  
   Thực hiện truy vấn `Task::whereHas('list', function ($query) use ($user) {...})->get()` để lấy tất cả nhiệm vụ thuộc về các danh sách của người dùng hiện tại.

5. **Tính toán thống kê (Stats):**  
   Tạo mảng `$stats` với các giá trị:
   - `totalLists`: Số lượng danh sách (`$lists->count()`).
   - `totalTasks`: Tổng số nhiệm vụ (`$tasks->count()`).
   - `completedTasks`: Số nhiệm vụ đã hoàn thành (`$tasks->where('is_completed', true)->count()`).
   - `pendingTasks`: Số nhiệm vụ chưa hoàn thành (`$tasks->where('is_completed', false)->count()`).

6. **Chuẩn bị dữ liệu để render:**  
   Gộp các dữ liệu vào mảng để truyền cho giao diện:
   - `stats`: Mảng thống kê.
   - `lists`: Danh sách đã lấy.
   - `tasks`: Nhiệm vụ đã lấy.
   - `flash`: Thông báo thành công hoặc lỗi từ session (`session('success')`, `session('error')`).

7. **Render trang chủ:**  
   Sử dụng `Inertia::render('dashboard', [...])` để hiển thị trang dashboard với dữ liệu đã chuẩn bị.

8. **Kết thúc:**  
   Quá trình hoàn tất, trang dashboard được hiển thị cho người dùng.

### Thuật toán thêm, sửa, xóa List

# Thuật toán Thêm, Sửa, Xóa List

## Chi tiết các bước:

### Bắt đầu:
- Hệ thống khởi động chức năng trong `ListController` dựa trên yêu cầu từ người dùng (thêm, sửa, xóa).

### Kiểm tra người dùng đã đăng nhập:
- Sử dụng `auth()->id()` để lấy ID của người dùng hiện tại.
- Nếu không có người dùng (chưa đăng nhập), chuyển hướng đến trang đăng nhập (`route to 'login'`).
- Nếu đã đăng nhập, tiếp tục xử lý yêu cầu.

### Chọn hành động: Thêm/Sửa/Xóa:
- Xác định hành động dựa trên route được gọi:
  - `store` cho việc thêm mới.
  - `update` cho việc sửa.
  - `destroy` cho việc xóa.

### Thêm List:
- **Lấy dữ liệu từ Request**: Nhận dữ liệu từ `Request $request` (bao gồm `title` và `description`).
- **Kiểm tra và xác thực dữ liệu**:
  - Sử dụng `$request->validate()` để kiểm tra:
    - `title` là chuỗi bắt buộc, tối đa 255 ký tự.
    - `description` là chuỗi tùy chọn.
- **Tạo mới List với dữ liệu đã xác thực**:
  - Sử dụng `TaskList::create()` với dữ liệu đã xác thực, thêm `user_id` từ `auth()->id()`.
- **Chuyển hướng về trang `lists.index` với thông báo thành công**:
  - Sử dụng `redirect()->route('lists.index')->with('success', 'List created successfully!')`.

### Sửa List:
- **Lấy List cụ thể**: Nhận đối tượng `TaskList $list` thông qua route model binding.
- **Kiểm tra và xác thực dữ liệu**:
  - Sử dụng `$request->validate()` để kiểm tra:
    - `title` là chuỗi bắt buộc, tối đa 255 ký tự.
    - `description` là chuỗi tùy chọn.
- **Cập nhật List với dữ liệu đã xác thực**:
  - Sử dụng `$list->update($validated)` để cập nhật thông tin.
- **Chuyển hướng về trang `lists.index` với thông báo thành công**:
  - Sử dụng `redirect()->route('lists.index')->with('success', 'List updated successfully!')`.

### Xóa List:
- **Lấy List cụ thể**: Nhận đối tượng `TaskList $list` thông qua route model binding.
- **Xóa List**: Sử dụng `$list->delete()` để xóa bản ghi khỏi cơ sở dữ liệu.
- **Chuyển hướng về trang `lists.index` với thông báo thành công**:
  - Sử dụng `redirect()->route('lists.index')->with('success', 'List deleted successfully!')`.

### Kết thúc:
- Quá trình hoàn tất, người dùng được chuyển hướng về trang danh sách (`lists.index`) với thông báo tương ứng.

### Thuật toán thêm, sửa, xóa Task

# Thuật toán Thêm, Sửa, Xóa Task

## Chi tiết các bước:

### Bắt đầu:
- Hệ thống khởi động chức năng trong `TaskController` dựa trên yêu cầu từ người dùng (thêm, sửa, xóa).

### Kiểm tra người dùng đã đăng nhập:
- Sử dụng `auth()->id()` để lấy ID của người dùng hiện tại.
- Nếu không có người dùng (chưa đăng nhập), chuyển hướng đến trang đăng nhập (`route to 'login'`).
- Nếu đã đăng nhập, tiếp tục xử lý yêu cầu.

### Chọn hành động: Thêm/Sửa/Xóa:
- Xác định hành động dựa trên route được gọi:
  - `store` cho việc thêm mới.
  - `update` cho việc sửa.
  - `destroy` cho việc xóa.

### Thêm Task:
- **Lấy dữ liệu từ Request**: Nhận dữ liệu từ `Request $request` (bao gồm `title`, `description`, `due_date`, `list_id`, và `is_completed`).
- **Kiểm tra và xác thực dữ liệu**:
  - Sử dụng `$request->validate()` để kiểm tra:
    - `title` là chuỗi bắt buộc, tối đa 255 ký tự.
    - `description` là chuỗi tùy chọn.
    - `due_date` là ngày hợp lệ (tùy chọn).
    - `list_id` là bắt buộc và phải tồn tại trong bảng `lists`.
    - `is_completed` là kiểu boolean (tùy chọn).
- **Tạo mới Task với dữ liệu đã xác thực**:
  - Sử dụng `Task::create($validated)` để lưu Task mới vào cơ sở dữ liệu.
- **Chuyển hướng về trang `tasks.index` với thông báo thành công**:
  - Sử dụng `redirect()->route('tasks.index')->with('success', 'Task created successfully!')`.

### Sửa Task:
- **Lấy Task cụ thể**: Nhận đối tượng `Task $task` thông qua route model binding.
- **Kiểm tra và xác thực dữ liệu**:
  - Sử dụng `$request->validate()` để kiểm tra:
    - `title` là chuỗi bắt buộc, tối đa 255 ký tự.
    - `description` là chuỗi tùy chọn.
    - `due_date` là ngày hợp lệ (tùy chọn).
    - `is_completed` là kiểu boolean (tùy chọn).
    - `list_id` là bắt buộc và phải tồn tại trong bảng `lists`.
- **Cập nhật Task với dữ liệu đã xác thực**:
  - Sử dụng `$task->update($validated)` để cập nhật thông tin.
- **Chuyển hướng về trang `tasks.index` với thông báo thành công**:

## Giao diện thực tế

### Giao diện màn hình chính

<p align="start">
    <img src="Image-report/Dashboard.png" alt="Giao diện Dashboard" width="900"/>
    <img src="Image-report/Dashboard-reponsive.png" alt="Giao diện Dashboard Responsive" width="250"/>
</p>

### Giao diện danh sách List
<p align="start">
    <img src="Image-report/List.png" alt="Giao diện List" width="900"/>
    <img src="Image-report/List-Reponsive.png" alt="Giao diện List Responsive" width="250"/>
</p>

### Giao diện danh sách Task
<p align="start">
    <img src="Image-report/Task.png" alt="Giao diện Task" width="900"/>
    <img src="Image-report/Task-Responsive.png" alt="Giao diện Task Responsive" width="250"/>
</p>

### Giao diện thêm List
<p align="start">
    <img src="Image-report/Add-List.png" alt="Giao diện Thêm List" width="250"/>
</p>

### Giao diện thêm Task
<p align="start">
    <img src="Image-report/Add-Task.png" alt="Giao diện Thêm Task" width="250"/>
</p>

## Code minh họa

## Link Repo

- [https://github.com/hieule0208/To-do-app-Laravel](https://github.com/hieule0208/To-do-app-Laravel)

## Link Deploy

- [https://expert-cod-x5r74j7wx6ggfp977.github.dev/](https://expert-cod-x5r74j7wx6ggfp977.github.dev/)
