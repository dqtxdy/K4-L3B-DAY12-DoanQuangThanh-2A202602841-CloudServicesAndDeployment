# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Đoàn Quang Thanh |
| Mã học viên | 2A202602841 |
| Repo | https://github.com/dqtxdy/K4-L3B-DAY12-DoanQuangThanh-2A202602841-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-t5yx.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | platform tự gán |
| `AGENT_API_KEY` | ✅ | đặt trong dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value service `day12-redis` (connectionString từ Blueprint) |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://day12-agent-t5yx.onrender.com/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://day12-agent-t5yx.onrender.com/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://day12-agent-t5yx.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://day12-agent-t5yx.onrender.com/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://day12-agent-t5yx.onrender.com/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
$ curl -i https://day12-agent-t5yx.onrender.com/health
HTTP/2 200
date: Tue, 29 Sep 2026 04:38:34 GMT
content-type: application/json
cf-cache-status: DYNAMIC
rndr-id: d8b33387-96c0-45f7
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-ray: a4284c92d8d9f93b-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ok","service":"day12-agent","version":"1.0.0"}

$ curl -i https://day12-agent-t5yx.onrender.com/ready
HTTP/2 200
date: Tue, 29 Sep 2026 04:38:36 GMT
content-type: application/json
rndr-id: cc93ba63-91f6-4065
server: cloudflare
vary: Accept-Encoding
cf-cache-status: DYNAMIC
cf-ray: a4284c9b9f0bff83-SIN
alt-svc: h3=":443"; ma=86400

{"status":"ready","redis":true}

$ curl -i -X POST https://day12-agent-t5yx.onrender.com/ask -H "Content-Type: application/json" -d '{"question":"Hello"}'
HTTP/2 401
date: Tue, 29 Sep 2026 04:38:45 GMT
content-type: application/json
rndr-id: 402f5b4c-cb1a-4d2b
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a4284cd26a0ec8a7-HKG
alt-svc: h3=":443"; ma=86400

{"detail":"invalid or missing API key"}

$ curl -i -X POST https://day12-agent-t5yx.onrender.com/ask -H "Content-Type: application/json" -H "X-API-Key: $AGENT_API_KEY" -H "X-User-Id: sv-test-evidence" -d '{"question":"Deploy là gì?"}'
HTTP/2 200
date: Tue, 29 Sep 2026 04:38:46 GMT
content-type: application/json
rndr-id: 36a77a84-b3b4-4313
server: cloudflare
vary: Accept-Encoding
x-render-origin-server: uvicorn
cf-cache-status: DYNAMIC
cf-ray: a4284cd88f145e02-HKG
alt-svc: h3=":443"; ma=86400

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud.","user_id":"sv-test-evidence","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}

$ python - <<'PY'
import os
import httpx
base = 'https://day12-agent-t5yx.onrender.com/ask'
headers = {'X-API-Key': os.environ['DEPLOY_API_KEY'], 'X-User-Id': 'q9-rate-evidence'}
codes = []
with httpx.Client(timeout=30) as client:
    for _ in range(15):
        codes.append(client.post(base, headers=headers, json={'question': 'rate limit check'}).status_code)
print(codes)
PY
[200, 200, 200, 200, 200, 200, 200, 200, 200, 200, 429, 429, 429, 429, 429]
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
Không dùng phương án dự phòng; đã deploy thành công trên Render.
```
