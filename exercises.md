# Phiếu Phản Ánh - K4 Ngày 12

Họ và tên: Nguyễn Huy Toàn  
Mã học viên: 2A202601716

---

### Câu 1 - Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Railway, nếu tôi quên set `API_TOKEN`, app sẽ lỗi ngay lúc khởi động hoặc healthcheck không qua. Nhờ vậy tôi biết cần sửa biến môi trường trước khi public URL hoạt động. Nếu để mặc định `"changeme"`, service vẫn chạy và người khác có thể đoán token để gọi `/chat`, làm mất kiểm soát chi phí.

---

### Câu 2 - Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi nêu hai việc bạn làm được với dòng log đó mà `print("đã trả lời xong")` không làm được.

> Ví dụ log: `{"event":"chat_completed","severity":"INFO","ts":"2026-08-10T10:51:15+00:00","client_id":"sv-test","prompt_tokens":4,"completion_tokens":35,"usd_cost":0.0000216}`. Với log JSON này tôi có thể lọc theo `event` hoặc `severity`, và có thể thống kê chi phí theo `client_id`. Một câu `print()` thường không có cấu trúc nên khó lọc, khó đếm và khó cảnh báo tự động.

---

### Câu 3 - Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng lớn hơn bản multi-stage vì dùng `python:3.11` đầy đủ |
| Multi-stage | dưới 400 MB theo test CP2 |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần chênh lệch chủ yếu đến từ base image đầy đủ, cache pip, compiler hoặc công cụ build không cần thiết ở runtime. Multi-stage cho phép cài dependency ở stage `builder`, sau đó chỉ copy kết quả cần chạy sang image runtime `python:3.11-slim`, nên image cuối nhỏ và ít bề mặt tấn công hơn.

---

### Câu 4 - Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt `COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile hiện copy `requirements.txt` rồi mới `pip install`, sau đó mới copy `app` và `utils`. Khi chỉ sửa code trong `app/main.py`, layer cài dependency vẫn dùng cache, chỉ các layer copy source và sau đó chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, mỗi lần sửa code nhỏ Docker sẽ mất cache của layer copy và phải cài lại toàn bộ thư viện, build chậm hơn nhiều.

---

### Câu 5 - Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu app có lỗ hổng cho phép chạy lệnh trong container, kẻ tấn công sẽ có quyền của user đang chạy process. Nếu process chạy root, quyền đó quá rộng và khi kết hợp với lỗi cấu hình volume hoặc runtime có thể ảnh hưởng host. Lệnh `USER appuser` làm app chạy bằng user thường, nên kể cả khi app bị khai thác thì quyền trong container vẫn bị giới hạn.

---

### Câu 6 - Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả cùng một thông báo lỗi cho cả ba trường hợp thiếu header, sai scheme, sai token?

> `WWW-Authenticate: Bearer` cho client biết endpoint yêu cầu kiểu xác thực Bearer theo chuẩn HTTP. Tôi trả cùng một thông báo lỗi để không tiết lộ chi tiết cho người đang dò token. Nếu nói rõ "sai scheme" hay "sai token", attacker có thêm tín hiệu để thử từng bước dễ hơn.

---

### Câu 7 - Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn `min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Nó gửi được tối đa 10 request liên tiếp trước khi bị 429, vì bucket chỉ chứa tối đa `capacity=10`. Nếu bỏ `min(capacity, ...)`, sau 10 phút bucket có thể tích thêm khoảng 100 token, nên client có thể gửi khoảng 100 request liên tiếp. Điều đó sai vì thời gian im lặng dài sẽ biến thành burst quá lớn.

---

### Câu 8 - Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là bao nhiêu và service tự hồi phục khi nào?

> Với hạn mức $30/tháng, sự cố có thể đốt gần hết $30 trước khi bị chặn, và client chỉ tự dùng lại được khi sang chu kỳ tháng mới hoặc khi mình can thiệp. Với hạn mức $1/ngày, thiệt hại tối đa trong ngày là khoảng $1 cho client đó, và sang ngày UTC mới key ngân sách đổi nên service tự hồi phục.

---

### Câu 9 - /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm 3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm endpoint health trả lỗi. Orchestrator tưởng cả 3 container đều hỏng và restart chúng. Trong lúc restart, service mất instance phục vụ request dù process app thật ra vẫn sống. Khi Redis quay lại, cụm còn phải khởi động lại thay vì chỉ tạm ngừng nhận traffic. Vì vậy `/healthz` chỉ kiểm tra process, còn `/readyz` mới kiểm tra Redis.

---

### Câu 10 - Deploy thật (CP5)

Ghi lại một lỗi bạn gặp khi deploy lên cloud: thông báo lỗi là gì, bạn tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Tôi gặp lỗi Railway `Deployment failed during the network process` và healthcheck failure. Sau đó app online nhưng `/readyz` trả 500 rồi 503 `{"status":"not ready","redis":false}`. Tôi kiểm tra Railway Variables thì thấy `REDIS_URL` ban đầu rỗng hoặc chưa nối đúng Redis service. Tôi sửa bằng cách tạo Redis database trên Railway, set `REDIS_URL` trỏ tới Redis service, bỏ `startCommand` trong `railway.toml` để Docker CMD tự đọc `$PORT`, rồi redeploy. Sau đó `/healthz` và `/readyz` hoạt động đúng.
