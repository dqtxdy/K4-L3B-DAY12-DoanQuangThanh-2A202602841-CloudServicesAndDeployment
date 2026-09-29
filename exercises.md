# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đoàn Quang Thanh  Mã học viên: 2A202602841

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Thiếu `AGENT_API_KEY` thì `Settings` lỗi ngay lúc khởi động. Mình phát hiện secret chưa được cấu hình trước khi service nhận traffic, thay vì để app chạy với khóa mặc định dễ đoán.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T03:02:23.222117+00:00","user_id":"shared-test","tokens_in":5,"tokens_out":39,"cost_usd":2.415e-05}`. Mình lọc được theo user và tổng hợp token/chi phí theo thời gian; `print("đã trả lời xong")` không có các trường dữ liệu đó.

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
| 1 stage (bản đầu) | 1,720 MB |
| Multi-stage | 269 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Docker hiển thị image một stage là 1.72 GB và multi-stage là 269 MB. Bản multi-stage nhỏ hơn vì chỉ giữ runtime, dependencies và source cần chạy; không giữ toàn bộ base image cùng môi trường build.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ đổi `app/main.py`, layer cài dependency từ `requirements.txt` được dùng lại; layer copy source và các layer phía sau chạy lại. Nếu `COPY . .` đứng trước `RUN pip install`, đổi source cũng làm layer cài dependency chạy lại.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app bị khai thác khi chạy root, tiến trình bị chiếm có quyền root trong container và có thể đọc/sửa nhiều thứ hơn. `USER appuser` giới hạn quyền tiến trình; nó giảm phạm vi tác động nhưng không tự bảo đảm host an toàn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi 10 request ngay trước mốc phút mới rồi 10 request ngay sau mốc đó: tổng 20 request trong khoảng hai giây.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong 60 giây; cost guard giới hạn tổng chi phí theo user trong tháng. Request rẻ vẫn có thể làm tổng tháng vượt ngân sách dù còn quota tốc độ. Ngược lại, request có chi phí dự kiến lớn có thể bị cost guard chặn dù user còn quota request.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu health check gộp kiểm tra Redis, Redis mất kết nối sẽ làm cả ba container bị đánh dấu unhealthy dù process còn chạy. Orchestrator có thể restart chúng; Redis hồi phục thì các app lại khởi động. `/health` nên kiểm tra process, `/ready` kiểm tra Redis và readiness.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Chạy đúng `docker-compose up -d --scale agent=3` thì Compose cảnh báo và replica bị lỗi `Bind for 0.0.0.0:8000 failed: port is already allocated` vì mỗi replica publish cùng host port. Trong phép thử hai agent riêng cùng nối Redis, instance 1 trả `history_length=0`, request kế tiếp ở instance 2 trả `2`; hai process cùng đọc history từ Redis.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Deploy Render đầu tiên của commit `a790628` thất bại với `NotImplementedError: TODO (CP4): cài đặt install` tại `Lifecycle.install()` trong `app/lifecycle.py`. Hàm cài signal handler còn TODO nên startup dừng. Sau khi implement `Lifecycle.install()` và push code lên `main`, deploy sau chạy Live.
