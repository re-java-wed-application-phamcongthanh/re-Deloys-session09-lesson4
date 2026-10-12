# Bài 4: Lưu Trữ Dữ Liệu Đơn Giản Với Docker Volume

## 🎯 1. Mục Tiêu
- Tạo và quản lý một Docker Named Volume (`easy-volume`) độc lập với vòng đời của Container.
- Chứng minh tính bền vững (Persistence) của dữ liệu: Khi container kết thúc và bị hủy bỏ (`--rm`), dữ liệu đã ghi vào volume vẫn được bảo toàn nguyên vẹn và có thể truy cập bởi một container hoàn toàn mới khác.

---

## 🛠️ 2. Quy Trình Thực Hiện & Lệnh CLI

### Bước 1: Khởi tạo Named Volume
Khởi tạo volume có tên `easy-volume`:
```bash
docker volume create easy-volume
```

**Kết quả:**
```text
easy-volume
```

Kiểm tra sự tồn tại của volume trong danh sách hệ thống:
```bash
docker volume ls
```

*(Dòng thông tin của volume mới tạo)*:
```text
DRIVER    VOLUME NAME
local     easy-volume
```

Kiểm tra thông tin chi tiết (Mountpoint trên host Docker):
```bash
docker volume inspect easy-volume
```
```json
[
    {
        "CreatedAt": "2026-10-12T05:52:04Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/easy-volume/_data",
        "Name": "easy-volume",
        "Options": null,
        "Scope": "local"
    }
]
```

---

### Bước 2: Dùng Container thứ nhất ghi dữ liệu vào Volume
Chạy một container `alpine` tạm thời gắn kết (mount) volume `easy-volume` vào thư mục `/data`, ghi chuỗi văn bản vào file `/data/test.txt` và tự động hủy container sau khi xong (`--rm`):
```bash
docker run --rm -v easy-volume:/data alpine sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"
```

**Giải thích tham số:**
- `--rm`: Tự động xóa sạch container ngay khi tiến trình bên trong kết thúc (chứng minh container không còn tồn tại).
- `-v easy-volume:/data`: Gắn volume `easy-volume` từ máy host vào thư mục đích `/data` bên trong container.
- `sh -c "echo 'Luu tru du lieu Docker' > /data/test.txt"`: Thực thi lệnh ghi nội dung chuỗi vào file `/data/test.txt`.

---

### Bước 3: Dùng Container thứ hai đọc dữ liệu từ Volume
Khởi chạy một container `alpine` hoàn toàn mới khác, tiếp tục mount volume `easy-volume` vào `/data` và đọc nội dung file:
```bash
docker run --rm -v easy-volume:/data alpine cat /data/test.txt
```

**Kết quả hiển thị trên Terminal:**
```text
Luu tru du lieu Docker
```

> ✅ **Xác nhận**: Dù container thứ nhất đã bị hủy hoàn toàn, container thứ hai vẫn đọc được chính xác 100% nội dung `"Luu tru du lieu Docker"` được lưu trữ bên trong `easy-volume`.

---

### Bước 4: Hướng dẫn dọn dẹp Volume (Sau khi hoàn tất bài tập)
Khi không còn nhu cầu sử dụng, có thể xóa volume bằng lệnh:
```bash
docker volume rm easy-volume
```

---

## 💡 3. Ý Nghĩa Cốt Lõi Về Docker Storage
1. **Container Ephemeral (Tính ngắn hạn)**: Mọi dữ liệu ghi trực tiếp lên tầng writable layer của container sẽ bị mất vĩnh viễn khi container bị xóa (`docker rm`).
2. **Docker Volumes (Tính bền vững)**: Volumes được quản lý độc lập tại `/var/lib/docker/volumes/`, không bị phụ thuộc vào vòng đời của bất kỳ container nào, là giải pháp chuẩn công nghiệp để lưu trữ dữ liệu Database (MySQL, PostgreSQL, MongoDB), Logs hoặc Media files.
