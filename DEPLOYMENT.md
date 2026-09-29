# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Đoàn Quang Thanh |
| Mã học viên | 2A202602841 |
| Repo | https://github.com/dqtxdy/K4-L3B-DAY12-DoanQuangThanh-2A202602841-CloudServicesAndDeployment |

## Trạng Thái Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-t5yx.onrender.com |
| Platform | Render Blueprint (`render.yaml`) |
| Ngày deploy | 2026-09-29 |
| Redis | `day12-redis` — readiness endpoint xác nhận kết nối thành công |

## Biến Môi Trường Đã Cấu Hình Trên Cloud

Blueprint yêu cầu `AGENT_API_KEY` khi tạo service; không ghi giá trị secret trong repository. Các biến còn lại được khai báo trong `render.yaml`:

| Tên biến | Nguồn |
|----------|-------|
| `PORT` | Cổng do platform cấp |
| `AGENT_API_KEY` | Secret tạo trong dashboard cloud |
| `REDIS_URL` | Redis cloud connection string |
| `RATE_LIMIT_PER_MINUTE` | `10` |
| `MONTHLY_BUDGET_USD` | `10.0` |
| `LOG_LEVEL` | `INFO` |

## Verification

Đã kiểm tra URL công khai từ môi trường bên ngoài vào ngày 2026-09-29:

```bash
GET  /health → 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET  /ready  → 200 {"status":"ready","redis":true}
POST /ask without API key → 401 {"detail":"invalid or missing API key"}
POST /ask with API key → 200 (answer returned; key omitted from this document)
POST /ask, 10 requests in one minute → first 10 return 200; request 11 returns 429
```

## Evidence

`health.png` chưa được chụp. `dashboard.png` chưa được chụp; cần ảnh dashboard Render thật để xác nhận service trên tài khoản.
