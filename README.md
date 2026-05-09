# Họ và tên: Lương Văn Học - MSSV: K225480106025
# lớp: K58KTP
# Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421
# Bài tập 02: SỬ DỤNG DJANGO ĐỂ TẠO WEB QUẢN LÝ TIỆM CẦM ĐỒ



## BƯỚC 1: TẠO THƯ MỤC DỰ ÁN

Trên Ubuntu VM của bạn, vào thư mục làm việc rồi tạo project:


```
mkdir pawnshop_project
cd pawnshop_project
```
Kiểm tra:
```
pwd
ls
```
Phải đang đứng trong thư mục ``` pawnshop_project ```
## **BƯỚC 2: TẠO FILE** ``` requirements.txt ```
Tạo file:

```
nano requirements.txt
```
<img width="1103" height="640" alt="image" src="https://github.com/user-attachments/assets/756c99a3-07d0-472e-8ec3-05cceef01d23" />

Dán nội dung này:

```
Django>=5.0
mysqlclient
Pillow
python-dotenv
```
Giải thích:

- Django: framework web
- mysqlclient: kết nối MariaDB
- Pillow: xử lý ảnh
- python-dotenv: đọc biến môi trường nếu cần

<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/b3843552-1a9b-4a51-a859-1a298abd9726" />

- Lưu file:
```
Ctrl + O
```
- Enter
```
Ctrl + X
```


Tạo file:

bash
nano requirements.txt

## BƯỚC 3: TẠO Dockerfile

Tạo file:
```
nano Dockerfile
```

<img width="1102" height="638" alt="image" src="https://github.com/user-attachments/assets/346a53dd-fbfd-4049-8609-89a60ffb4e45" />

Dán nội dung này:

```
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && apt-get install -y \
    default-libmysqlclient-dev \
    build-essential \
    pkg-config \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

```
<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/e68b4afc-7da7-4d87-8a11-10a521ed51a8" />

Ý nghĩa từng phần:
- lấy image Python
- cài thư viện để mysqlclient chạy
- copy requirements
- cài package
- copy code vào container
- chạy Django server




## BƯỚC 4: TẠO docker-compose.yml
Tạo file:

```
nano docker-compose.yml
```
<img width="1101" height="636" alt="image" src="https://github.com/user-attachments/assets/70efd218-380f-49dd-b8a8-cdc816c1f1c1" />


Dán nội dung này:

```
services:
  db:
    image: mariadb:11
    container_name: pawn_db
    restart: always
    environment:
      MARIADB_ROOT_PASSWORD: rootpass
      MARIADB_DATABASE: pawnshop
      MARIADB_USER: pawnuser
      MARIADB_PASSWORD: pawnpass
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: pawn_phpmyadmin
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
    container_name: pawn_web
    restart: always
    command: sh -c "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    depends_on:
      - db

volumes:
  db_data:

```
<img width="1103" height="639" alt="image" src="https://github.com/user-attachments/assets/69051f3e-63a2-458c-a458-c68e4b5e6f30" />

Ý nghĩa:
- db: chạy MariaDB
- phpmyadmin: xem database bằng trình duyệt
- web: chạy Django

## BƯỚC 5: TẠO PROJECT DJANGO
Bây giờ tạo project Django:

```
docker compose run --rm web django-admin startproject config .
```

Sau lệnh này, bạn sẽ có:

- manage.py
- thư mục config/

Kiểm tra:

```
ls
```
Phải thấy:

- Dockerfile
- docker-compose.yml
- requirements.txt
- manage.py
- config/

<img width="1106" height="639" alt="image" src="https://github.com/user-attachments/assets/c5177ee4-ccb8-4c08-8fb8-ca381a0920bb" />


## BƯỚC 6: TẠO APP CHÍNH
Tạo app tên main:

```
docker compose run --rm web python manage.py startapp main

```
Sau đó kiểm tra:

```
ls main
```
Sẽ thấy các file như:

- models.py
- views.py
- admin.py
- apps.py

<img width="1109" height="642" alt="image" src="https://github.com/user-attachments/assets/69148002-91a3-4b96-ba23-5df8ace2f50e" />

## BƯỚC 7: CHỈNH ``` settings.py ```
Mở file:

```
nano config/settings.py
```
### 7.1. Thêm app main
Tìm ```INSTALLED_APPS``` và thêm ``` 'main', ```:

<img width="1105" height="642" alt="image" src="https://github.com/user-attachments/assets/852b4d53-00c2-4d11-be56-efd707a6d2e5" />

```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'main',
]
```

### 7.2. Đổi database sang MariaDB
Tìm phần DATABASES và thay bằng:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'pawnshop',
        'USER': 'pawnuser',
        'PASSWORD': 'pawnpass',
        'HOST': 'db',
        'PORT': '3306',
        'OPTIONS': {
            'charset': 'utf8mb4',
        },
    }
}
```

<img width="1106" height="641" alt="image" src="https://github.com/user-attachments/assets/48fee802-0669-4154-a1a6-52bf3773cd49" />

### 7.3. Cho phép dùng ảnh
Cuối file thêm:

```
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

```

<img width="1103" height="641" alt="image" src="https://github.com/user-attachments/assets/6df83fab-dc55-4ec2-9085-3dd18bf9456d" />

Lưu lại.



















<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cdbde295-a515-4c86-87ce-7db811f3444e" />





















