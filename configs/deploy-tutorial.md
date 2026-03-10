# Mục lục
1. [Tạo user và gán quyền sudo trên VPS](#tạo-user-và-gán-quyền-sudo-trên-vps)
2. [Các bước kết nối máy tính cá nhân đến VPS và VPS đến github (gitlab) qua SSH](#các-bước-kết-nối-máy-tính-cá-nhân-đến-vps-và-vps-đến-github-gitlab-qua-ssh)
3. [Một số câu lệnh hay dùng trên Ubuntu](#một-số-câu-lệnh-hay-dùng-trên-ubuntu)
4. [Cài đặt NodeJS trên VPS sử dụng NVM (Node Version Manager)](#cài-đặt-nodejs-trên-vps-sử-dụng-nvm-node-version-manager)
5. [Deploy một ứng dụng html, css, js đơn giản lên VPS](#deploy-một-ứng-dụng-html-css-js-đơn-giản-lên-vps)
6. [Sử dụng Nginx làm reverse proxy để chạy ứng dụng](#sử-dụng-nginx-làm-reverse-proxy-để-chạy-ứng-dụng)
7. [Cấu hình HTTPS/SSL sử dụng Let's Encrypt và Certbot](#cấu-hình-httpsssl-sử-dụng-lets-encrypt-và-certbot)
8. [Deploy ứng dụng sử dụng Docker](#deploy-ứng-dụng-sử-dụng-docker)
---

# Tạo user và gán quyền sudo trên VPS
## Bước 1: Truy cập vào VPS bằng tài khoản root
Tham khảo hướng dẫn kết nối đến VPS qua SSH ở phần dưới (cách 1).

[Hướng dẫn kết nối đến VPS qua SSH](#các-bước-kết-nối-máy-tính-cá-nhân-đến-vps-và-vps-đến-github-gitlab-qua-ssh)

## Bước 2: Tạo user mới
```bash
adduser new_username
```

## Bước 3: Gán quyền sudo cho user mới
```bash
usermod -aG sudo new_username
```

## Bước 4: Switch sang user mới
```bash
su - new_username
```

# Các bước kết nối máy tính cá nhân đến VPS và VPS đến github (gitlab) qua SSH
## Bước 1: Kết nối máy tính cá nhân đến VPS
* Cách 1: Sử dụng thông tin đăng nhập (username, password) để kết nối đến VPS qua SSH
```bash
ssh username@vps_ip_address
```
Sau khi nhập xong, Sử dụng mật khẩu để đăng nhập vào VPS.

* Cách 2: Sử dụng SSH key để kết nối đến VPS qua SSH

1. Tạo SSH key trên máy tính cá nhân (nếu chưa có):
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. Tại bước save key, nhập đường dẫn để lưu key (mặc định là `~/.ssh/id_ed25519`) ví dụ: `~/.ssh/example-vps`

3. Cấu hình config SSH để dễ dàng kết nối đến VPS:
Tạo file `~/.ssh/config` nếu chưa tồn tại và thêm vào nội dung sau:
```bash
Host example-vps # Tên host tùy ý để dễ nhớ
HostName vps_ip_address # Địa chỉ IP của VPS
User username # Tên user đăng nhập trên VPS
IdentityFile ~/.ssh/example-vps # Đường dẫn đến private key đã tạo ở bước 2
```
Ví dụ khác config cho github (nếu muốn kết nối đến github qua SSH):
```bash
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_ed25519_github
```

4. Copy public key lên VPS:
Tạo file `~/.ssh/authorized_keys` trên VPS nếu chưa tồn tại và thêm public key vào file này:
```bash
cat ~/.ssh/example-vps.pub | ssh username@vps_ip_address "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

`Lưu ý`: Có thể sử dụng `ssh-copy-id` để copy public key lên VPS:
```bash
ssh-copy-id -i ~/.ssh/example-vps.pub username@vps_ip_address
```
Lệnh này sẽ tự động thêm public key vào file `~/.ssh/authorized_keys` trên VPS.

5. Kết nối đến VPS bằng SSH cho các lần sau:
Sử dụng tên host đã cấu hình trong file `~/.ssh/config` để kết nối:
```bash
ssh example-vps
```
Nếu cấu hình đúng, bạn sẽ đăng nhập vào VPS mà không cần nhập mật khẩu.

## Bước 2: Kết nối VPS đến github (gitlab) qua SSH
Tương tự như bước 1, Tạo SSH key trên VPS và cấu hình để kết nối đến github (gitlab):

1. Tạo SSH key trên VPS:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

2. Cấu hình config SSH trên VPS để kết nối đến github (gitlab):
Tạo file `~/.ssh/config` nếu chưa tồn tại và thêm vào nội dung sau
```bash
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_ed25519_github
```

3. Copy public key lên github (gitlab):
Sao chép nội dung của public key và thêm vào phần SSH keys trong cài đặt tài khoản github (gitlab)

4. Kiểm tra kết nối đến github (gitlab) từ VPS:
```bash
ssh -T git@github.com
```

# Một số câu lệnh hay dùng trên Ubuntu
## Các câu lệnh cơ bản:
* Tình trạng RAM: `free -h`
* Tình trạng DISK: `df -h`
* Tạo file: `touch file.txt`
* Tạo thư mục: `mkdir folder`
* Di chuyển: `cd folder`
* Xem danh sách file: `ls`
* Xem danh sách file ẩn: `ls -a`
* Xem nội dung file: `cat file.txt`
* Copy file: `cp file.txt file2.txt`
* Xóa file: `rm file.txt`
* Xóa thư mục: `rm -r folder`
* Xóa folder và tất cả nội dung bên trong mà không cần xác nhận: `rm -rf folder`
* Edit file: `nano file.txt`
* Restart ubuntu: `sudo reboot`

## Quản lý tường lửa UFW (Uncomplicated Firewall):
* Kiểm tra trạng thái tường lửa (nếu bật thì sẽ show port mở)
```bash
sudo ufw status
```
* Bật tường lửa
```bash
sudo ufw enable
```
* Yêu cầu tường lửa lên mỗi khi khởi động lại server
```bash
sudo systemctl enable ufw
```

* Mở port 22 (ssh)
```bash
sudo ufw allow ssh
```

* Mở port 3000
```bash
sudo ufw allow 3000
```

* Đóng và xóa rule mở port 3000
```bash
sudo ufw delete allow 3000
```

## Tạo ram swap
Bước 1: Tạo file swap 1GB
```bash
sudo fallocate -l 1G /swapfile
```

Bước 2: Cấp quyền cho file swap
```bash
sudo chmod 600 /swapfile
```

Bước 3: Thiết lập file swap
```bash
sudo mkswap /swapfile
```

Bước 4: Kích hoạt file swap
```bash
sudo swapon /swapfile
```

Bước 5: Thêm file swap vào fstab để tự động kích hoạt khi khởi động lại
```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

Bước 6: Kiểm tra trạng thái swap
```bash
sudo swapon --show
```

```bash
sudo free -h
```

# Cài đặt NodeJS trên VPS sử dụng NVM (Node Version Manager)
## Bước 1: Cập nhật hệ thống
```bash
sudo apt update
```
## Bước 2: Cài đặt NVM
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
```

``` bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

Có thể thay đổi phiên bản NVM nếu muốn bằng cách thay `v0.40.4` thành phiên bản mới nhất trên trang github của NVM (https://github.com/nvm-sh/nvm).

## Bước 3: Cài đặt phiên bản NodeJS mong muốn
```bash
nvm install 22
```

Hoặc phiên bản long term support (LTS) mới nhất:
```bash
nvm install --lts
```

Chuyển sang phiên bản NodeJS vừa cài đặt:
```bash
nvm use 22
```

Đặt mặc định phiên bản NodeJS vừa cài đặt:
```bash
nvm alias 22
```

# Deploy một ứng dụng html, css, js đơn giản lên VPS
Lưu ý: Ứng dụng này có file index.html làm root và chỉ sử dụng html, css, js thuần nên không cần cài đặt thêm bất kỳ phần mềm nào trên VPS.

## Bước 1: Clone code từ github (gitlab) về VPS
```bash
git clone ...
```
Sau đó `cd` vào thư mục chứa code và kiểm tra xem có file `index.html` hay không:

## Bước 2: Cài đặt thư viện `serve` để chạy ứng dụng (nếu chưa có)
```bash
npm install -g serve
```

## Bước 3: Chạy ứng dụng
Chạy ứng dụng trên port 3000:
```bash
serve -s . -l 3000
```

* Nếu muốn chạy ứng dụng ngầm (background) để có thể tắt terminal mà ứng dụng vẫn chạy, sử dụng `screen`:
1. Cài đặt `screen` nếu chưa có:
```bash
sudo apt install screen
```

2. Tạo một phiên `screen` mới:
```bash
screen -S myapp
```

3. Chạy ứng dụng trong phiên `screen` như bình thường:
```bash
serve -s . -l 3000
```

4. Để tách khỏi phiên `screen` mà không dừng ứng dụng, nhấn tổ hợp phím `Ctrl + A`, sau đó nhấn `D`.

5. Để quay lại phiên `screen` đang chạy ứng dụng, sử dụng lệnh:
```bash
screen -r myapp
```

6. Xóa phiên `screen` khi không còn cần thiết:
```bash
screen -X -S myapp quit
```

# Sử dụng Nginx làm reverse proxy để chạy ứng dụng
## Bước 1: Cài đặt Nginx
```bash
sudo apt install nginx
```

### Bước 2: Cho phép mở full nginx trên tường lửa UFW
```bash
sudo ufw allow 'Nginx Full'
```

### Bước 3: Cấu hình Nginx
Tạo file cấu hình mới cho ứng dụng:
```bash
sudo nano /etc/nginx/sites-available/myapp
```

Thêm nội dung sau vào file cấu hình:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://localhost:3000; # Địa chỉ và port của ứng dụng NodeJS
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Lưu ý: Thay `your_domain_or_ip` bằng tên miền hoặc địa chỉ IP của VPS.

### Bước 4: Kích hoạt cấu hình Nginx
Tạo liên kết từ file cấu hình vừa tạo đến thư mục `sites-enabled` để kích hoạt cấu hình:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
```

### Bước 5: Kiểm tra cấu hình Nginx
```bash
sudo nginx -t
```

### Bước 6: Khởi động lại Nginx để áp dụng cấu hình mới
```bash
sudo systemctl restart nginx
```

## Bước 7: Truy cập ứng dụng
Mở trình duyệt và truy cập vào `http://your_domain_or_ip`

# Cấu hình HTTPS/SSL sử dụng Let's Encrypt và Certbot
## Bước 1: Cài đặt certbot và plugin cho Nginx
```bash
sudo apt update
sudo apt install snapd
```

Cài certbot classic:
```bash
sudo snap install --classic certbot
```

## Bước 2: Tạo liên kết cho certbot
```bash
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot
```

## Bước 3: Chạy certbot
Lấy chứng chỉ và cài đặt tự động cho Nginx:
```bash
sudo certbot --nginx
```

Hoặc, nếu muốn lấy chứng chỉ mà không tự động cài đặt cho Nginx:
```bash
sudo certbot certonly --nginx
```

## Bước 4: Kiểm tra tự động gia hạn chứng chỉ
```bash
sudo certbot renew --dry-run
```

## Bước 5: Bổ sung Http2 vào cấu hình Nginx để cải thiện hiệu suất khi sử dụng HTTPS
Mở file /etc/nginx/sites-available/myapp và thay đổi dòng `listen 80;` thành:
```nginx
listen 80;
listen 443 ssl http2;
```

Sau đó khởi động lại Nginx để áp dụng thay đổi:
```bash
sudo systemctl restart nginx
```

# Deploy ứng dụng sử dụng Docker
## Bước 1: Cài đặt Docker
### Cài đặt Docker trên VPS
```bash
sudo apt update
sudo apt install docker.io
```

### Cài đặt Docker Compose (tùy chọn nhưng rất hữu ích)
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Kiểm tra phiên bản Docker
```bash
docker --version
docker-compose --version
```

### Cấp quyền cho user hiện tại (không cần sudo)
```bash
sudo usermod -aG docker $USER
```
Sau khi chạy lệnh này, cần đăng xuất và đăng nhập lại hoặc chạy:
```bash
newgrp docker
```

## Bước 2: Tạo Dockerfile
Tạo file `Dockerfile` trong thư mục gốc của ứng dụng:
```dockerfile
# Sử dụng Node.js image làm base
FROM node:22-alpine

# Thiết lập thư mục làm việc
WORKDIR /app

# Copy package.json và package-lock.json (nếu có)
COPY package*.json ./

# Cài đặt dependencies
RUN npm install

# Copy toàn bộ mã nguồn
COPY . .

# Expose port (thay 3000 bằng port của ứng dụng)
EXPOSE 3000

# Lệnh khởi động ứng dụng
CMD ["npm", "start"]
```

### Ví dụ Dockerfile cho ứng dụng HTML/CSS/JS đơn giản
```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install -g serve

COPY . .

EXPOSE 3000

CMD ["serve", "-s", ".", "-l", "3000"]
```

### Ví dụ Dockerfile cho ứng dụng Nginx
```dockerfile
FROM nginx:alpine

# Copy static files vào nginx
COPY . /usr/share/nginx/html/

# Copy cấu hình nginx (nếu có)
# COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Bước 3: Tạo file .dockerignore (tùy chọn)
Tạo file `.dockerignore` để loại bỏ các file không cần thiết khi build Docker image:
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.env.local
dist
build
```

## Bước 4: Build Docker Image
```bash
docker build -t myapp:1.0 .
```

Lưu ý: `myapp` là tên image, `1.0` là tag (phiên bản). Có thể sử dụng tag `latest` để chỉ định phiên bản mới nhất:
```bash
docker build -t myapp:latest .
```

### Kiểm tra image vừa tạo
```bash
docker images
```

## Bước 5: Chạy Docker Container
### Chạy container đơn giản
```bash
docker run -p 3000:3000 myapp:latest
```

### Chạy container ngầm (background)
```bash
docker run -d -p 3000:3000 --name myapp-container myapp:latest
```

Giải thích các tùy chọn:
* `-d`: Chạy container ở chế độ detached (background)
* `-p 3000:3000`: Map port từ container đến host (host_port:container_port)
* `--name myapp-container`: Đặt tên cho container
* `myapp:latest`: Tên và tag của image cần chạy

### Chạy container với các biến môi trường
```bash
docker run -d -p 3000:3000 --name myapp-container -e NODE_ENV=production -e DATABASE_URL=postgres://... myapp:latest
```

### Chạy container với volume (để lưu trữ dữ liệu)
```bash
docker run -d -p 3000:3000 --name myapp-container -v /data:/app/data myapp:latest
```

## Bước 6: Quản lý Docker Containers
### Xem danh sách các container đang chạy
```bash
docker ps
```

### Xem danh sách tất cả các container (bao gồm cả đã dừng)
```bash
docker ps -a
```

### Dừng container
```bash
docker stop myapp-container
```

### Khởi động lại container
```bash
docker start myapp-container
```

### Xóa container
```bash
docker rm myapp-container
```

### Xem logs của container
```bash
docker logs myapp-container
```

### Xem logs real-time
```bash
docker logs -f myapp-container
```

### Truy cập vào container (shell)
```bash
docker exec -it myapp-container /bin/sh
```

Hoặc với bash:
```bash
docker exec -it myapp-container bash
```

## Bước 7: Sử dụng Docker Compose (tùy chọn nhưng khuyến khích)
Tạo file `docker-compose.yml` trong thư mục gốc của ứng dụng:

### Ví dụ 1: Ứng dụng Node.js đơn giản
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    restart: always
```

### Ví dụ 2: Ứng dụng Node.js với Database
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:password@db:5432/dbname
    restart: always

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: always

volumes:
  db_data:
```

### Các lệnh Docker Compose phổ biến
```bash
# Build và chạy containers
docker-compose up -d

# Xem logs
docker-compose logs -f

# Dừng containers
docker-compose down

# Khởi động lại containers
docker-compose restart

# Xem trạng thái containers
docker-compose ps

# Chạy lệnh trong container
docker-compose exec app bash
```

## Bước 8: Sử dụng Nginx làm Reverse Proxy cho Docker Container
### Tạo cấu hình Nginx
```bash
sudo nano /etc/nginx/sites-available/myapp
```

Thêm nội dung:
```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Kích hoạt cấu hình
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

## Bước 9: Triển khai tự động với Docker
### Tạo script deploy.sh
```bash
#!/bin/bash

# Pull code mới từ git
git pull origin main

# Build Docker image mới
docker build -t myapp:latest .

# Dừng container cũ
docker stop myapp-container || true
docker rm myapp-container || true

# Chạy container mới
docker run -d -p 3000:3000 --name myapp-container myapp:latest

echo "Deploy hoàn tất!"
```

### Cấp quyền thực thi
```bash
chmod +x deploy.sh
```

### Chạy script
```bash
./deploy.sh
```

## Bước 10: Một số lệnh Docker hữu ích khác
### Xóa image
```bash
docker rmi myapp:latest
```

### Xóa tất cả container dừng
```bash
docker container prune
```

### Xóa tất cả image không được sử dụng
```bash
docker image prune
```

### Push image lên Docker Hub (nếu muốn chia sẻ)
1. Tạo tài khoản trên Docker Hub (https://hub.docker.com)
2. Đăng nhập:
```bash
docker login
```

3. Tag image:
```bash
docker tag myapp:latest username/myapp:latest
```

4. Push image:
```bash
docker push username/myapp:latest
```

## Bước 11: Cấu hình HTTPS/SSL cho Docker Container
### Sử dụng Certbot với Nginx reverse proxy
```bash
sudo certbot --nginx -d your_domain.com
```

Nginx sẽ tự động cấu hình HTTPS và redirect HTTP sang HTTPS.

### Sau đó cập nhật cấu hình Nginx
```nginx
server {
    listen 80;
    server_name your_domain.com;
    redirect to https
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your_domain.com;

    ssl_certificate /etc/letsencrypt/live/your_domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Khởi động lại Nginx:
```bash
sudo systemctl restart nginx
```
