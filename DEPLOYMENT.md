# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyễn Vũ Quang Anh |
| Mã học viên | L3A202602805 |
| Repo | https://github.com/quanganhnguyenuet/K4-L3A-DAY12-NguyenVuQuangAnh-L3A202602805-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3a-day12-nguyenvuquanganh-l3a202602805-cloud-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Cấu Hình

Chỉ liệt kê tên biến và nguồn cấu hình; không lưu giá trị secret trong repository.

| Biến | Đã set | Nguồn |
|------|--------|-------|
| `PORT` | ✅ | Railway tự cấp cho web service |
| `AGENT_API_KEY` | ✅ | Secret cấu hình trên Railway Dashboard |
| `REDIS_URL` | ✅ | Reference tới `REDIS_URL` của Redis service cùng Railway project |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Railway Variables |
| `MONTHLY_BUDGET_USD` | ✅ | Railway Variables |
| `LOG_LEVEL` | ✅ | Railway Variables |

## Lệnh Kiểm Tra

```powershell
$URL = "https://k4-l3a-day12-nguyenvuquanganh-l3a202602805-cloud-production.up.railway.app"

# 1. Liveness
curl.exe -i "$URL/health"

# 2. Readiness và kết nối Redis
curl.exe -i "$URL/ready"

# 3. Thiếu API key
Invoke-WebRequest -Uri "$URL/ask" -Method POST `
  -ContentType "application/json" -Body '{"question":"Hello"}'

# 4. Có API key
$API_KEY = Read-Host "Nhap AGENT_API_KEY tren Railway"
$Headers = @{ "X-API-Key" = $API_KEY; "X-User-Id" = "sv-test" }
$Body = @{ question = "Deploy là gì?" } | ConvertTo-Json
Invoke-WebRequest -Uri "$URL/ask" -Method POST -Headers $Headers `
  -ContentType "application/json; charset=utf-8" -Body $Body
```

## Kết Quả Chạy Thật

Kết quả được kiểm tra trực tiếp trên deployment Railway ngày 2026-09-28:

```text
GET  /health          -> 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready           -> 200 {"status":"ready","redis":true}
POST /ask (không key) -> 401 Unauthorized
POST /ask (có key)    -> 200, trả về answer, user_id, history_length, cost_usd và tokens

Rate limit, 15 request liên tiếp với cùng X-User-Id:
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
```

## Ảnh Minh Chứng

- `screenshots/dashboard.png` — Railway deployment và service đang hoạt động.
- `screenshots/health.png` — kết quả kiểm tra endpoint `/health`.

Deployment sử dụng Railway cloud, không sử dụng phương án `LOCAL_FALLBACK`.
