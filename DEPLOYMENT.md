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
| Public URL | Chưa có service healthy để kiểm tra |
| Platform | Render Blueprint (`render.yaml`) |
| Ngày deploy | Đã thử tạo Blueprint; deploy đầu thất bại do GitHub `main` vẫn chứa mã TODO cũ |
| Redis | Blueprint khai báo `day12-redis`; trạng thái cần xác nhận sau khi đồng bộ mã mới |

## Biến Môi Trường Cần Cấu Hình Trên Cloud

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

Chưa có URL HTTPS healthy nên chưa thể gọi `/health`, `/ready` hoặc `/ask` từ Internet. Chưa có output verification hoặc screenshot dashboard/health thật. Các lệnh kiểm tra cần chạy sau khi deploy thành công:

```bash
curl -i https://<service-host>/health
curl -i https://<service-host>/ready
curl -i -X POST https://<service-host>/ask -H 'Content-Type: application/json' -d '{"question":"Hello"}'
```

## Evidence

`dashboard.png` và `health.png` chưa được tạo: không có service cloud để chụp ảnh xác thực.
