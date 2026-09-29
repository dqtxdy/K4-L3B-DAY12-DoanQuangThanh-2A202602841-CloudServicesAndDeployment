# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> Thiếu `AGENT_API_KEY` làm app lỗi ngay khi tạo Settings lúc khởi động. Mình phát hiện biến secret chưa được cấu hình trước khi nhận traffic, thay vì để service chạy với khóa mặc định dễ đoán.` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đoàn Quang Thanh  Mã học viên: 2A202602841

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T03:02:23.222117+00:00","user_id":"shared-test","tokens_in":5,"tokens_out":39,"cost_usd":2.415e-05}`. Mình lọc được theo user và tổng hợp token/chi phí; `print("đã trả lời xong")` không có các trường đó.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Mình build Dockerfile một stage nguyên bản từ template và image multi-stage hiện tại. Docker hiển thị 1.72GB cho bản một stage và 269MB cho bản multi-stage. Bản sau nhỏ hơn vì chỉ mang dependencies và source runtime, không giữ toàn bộ base image cùng môi trường cài đặt ở stage build.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.72 GB |
| Multi-stage | 269 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Sau khi chỉ sửa source, Docker dùng lại layer `COPY requirements.txt` và `RUN pip install`; layer copy source cùng các layer sau chạy lại. Nếu `COPY . .` đứng trước lệnh cài dependency, đổi một file source sẽ làm layer cài dependency bị chạy lại.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Nếu app bị khai thác khi chạy root, tiến trình bị chiếm có quyền root trong container và có thể ảnh hưởng nhiều file/process hơn. `USER appuser` giới hạn quyền của tiến trình; nó giảm phạm vi tác động nhưng không tự đảm bảo host an toàn.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Có thể gửi 10 request ngay trước mốc phút mới và 10 request ngay sau mốc đó: tổng 20 request trong khoảng hai giây.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Rate limit giới hạn số request trong 60 giây; cost guard giới hạn tổng tiền theo user trong tháng. Request rẻ có thể qua rate limit nhưng bị chặn khi tổng chi phí dự kiến vượt ngân sách. Ngược lại, request đắt có thể bị cost guard chặn dù user còn quota request.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Nếu liveness cũng kiểm tra Redis, lúc Redis rớt cả ba container đều bị đánh dấu unhealthy dù process vẫn chạy. Orchestrator có thể restart chúng; khi Redis hồi phục, app lại phải khởi động và có thể tạo vòng restart. `/health` chỉ kiểm tra process, còn `/ready` mới kiểm tra Redis.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Mình chạy `.venv/bin/docker-compose up -d --scale agent=3`. Compose cảnh báo host port sẽ clash; replica 1 và 2 lỗi `Bind for 0.0.0.0:8000 failed: port is already allocated` vì mỗi replica publish `8000:8000`. Trong phép thử hai agent riêng nối cùng Redis, instance 1 trả `history_length=0`, request sau ở instance 2 trả `2`; nếu mỗi process giữ dict riêng, instance 2 sẽ không thấy hai message của instance 1.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Deploy Render đầu tiên của commit `a790628` thoát với `NotImplementedError: TODO (CP4): cài đặt install`; log chỉ tới `Lifecycle.install()` trong `app/lifecycle.py`. Commit đó còn TODO ở handler SIGTERM/SIGINT nên startup thất bại. Mình hoàn thiện `Lifecycle.install()` rồi đẩy code lên `main`; deploy sau đó chạy Live.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> *Câu trả lời của bạn*
