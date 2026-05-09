# Họ và tên: Lương Văn Học - MSSV: K225480106025
# lớp: K58KTP
# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
# Bài tập 02: SỬ DỤNG DJANGO ĐỂ TẠO WEB QUẢN LÝ TIỆM CẦM ĐỒ



# Bài tập 02 - Sử dụng Django để tạo web quản lý tiệm cầm đồ

## 1. Mục tiêu
Xây dựng website quản lý tiệm cầm đồ bằng **Django**, chạy trong **Docker**, sử dụng **MariaDB** làm cơ sở dữ liệu và **phpMyAdmin** để kiểm tra dữ liệu.

---

## 2. Chuẩn bị môi trường

### 2.1 Cập nhật hệ thống
```bash
sudo apt update && sudo apt upgrade -y
```

### 2.2 Cài Docker
```bash
sudo apt install -y docker.io docker-compose-plugin
```

### 2.3 Kiểm tra Docker
```bash
docker --version
docker compose version
```

### 2.4 Khởi động Docker
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### 2.5 Thêm user vào nhóm docker
```bash
sudo usermod -aG docker $USER
```

Sau khi chạy lệnh trên, thoát SSH và đăng nhập lại để quyền có hiệu lực.

---

## 3. Tạo thư mục dự án

```bash
mkdir -p ~/shopcamdo
cd ~/shopcamdo
```

Kiểm tra thư mục hiện tại:

```bash
pwd
```

Kết quả mong đợi:

```bash
/home/admin1/shopcamdo
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/3dae880e-971b-4ae2-9743-3fca4d70c010" />


## 4. Tạo cấu trúc ban đầu

Tạo các file cần thiết:

```bash
touch Dockerfile requirements.txt docker-compose.yml
```

Tạo các thư mục cần thiết:

```bash
mkdir -p config pawn/templates/pawn
```
<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/9f6bb87a-d508-4a52-9910-2c5ce7ff1764" />


## 5. Tạo file `requirements.txt`

```
nano requirements.txt
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/228891b6-c3ba-42c2-8642-75973514290b" />

Nội dung file `requirements.txt`:

```txt
Django==5.0.6
mysqlclient==2.2.4
python-dotenv==1.0.1
Pillow==10.4.0
```

<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/2ab21c6a-cdfd-4042-a885-e476be55c965" />

## 6. Tạo file `Dockerfile`

``` 
nano Dockerfile
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/f7e9d1da-6ba5-42c3-a1d1-d3782bc250b4" />




Nội dung file `Dockerfile`:

```dockerfile
FROM python:3.12-slim

RUN apt-get update && apt-get install -y \
    build-essential \
    default-libmysqlclient-dev \
    pkg-config \
    gcc \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

COPY . /app/

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/2593bc89-61cf-4dee-a05c-082dfdf1ea08" />

## 7. Tạo file `docker-compose.yml`

```
nano docker-compose.yml
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/1babe9dc-3f74-494e-8280-621c94dd00a2" />

Nội dung file `docker-compose.yml`:

```yaml
services:
  db:
    image: mariadb:11.4
    container_name: shopcamdo_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass123
      MYSQL_DATABASE: shopcamdo_db
      MYSQL_USER: shopuser
      MYSQL_PASSWORD: shoppass123
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: shopcamdo_phpmyadmin
    restart: always
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
    ports:
      - "8080:80"
    depends_on:
      - db

  web:
    build: .
    container_name: shopcamdo_web
    restart: always
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db

volumes:
  db_data:
```

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/e02934af-0e7a-4bb4-adb0-9df847c18673" />

## 8. Khởi tạo Django project

Chạy lệnh sau để tạo project Django:

```bash
docker compose run --rm web django-admin startproject config .
```
<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/9c7bbf04-b819-4bd7-b243-38e4c4d3c8a4" />

Sau đó kiểm tra:

```bash
ls -la
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/ba598c52-9269-45e4-ad7c-acb03ebda496" />


Kết quả mong đợi có các file/thư mục như:

- `manage.py`
- `config/`
- `Dockerfile`
- `docker-compose.yml`
- `requirements.txt`

## 7. Tạo ứng dụng Django

### 7.1 Tạo app `core`

Chạy lệnh sau:

```bash
docker compose run --rm web python manage.py startapp core
```

<img width="1108" height="640" alt="image" src="https://github.com/user-attachments/assets/d9ec8810-675c-4440-abb6-27ae8119cdd0" />

### 7.2 Kiểm tra kết quả

Sau khi chạy xong, kiểm tra thư mục dự án:

```bash
ls -la
```

Kết quả có thêm thư mục:

```bash
core/
```
<img width="1102" height="641" alt="image" src="https://github.com/user-attachments/assets/0413015a-30f0-4034-b18e-f4e9b3a74fc6" />


## 8. Sửa file `config/settings.py`

Sau khi tạo app `core`, cần khai báo app này trong Django để hệ thống nhận diện được.

### 8.1 Mở file cấu hình

```bash
nano config/settings.py
```

<img width="1106" height="638" alt="image" src="https://github.com/user-attachments/assets/0edb81f0-2c3f-43f5-a2bc-c8a24de23950" />

### 8.2 Thêm `core` vào `INSTALLED_APPS`

Tìm đoạn `INSTALLED_APPS` và chỉnh thành:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',
]
```

<img width="1980" height="1080" alt="image" src="https://github.com/user-attachments/assets/acd5c4f7-1173-4621-89e1-26c8f8c41d76" />

### 8.3 Cấu hình cơ sở dữ liệu MariaDB

Tìm phần `DATABASES` và thay bằng:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'shopcamdo_db',
        'USER': 'shopuser',
        'PASSWORD': 'shoppass123',
        'HOST': 'db',
        'PORT': '3306',
    }
}
```

<img width="1104" height="639" alt="image" src="https://github.com/user-attachments/assets/e6916ea3-2ef7-4b03-af36-242b91a674bd" />

### 8.4 Cấu hình static files và media files

Thêm ở cuối file:

```python
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

MEDIA_URL = 'media/'
MEDIA_ROOT = BASE_DIR / 'media'

ALLOWED_HOSTS = ['*']
```

<img width="1101" height="639" alt="image" src="https://github.com/user-attachments/assets/4af56819-ccee-49e9-ae65-93424da1bdc9" />

### 8.5 Ý nghĩa các cấu hình
- `INSTALLED_APPS`: khai báo app `core` để Django nhận biết.
- `DATABASES`: kết nối Django với MariaDB chạy trong Docker.
- `STATIC_URL`, `MEDIA_URL`: cấu hình file tĩnh và file upload.
- `ALLOWED_HOSTS`: cho phép truy cập từ bên ngoài khi chạy trên VM hoặc Cloudflare Tunnel.

### 8.6 Lưu file
Sau khi sửa xong:
- Nhấn `Ctrl + O` để lưu
- Nhấn `Enter` để xác nhận
- Nhấn `Ctrl + X` để thoát

### 8.7 Lưu ý
Nếu gặp lỗi `Permission denied` khi lưu file, hãy chạy:

```bash
sudo chown -R admin1:admin1 ~/shopcamdo
```

Sau đó mở lại file và lưu lại.

## 9. Tạo models theo CSDL của bạn

Trong bước này, tạo các model Django tương ứng với các bảng trong cơ sở dữ liệu đã thiết kế.

### 9.1 Mở file `core/models.py`

```bash
nano core/models.py
```

### 9.2 Nội dung file `core/models.py`

```python
from django.db import models


class KhachHang(models.Model):
    TRANG_THAI_CHOICES = [
        ('binh_thuong', 'Bình thường'),
        ('no_xau', 'Nợ xấu'),
    ]

    ma_kh = models.AutoField(primary_key=True)
    cccd = models.CharField(max_length=20, unique=True)
    ho_ten = models.CharField(max_length=200)
    sdt = models.CharField(max_length=20, unique=True)
    dia_chi = models.TextField()
    ngay_sinh = models.DateField()
    gioi_tinh = models.CharField(max_length=10)
    trang_thai = models.CharField(max_length=20, choices=TRANG_THAI_CHOICES, default='binh_thuong')

    def __str__(self):
        return f"{self.ho_ten} ({self.cccd})"


class NhanVien(models.Model):
    ma_nv = models.AutoField(primary_key=True)
    ho_ten = models.CharField(max_length=200)
    sdt = models.CharField(max_length=20)
    dia_chi = models.TextField()
    cccd = models.CharField(max_length=20, unique=True)
    tai_khoan = models.CharField(max_length=100, unique=True)
    chuc_vu = models.CharField(max_length=100)
    ngay_vao_lam = models.DateField()

    def __str__(self):
        return f"{self.ho_ten} - {self.chuc_vu}"


class LoaiTaiSan(models.Model):
    ma_loai_tai_san = models.AutoField(primary_key=True)
    ten_loai = models.CharField(max_length=100, unique=True)
    mo_ta = models.TextField(blank=True, null=True)

    def __str__(self):
        return self.ten_loai


class TaiSan(models.Model):
    TRANG_THAI_CHOICES = [
        ('dang_kiem_tra', 'Đang kiểm tra'),
        ('da_cam', 'Đã cầm'),
        ('da_tra_lai', 'Đã trả lại'),
        ('thanh_ly', 'Thanh lý'),
    ]

    ma_tai_san = models.AutoField(primary_key=True)
    ma_kh = models.ForeignKey(KhachHang, on_delete=models.CASCADE)
    ma_loai_tai_san = models.ForeignKey(LoaiTaiSan, on_delete=models.PROTECT)
    ten_tai_san = models.CharField(max_length=200)
    mo_ta = models.TextField(blank=True, null=True)
    gia_tri_tham_dinh = models.DecimalField(max_digits=12, decimal_places=2)
    trang_thai = models.CharField(max_length=30, choices=TRANG_THAI_CHOICES, default='dang_kiem_tra')
    so_luong = models.IntegerField(default=1)
    anh_tai_san = models.ImageField(upload_to='tai_san/', blank=True, null=True)
    tinh_trang_tai_san = models.TextField(blank=True, null=True)
    vi_tri_luu_kho = models.CharField(max_length=200, blank=True, null=True)

    def __str__(self):
        return self.ten_tai_san


class HopDong(models.Model):
    TRANG_THAI_CHOICES = [
        ('dang_hieu_luc', 'Đang hiệu lực'),
        ('qua_han', 'Quá hạn'),
        ('da_tat_toan', 'Đã tất toán'),
        ('thanh_ly', 'Thanh lý'),
    ]

    ma_hop_dong = models.AutoField(primary_key=True)
    ma_kh = models.ForeignKey(KhachHang, on_delete=models.CASCADE)
    ma_nv = models.ForeignKey(NhanVien, on_delete=models.PROTECT)
    ma_tai_san = models.ForeignKey(TaiSan, on_delete=models.PROTECT)

    so_tien_vay = models.DecimalField(max_digits=12, decimal_places=2)
    lai_suat = models.DecimalField(max_digits=5, decimal_places=2)
    ngay_vay = models.DateField()
    ngay_den_han = models.DateField()
    ghi_chu = models.TextField(blank=True, null=True)

    da_tra = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    tien_lai_phai_tra = models.DecimalField(max_digits=12, decimal_places=2, default=0)
    tong_tien_phai_tra = models.DecimalField(max_digits=12, decimal_places=2, default=0)

    trang_thai_hop_dong = models.CharField(max_length=20, choices=TRANG_THAI_CHOICES, default='dang_hieu_luc')

    def __str__(self):
        return f"HĐ-{self.ma_hop_dong} - {self.ma_kh.ho_ten}"


class ThanhToan(models.Model):
    HINH_THUC_CHOICES = [
        ('tien_mat', 'Tiền mặt'),
        ('chuyen_khoan', 'Chuyển khoản'),
    ]

    LOAI_THANH_TOAN_CHOICES = [
        ('tra_lai', 'Trả lãi'),
        ('tra_goc', 'Trả gốc'),
        ('tat_toan', 'Tất toán'),
    ]

    ma_thanh_toan = models.AutoField(primary_key=True)
    ma_hop_dong = models.ForeignKey(HopDong, on_delete=models.CASCADE)
    ma_nv = models.ForeignKey(NhanVien, on_delete=models.PROTECT)
    ngay_thanh_toan = models.DateTimeField(auto_now_add=True)
    so_tien = models.DecimalField(max_digits=12, decimal_places=2)
    noi_dung = models.TextField(blank=True, null=True)
    hinh_thuc_tt = models.CharField(max_length=20, choices=HINH_THUC_CHOICES)
    loai_thanh_toan = models.CharField(max_length=20, choices=LOAI_THANH_TOAN_CHOICES)

    def __str__(self):
        return f"TT-{self.ma_thanh_toan} - HĐ-{self.ma_hop_dong.ma_hop_dong}"
```

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/10a613a5-8eb0-458c-b67b-7d7e1d0fca86" />

## 10. Đăng ký models trong admin

Sau khi tạo models, cần đăng ký chúng trong trang quản trị Django để có thể thêm/sửa/xóa dữ liệu dễ dàng.

### 10.1 Mở file `core/admin.py`

```bash
nano core/admin.py
```

### 10.2 Nội dung file `core/admin.py`

```python
from django.contrib import admin
from .models import KhachHang, NhanVien, LoaiTaiSan, TaiSan, HopDong, ThanhToan


@admin.register(KhachHang)
class KhachHangAdmin(admin.ModelAdmin):
    list_display = ('ma_kh', 'ho_ten', 'cccd', 'sdt', 'trang_thai')
    search_fields = ('ho_ten', 'cccd', 'sdt')
    list_filter = ('trang_thai', 'gioi_tinh')


@admin.register(NhanVien)
class NhanVienAdmin(admin.ModelAdmin):
    list_display = ('ma_nv', 'ho_ten', 'chuc_vu', 'sdt', 'tai_khoan')
    search_fields = ('ho_ten', 'cccd', 'tai_khoan')


@admin.register(LoaiTaiSan)
class LoaiTaiSanAdmin(admin.ModelAdmin):
    list_display = ('ma_loai_tai_san', 'ten_loai')


@admin.register(TaiSan)
class TaiSanAdmin(admin.ModelAdmin):
    list_display = ('ma_tai_san', 'ten_tai_san', 'ma_kh', 'ma_loai_tai_san', 'gia_tri_tham_dinh', 'trang_thai')
    search_fields = ('ten_tai_san', 'ma_kh__ho_ten')


@admin.register(HopDong)
class HopDongAdmin(admin.ModelAdmin):
    list_display = ('ma_hop_dong', 'ma_kh', 'ma_nv', 'ma_tai_san', 'so_tien_vay', 'ngay_den_han', 'trang_thai_hop_dong')
    list_filter = ('trang_thai_hop_dong',)
    search_fields = ('ma_kh__ho_ten', 'ma_tai_san__ten_tai_san')


@admin.register(ThanhToan)
class ThanhToanAdmin(admin.ModelAdmin):
    list_display = ('ma_thanh_toan', 'ma_hop_dong', 'ma_nv', 'ngay_thanh_toan', 'so_tien', 'hinh_thuc_tt', 'loai_thanh_toan')
    list_filter = ('hinh_thuc_tt', 'loai_thanh_toan')
```

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/b7632bcb-8f7a-4c59-89cd-9f880edd31f5" />

## 11. Tạo views

Tạo trang chủ để hiển thị dữ liệu tổng quan của hệ thống.

### 11.1 Mở file `core/views.py`

```bash
nano core/views.py
```

### 11.2 Nội dung file `core/views.py`

```python
from django.shortcuts import render
from django.utils import timezone
from .models import KhachHang, HopDong, ThanhToan


def home(request):
    today = timezone.now().date()

    hop_dong_dang_hieu_luc = HopDong.objects.filter(trang_thai_hop_dong='dang_hieu_luc')
    hop_dong_qua_han = HopDong.objects.filter(trang_thai_hop_dong='qua_han')
    khach_hang_no_xau = KhachHang.objects.filter(trang_thai='no_xau')
    thanh_toan_moi_nhat = ThanhToan.objects.order_by('-ngay_thanh_toan')[:10]

    context = {
        'today': today,
        'hop_dong_dang_hieu_luc': hop_dong_dang_hieu_luc,
        'hop_dong_qua_han': hop_dong_qua_han,
        'khach_hang_no_xau': khach_hang_no_xau,
        'thanh_toan_moi_nhat': thanh_toan_moi_nhat,
    }
    return render(request, 'core/home.html', context)
```

<img width="1920" height="1030" alt="image" src="https://github.com/user-attachments/assets/0fd752fb-9c92-45b6-855d-ac7ecbc886e3" />

## 12. Tạo file URL cho app

### 12.1 Tạo file `core/urls.py`

```bash
nano core/urls.py
```

### 12.2 Nội dung file `core/urls.py`

```python
from django.urls import path
from .views import home

urlpatterns = [
    path('', home, name='home'),
]
```

<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/867ed4cb-8861-44d2-a43a-868373c9cd90" />

## 13. Sửa file URL chính của project

### 13.1 Mở file `config/urls.py`

```bash
nano config/urls.py
```

### 13.2 Nội dung file `config/urls.py`

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('core.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/ea20a2be-2749-4566-b4bd-6055a1981e46" />

## 14. Tạo giao diện trang chủ

### 14.1 Tạo thư mục template

```bash
mkdir -p core/templates/core
```

### 14.2 Tạo file `core/templates/core/home.html`

```bash
nano core/templates/core/home.html
```

### 14.3 Nội dung file `home.html`

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Quản lý tiệm cầm đồ</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 30px; background: #f4f6f8; }
        .container { max-width: 1300px; margin: auto; background: white; padding: 20px; border-radius: 10px; }
        h1 { color: #222; }
        h2 { margin-top: 30px; color: #333; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 10px; text-align: left; }
        th { background: #007bff; color: white; }
        .badge-red { color: red; font-weight: bold; }
        .badge-green { color: green; font-weight: bold; }
    </style>
</head>
<body>
<div class="container">
    <h1>Hệ thống quản lý tiệm cầm đồ</h1>
    <p>Ngày hiện tại: {{ today }}</p>

    <h2>Hợp đồng đang hiệu lực</h2>
    <table>
        <tr>
            <th>Mã HĐ</th>
            <th>Khách hàng</th>
            <th>Tài sản</th>
            <th>Số tiền vay</th>
            <th>Ngày đến hạn</th>
            <th>Trạng thái</th>
        </tr>
        {% for hd in hop_dong_dang_hieu_luc %}
        <tr>
            <td>{{ hd.ma_hop_dong }}</td>
            <td>{{ hd.ma_kh.ho_ten }}</td>
            <td>{{ hd.ma_tai_san.ten_tai_san }}</td>
            <td>{{ hd.so_tien_vay }}</td>
            <td>{{ hd.ngay_den_han }}</td>
            <td class="badge-green">{{ hd.get_trang_thai_hop_dong_display }}</td>
        </tr>
        {% empty %}
        <tr><td colspan="6">Không có dữ liệu</td></tr>
        {% endfor %}
    </table>

    <h2>Hợp đồng quá hạn</h2>
    <table>
        <tr>
            <th>Mã HĐ</th>
            <th>Khách hàng</th>
            <th>Tài sản</th>
            <th>Số tiền vay</th>
            <th>Ngày đến hạn</th>
            <th>Trạng thái</th>
        </tr>
        {% for hd in hop_dong_qua_han %}
        <tr>
            <td>{{ hd.ma_hop_dong }}</td>
            <td>{{ hd.ma_kh.ho_ten }}</td>
            <td>{{ hd.ma_tai_san.ten_tai_san }}</td>
            <td>{{ hd.so_tien_vay }}</td>
            <td>{{ hd.ngay_den_han }}</td>
            <td class="badge-red">{{ hd.get_trang_thai_hop_dong_display }}</td>
        </tr>
        {% empty %}
        <tr><td colspan="6">Không có dữ liệu</td></tr>
        {% endfor %}
    </table>

    <h2>Khách hàng nợ xấu</h2>
    <table>
        <tr>
            <th>Mã KH</th>
            <th>Họ tên</th>
            <th>CCCD</th>
            <th>SĐT</th>
            <th>Trạng thái</th>
        </tr>
        {% for kh in khach_hang_no_xau %}
        <tr>
            <td>{{ kh.ma_kh }}</td>
            <td>{{ kh.ho_ten }}</td>
            <td>{{ kh.cccd }}</td>
            <td>{{ kh.sdt }}</td>
            <td class="badge-red">{{ kh.get_trang_thai_display }}</td>
        </tr>
        {% empty %}
        <tr><td colspan="5">Không có khách hàng nợ xấu</td></tr>
        {% endfor %}
    </table>

    <h2>10 thanh toán gần nhất</h2>
    <table>
        <tr>
            <th>Mã TT</th>
            <th>Hợp đồng</th>
            <th>Số tiền</th>
            <th>Hình thức</th>
            <th>Loại thanh toán</th>
            <th>Ngày thanh toán</th>
        </tr>
        {% for tt in thanh_toan_moi_nhat %}
        <tr>
            <td>{{ tt.ma_thanh_toan }}</td>
            <td>{{ tt.ma_hop_dong.ma_hop_dong }}</td>
            <td>{{ tt.so_tien }}</td>
            <td>{{ tt.get_hinh_thuc_tt_display }}</td>
            <td>{{ tt.get_loai_thanh_toan_display }}</td>
            <td>{{ tt.ngay_thanh_toan }}</td>
        </tr>
        {% empty %}
        <tr><td colspan="6">Không có dữ liệu</td></tr>
        {% endfor %}
    </table>
</div>
</body>
</html>
```

<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/a4ce75af-42a5-4d61-ba82-0bf856110ce0" />

## 15. Tạo migration cho các model

Sau khi đã tạo xong `models.py`, Django cần sinh các file migration để tạo bảng trong cơ sở dữ liệu MariaDB.

### 15.1 Chạy lệnh tạo migration

```bash
docker compose exec web python manage.py makemigrations
```

### 15.2 Kết quả mong đợi
Nếu thành công, Django sẽ tạo các file migration cho app `core`.

---

## 16. Áp migration vào database

Sau khi tạo migration, tiếp tục chạy lệnh sau để tạo bảng trong MariaDB:

```bash
docker compose exec web python manage.py migrate
```

### 16.1 Ý nghĩa
- Lệnh này sẽ tạo toàn bộ bảng trong database.
- Các bảng sẽ tương ứng với các model đã khai báo trong `core/models.py`.

---

## 17. Tạo tài khoản quản trị

Để đăng nhập vào trang quản trị Django, cần tạo một superuser.

### 17.1 Chạy lệnh tạo superuser

```bash
docker compose exec web python manage.py createsuperuser
```

### 17.2 Nhập thông tin
Khi được yêu cầu, nhập:
- `username`
- `email`
- `password`

Ví dụ:
- Username: `admin`
- Email: `admin@gmail.com`
- Password: `Admin@12345`

---

## 18. Chạy lại container nếu cần

Nếu container chưa chạy, bật lại bằng lệnh sau:

```bash
docker compose up -d
```

### 18.1 Kiểm tra trạng thái container

```bash
docker compose ps
```

### 18.2 Ý nghĩa
- `db`: container MariaDB
- `phpmyadmin`: container giao diện quản lý database
- `web`: container Django

---

## 19. Truy cập website Django

Sau khi container chạy thành công, mở trình duyệt và truy cập website Django.

### 19.1 Truy cập trang chủ

```text
http://IP_MAY_UBUNTU:8000
```

Trong đó `IP_MAY_UBUNTU` là địa chỉ IP của máy ảo Ubuntu.

### 19.2 Kiểm tra IP của máy ảo

```bash
hostname -I
```

---

## 20. Truy cập trang admin

Trang quản trị Django dùng để nhập và quản lý dữ liệu trong các bảng.

### 20.1 URL trang admin

```text
http://IP_MAY_UBUNTU:8000/admin
```

### 20.2 Đăng nhập
Sử dụng tài khoản superuser đã tạo ở bước 17 để đăng nhập.

### 20.3 Chức năng
Tại đây có thể:
- thêm khách hàng
- thêm nhân viên
- thêm loại tài sản
- thêm tài sản
- thêm hợp đồng
- thêm thanh toán


































<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cdbde295-a515-4c86-87ce-7db811f3444e" />





















