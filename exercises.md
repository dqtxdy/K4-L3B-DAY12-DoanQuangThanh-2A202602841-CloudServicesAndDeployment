# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Ghi rõ phần nào đã quan sát và phần nào chưa chạy được.
>
> Cách trả lời: thay phần gợi ý trong mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Đoàn Quang Thanh  Mã học viên: 2A202602841

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Thiếu `AGENT_API_KEY` làm `Settings` báo lỗi ngay khi khởi động. Như vậy deploy dừng ở health check và mình biết secret chưa được cấu hình, thay vì để service chạy với một khóa chung mà người ngoài có thể đoán.

---
### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> `{"event":"ask_completed","level":"info","timestamp":"2026-09-29T03:02:23.222117+00:00","user_id":"shared-test","tokens_in":5,"tokens_out":39,"cost_usd":2.415e-05}`. Mình có thể lọc theo user để xem mức dùng, hoặc tổng hợp token/chi phí theo thời gian; một câu `print` cố định không có các trường dữ liệu đó.

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
| 1 stage (starter `python:3.11`) | 1.72 GB |
| Multi-stage (`day12-agent:prod`) | 269 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Mình build starter một stage và image hiện tại trên cùng Docker daemon. Bản starter giữ cả base image đầy đủ và môi trường build nên lớn hơn khoảng 1.45 GB; multi-stage chỉ mang runtime cùng dependencies đã cài, bỏ compiler và các file build.

---
### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Mình thêm tạm một comment vào `app/main.py` rồi build lại. Docker báo `WORKDIR`, `COPY requirements.txt`, `RUN pip install` và `COPY --from=builder` dùng cache; `COPY app` cùng các layer sau chạy lại. Nếu `COPY . .` đứng trước pip install thì đổi source sẽ làm layer pip install chạy lại.

---
### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu tiến trình bị khai thác, kẻ tấn công được quyền truy cập những gì user tiến trình có thể đọc hoặc sửa; chạy root mở rộng phạm vi thiệt hại. `USER appuser` chuyển app sang UID không đặc quyền và giảm quyền trong container, nhưng không tự nó bảo đảm an toàn cho host.

---
### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với hạn mức 10 mỗi phút theo đồng hồ, có thể gửi 10 request ngay trước mốc phút mới rồi 10 ngay sau mốc đó: tổng cộng 20 request trong khoảng hai giây.

---
### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit chặn tốc độ request trong cửa sổ 60 giây; cost guard chặn tổng tiền theo user trong tháng. Nhiều request rẻ có thể chạm rate limit mà chưa hết tiền; request có chi phí dự kiến lớn có thể bị cost guard từ chối dù còn quota tốc độ.

---
### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu cả ba container dùng health check phụ thuộc Redis, Redis mất kết nối khiến cả ba bị đánh dấu không khỏe và có thể bị restart. `/health` chỉ kiểm tra process; `/ready` kiểm tra Redis rồi ngừng gửi traffic tới instance chưa sẵn sàng.

---
### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Mình chạy hai agent containers trên cùng Docker network, nối tới Redis chung ở các cổng host 8000 và 8001. Request đầu ở instance 1 trả `history_length=0`; request kế tiếp qua instance 2 trả `history_length=2`. Điều này xác nhận hai process đọc cùng history từ Redis; dict riêng trong mỗi process sẽ không chia sẻ hai message đó.

---
### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy Render đầu báo thất bại. Mình đối chiếu mã trên nhánh `main` lúc đó với bản đã sửa trong workspace và thấy GitHub vẫn có `/health` ném `NotImplementedError`; các sửa đổi cần thiết chưa được đẩy lên repo nên Render build mã cũ. Sau khi mã đã sửa được đưa lên `main`, deploy thành công. Mình không giữ nguyên log chi tiết của lần thất bại đầu, nên không ghi nguyên văn thông báo ngoài trạng thái failed trên dashboard.
