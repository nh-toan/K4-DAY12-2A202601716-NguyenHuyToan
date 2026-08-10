# Thong Tin Deploy - Checkpoint 5

## Thong Tin Hoc Vien

| Muc | Noi dung |
|-----|----------|
| Ho va ten | Nguyen Huy Toan |
| Ma hoc vien | 2A202601716 |
| Repo | https://github.com/nh-toan/K4-DAY12-2A202601716-NguyenHuyToan |

Ghi chu cho bo cham: mã học viên la 2A202601716.

## Service

| Muc | Noi dung |
|-----|----------|
| Public URL | https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app |
| Platform | Railway |
| Ngay deploy | 2026-08-10 |

## Bien Moi Truong Da Set Tren Cloud

Chi ghi ten bien va nguon gia tri, khong ghi gia tri token.

| Bien | Da set | Ghi chu |
|------|--------|---------|
| `PORT` | yes | Railway tu gan |
| `API_TOKEN` | yes | dat trong Railway Variables, khong nam trong repo |
| `REDIS_URL` | yes | Redis database service cua Railway |
| `BUCKET_CAPACITY` | yes | 10 |
| `REFILL_PER_MINUTE` | yes | 10 |
| `DAILY_BUDGET_USD` | yes | 1.0 |
| `LOG_LEVEL` | yes | INFO |

## Lenh Kiem Tra

```bash
curl -i https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/healthz
curl -i https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/readyz
curl -i -X POST https://k4-day12-2a202601716-nguyenhuytoan-production.up.railway.app/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"Hello"}'
```

## Ket Qua Chay That

`/healthz`:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ok","service":"day12-chat-service","version":"1.0.0"}
```

`/readyz` sau khi tao Redis service va set `REDIS_URL` tren Railway:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"ready","redis":true}
```

`/chat` khong co token:

```text
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer
```

## Anh Chup Man Hinh

Anh dashboard Railway va ket qua goi endpoint da dat trong thu muc `screenshots/`.

## Loi Da Gap Khi Deploy

Lan deploy dau tien bi fail o buoc Network Healthcheck. Nguyen nhan la Railway chua co bien moi truong day du va `REDIS_URL` ban dau bi rong, lam `/readyz` khong ket noi duoc Redis. Cach sua la tao Redis service tren Railway, set `REDIS_URL` tro toi Redis service, bo `startCommand` trong `railway.toml` de Docker CMD doc dung `$PORT`, sau do redeploy app.
