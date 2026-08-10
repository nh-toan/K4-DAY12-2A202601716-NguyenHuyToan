# Thông Tin Deploy - Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Huy Toàn |
| Mã học viên | 2A202601716 |
| Repo | https://github.com/nh-toan/K4-DAY12-2A202601716-NguyenHuyToan |

Ghi chú cho bộ chấm: mã học viên là 2A202601716.

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-08-10 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn giá trị, không ghi giá trị token.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | có | Railway tự gán |
| `API_TOKEN` | có | đặt trong Railway Variables, không nằm trong repo |
| `REDIS_URL` | có | Redis database service của Railway |
| `BUCKET_CAPACITY` | có | 10 |
| `REFILL_PER_MINUTE` | có | 10 |
| `DAILY_BUDGET_USD` | có | 1.0 |
| `LOG_LEVEL` | có | INFO |

## Lệnh Kiểm Tra

```bash
curl -i https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/healthz
curl -i https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/readyz
curl -i -X POST https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'
```

## Kết Quả Chạy Thật

`/healthz`:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ok","service":"day12-chat-service","version":"1.0.0"}
```

`/readyz` sau khi tạo Redis service và set `REDIS_URL` trên Railway:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ready","redis":true}
```

`/chat` không có token:

```text
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer
```

## Ảnh Chụp Màn Hình

Ảnh dashboard Railway và kết quả gọi endpoint đã đặt trong thư mục `screenshots/`.

## Lỗi Đã Gặp Khi Deploy

Lần deploy đầu tiên bị fail ở bước Network Healthcheck. Nguyên nhân là Railway chưa có biến môi trường đầy đủ và `REDIS_URL` ban đầu bị rỗng, làm `/readyz` không kết nối được Redis. Cách sửa là tạo Redis service trên Railway, set `REDIS_URL` trỏ tới Redis service, bỏ `startCommand` trong `railway.toml` để Docker CMD đọc đúng `$PORT`, sau đó redeploy app.
