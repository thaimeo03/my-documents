✅ 1. Kiểm tra Docker Desktop đã bật WSL2 chưa

Vào Docker Desktop → Settings → General:

✔ "Use the WSL 2 based engine" phải bật
✔ "Start Docker Desktop when you log in" nên bật

Nếu chưa bật thì bật lên rồi reboot.

✅ 2. Cài WSL2 (nếu chưa có)

Chạy trên PowerShell (admin):

wsl --install


Sau đó restart.

✅ 3. Clone repo build Oracle 19c image
git clone https://github.com/oracle/docker-images.git
cd docker-images/OracleDatabase/SingleInstance/dockerfiles

✅ 4. Tải file cài Oracle 19c cho Linux

Vào:

👉 https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html

Tải file:

LINUX.X64_193000_db_home.zip


Kéo file ZIP vào folder:

docker-images/OracleDatabase/SingleInstance/dockerfiles/19.3.0/

✅ 5. Build image Oracle 19c

Trong PowerShell:

./buildContainerImage.ps1 -Version 19.3.0 -Edition EE


Nếu dùng Git Bash/WSL:

./buildContainerImage.sh -v 19.3.0 -e


Sau khi build xong, bạn sẽ thấy image:

oracle/database:19.3.0-ee

✅ 6. Chạy Oracle 19c container trên Windows
docker run -d `
  --name oracle19c `
  -p 1521:1521 `
  -p 5500:5500 `
  -e ORACLE_PWD=Admin123 `
  -v oracle19c_data:/opt/oracle/oradata `
  oracle/database:19.3.0-ee


Kiểm tra:

docker logs -f oracle19c


Đợi đến dòng:

DATABASE IS READY TO USE!

✅ 7. Kết nối từ DBeaver, Spring Boot, hoặc SQLPlus
JDBC:
jdbc:oracle:thin:@localhost:1521/ORCLPDB1


User:

Username	Password
SYS	Admin123 (connect as sysdba)
SYSTEM	Admin123
✅ 8. Tạo user cho ứng dụng

Vào SQLPlus bên trong container:

docker exec -it oracle19c bash
sqlplus / as sysdba


Tạo user:

CREATE USER thai IDENTIFIED BY Thai123;
GRANT CONNECT, RESOURCE, CREATE SESSION TO thai;
ALTER USER thai QUOTA UNLIMITED ON USERS;

🔥 Bonus: Nếu bạn muốn nhanh hơn

Bạn chỉ cần nói:

➡️ “Cancer, giúp tôi build image Oracle 19c trên Windows từng bước”
hoặc
➡️ “Cancer, setup Spring Boot kết nối Oracle 19c”

Mình sẽ viết full lệnh + cấu hình cho bạn chạy 100% không lỗi.
