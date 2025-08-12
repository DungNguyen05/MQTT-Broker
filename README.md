# MQTT Mosquitto Docker

Docker-based MQTT broker với authentication và ACL.

## Setup nhanh

```bash
# 1. Sao chép và chỉnh sửa config
cp .env.example .env

# 2. Chọn chế độ trong docker-compose.yml
# DEV (anonymous): bỏ comment dòng mosquitto.dev.conf
# PRODUCTION (auth): giữ dòng mosquitto.secure.conf (mặc định)

# 3. Tạo user (nếu dùng secure mode)
docker run --rm -it \
  -v "$(pwd)/mosquitto:/mosquitto" \
  eclipse-mosquitto:2 \
  mosquitto_passwd -c /mosquitto/passwd myuser

# 4. Khởi động
docker-compose up -d
```

## Quản lý user

```bash
# Tạo user mới (file mới) (thay username bằng tên cần thêm)
docker run --rm -it -v "$(pwd)/mosquitto:/mosquitto" \
  eclipse-mosquitto:2 mosquitto_passwd -c /mosquitto/passwd "username"

# Thêm user (file có sẵn) (thay username bằng tên cần thêm)
docker run --rm -it -v "$(pwd)/mosquitto:/mosquitto" \
  eclipse-mosquitto:2 mosquitto_passwd /mosquitto/passwd "username"

# Xóa user (thay username bằng tên cần xoá)
docker run --rm -it -v "$(pwd)/mosquitto:/mosquitto" \
  eclipse-mosquitto:2 mosquitto_passwd -D /mosquitto/passwd "username"
```

## Cấu hình ACL

Chỉnh sửa file `mosquitto/acl`:

```bash
# Admin có quyền full
user admin
topic readwrite #

# User thường chỉ đọc/ghi topic riêng
user alice
topic readwrite home/alice/+
topic read sensors/+

# Device chỉ gửi data
user device1
topic write sensors/device1/+
topic read commands/device1/+
```

Bật ACL trong `mosquitto/conf/mosquitto.secure.conf`:
```conf
acl_file /mosquitto/acl
```

## Test kết nối

```bash
# Dev mode (anonymous)
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "hello"
mosquitto_sub -h localhost -p 1883 -t "test/topic"

# Secure mode (có auth)
mosquitto_pub -h localhost -p 1883 -t "test/topic" -m "hello" -u username -P password
mosquitto_sub -h localhost -p 1883 -t "test/topic" -u username -P password

# WebSocket
mosquitto_pub -h localhost -p 9001 -t "test/topic" -m "hello" -u username -P password
```

## SSL/TLS (optional)

Bỏ comment service `caddy` trong `docker-compose.yml` để có WSS + Let's Encrypt.

## Commands hữu ích

```bash
# Xem logs
docker-compose logs -f mosquitto

# Restart
docker-compose restart

# Backup config
tar -czf backup.tar.gz mosquitto/

# Reset data (cẩn thận!)
docker-compose down -v
rm -rf mosquitto/data/* mosquitto/log/*
```

## Ports

- `1883`: MQTT TCP
- `9001`: MQTT WebSocket  
- `443`: WSS (nếu dùng Caddy)