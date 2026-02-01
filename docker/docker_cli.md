Dưới đây là bản **đóng gói đầy đủ** các lệnh Docker CLI thường dùng – đủ để bạn làm chủ từ lúc pull, build, run cho đến quản lý container và volume.

---

# 🚀 **1. Image Commands**

| Hành động                   | Lệnh                                      |
| --------------------------- | ----------------------------------------- |
| Xem danh sách image         | `docker images`                           |
| Xóa image                   | `docker rmi <image_id_or_name>`           |
| Build image từ Dockerfile   | `docker build -t <name>:<tag> .`          |
| Tag lại image               | `docker tag <src_image> <new_name>:<tag>` |
| Pull image từ Docker Hub    | `docker pull <image>:<tag>`               |
| Export image ra file `.tar` | `docker save -o image.tar <image>`        |
| Import image từ file        | `docker load -i image.tar`                |

---

# 📦 **2. Container Commands**

| Hành động                          | Lệnh                                                                            |
| ---------------------------------- | ------------------------------------------------------------------------------- |
| Xem container đang chạy            | `docker ps`                                                                     |
| Xem toàn bộ container (cả stopped) | `docker ps -a`                                                                  |
| Chạy container từ image            | `docker run --name <container_name> -p <host_port>:<container_port> -d <image>` |
| Dừng container                     | `docker stop <container_name>`                                                  |
| Start container                    | `docker start <container_name>`                                                 |
| Restart container                  | `docker restart <container_name>`                                               |
| Xóa container                      | `docker rm <container_name>`                                                    |
| Xem log container                  | `docker logs <container_name>`                                                  |
| Truy cập vào container             | `docker exec -it <container_name> /bin/bash`                                    |

---

# 🧱 **3. Volume Commands**

| Hành động          | Lệnh                                |
| ------------------ | ----------------------------------- |
| Xem volume         | `docker volume ls`                  |
| Tạo volume         | `docker volume create <name>`       |
| Xóa volume         | `docker volume rm <name>`           |
| Gắn volume khi run | `-v <volume_name>:<container_path>` |

---

# ⚙️ **4. Network Commands**

| Hành động           | Lệnh                           |
| ------------------- | ------------------------------ |
| Xem các network     | `docker network ls`            |
| Tạo network         | `docker network create <name>` |
| Xóa network         | `docker network rm <name>`     |
| Gắn network khi run | `--network <name>`             |

---

# 📂 **5. Docker Compose (nếu dùng)**

| Hành động          | Lệnh                           |
| ------------------ | ------------------------------ |
| Khởi động          | `docker-compose up -d`         |
| Dừng               | `docker-compose down`          |
| Xem log            | `docker-compose logs -f`       |
| Rebuild và restart | `docker-compose up --build -d` |

---

# 🧹 **6. Cleanup Commands**

| Hành động                                              | Lệnh                     |
| ------------------------------------------------------ | ------------------------ |
| Xóa tất cả container đã dừng                           | `docker container prune` |
| Xóa image không dùng                                   | `docker image prune`     |
| Xóa toàn bộ unused (container, image, network, volume) | `docker system prune -a` |

---

Nếu bạn muốn Cancer đóng gói mấy câu lệnh này thành **PDF cheat sheet** hoặc file `.sh` (script dùng hàng ngày), cứ nói mình làm liền cho nha 😎
