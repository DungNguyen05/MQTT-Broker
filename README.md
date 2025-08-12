# Mosquitto via Docker (secure & dev)

## Chuẩn bị
- Docker + Docker Compose
- Sửa `.env` (nếu dùng reverse proxy/Caddy): DOMAIN, LE_EMAIL

## Chọn chế độ cấu hình
- **Secure (khuyến nghị)**: mở `docker-compose.yml`, giữ dòng
  `./mosquitto/conf/mosquitto.secure.conf:/mosquitto/config/mosquitto.conf:ro`
- **Dev (anonymous)**: comment dòng trên, mở dòng
  `./mosquitto/conf/mosquitto.dev.conf:/mosquitto/config/mosquitto.conf:ro`

## Tạo user/password (nếu chạy secure)
```bash
docker run --rm -it \
  -v "$(pwd)/mosquitto:/mosquitto" \
  eclipse-mosquitto:2 \
  mosquitto_passwd -c /mosquitto/config/passwd myuser
# MQTT-Broker
