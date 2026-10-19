# Assisstant_Speech

Ứng dụng trợ lý ảo luyện nói tiếng Anh tương tác bằng giọng nói xây dựng trên nền tảng **MIT App Inventor**, kết hợp với mô hình **Google Gemini AI** (`gemini-2.5-flash`).

---

## 📌 Tính năng chính

- **Nhận diện giọng nói (SpeechRecognizer):** Thu âm và chuyển giọng nói của người dùng thành văn bản tiếng Anh.
- **Phát âm phản hồi (TextToSpeech):** Phát âm chuẩn câu hỏi, hướng dẫn và phản hồi từ AI.
- **Tích hợp Gemini API:** Xử lý ngôn ngữ tự nhiên, đóng vai trò trợ giảng tương tác thông minh.
- **Luyện tập theo từng Unit:** Tự động nhận diện Unit người học chọn và nạp ngân hàng từ vựng tương ứng.
- **Reset phiên học:** Cho phép bắt đầu lại buổi học bất kỳ lúc nào.

---

## 🛠️ Nhật ký cập nhật & Sửa lỗi (Changelog)

### 1. Fix BUG-1: Chặn gửi trùng request khi API đang xử lý
- **Vấn đề:** Nút Mic (`Button1`) vẫn hiển thị/cho phép bấm trong lúc request gửi đi đang chờ phản hồi từ Gemini API, dẫn đến việc người dùng có thể kích hoạt nhiều request trùng lặp.
- **Giải pháp:** Ẩn nút Mic (`Button1.Visible ← FALSE`) ngay khi bắt đầu gửi request và chỉ bật lại (`Button1.Visible ← TRUE`) khi nhận xong kết quả (`Web1.GotText`) hoặc khi bấm Reset (`Button3.Click`).

### 2. Fix BUG-2: Chuẩn hóa payload gọi Gemini API
- **Vấn đề:** Payload JSON gửi lên Gemini API thiếu trường `"role": "user"` trong mảng `contents`, gây lỗi **HTTP 400 Bad Request**.
- **Giải pháp:** Cập nhật khối tạo payload trong `SpeechRecognizer1.AfterGettingText` để thêm `"role": "user"`.

### 3. Fix BUG-3: Cập nhật API endpoint sang `v1beta`
- **Vấn đề:** Endpoint ban đầu sử dụng `/v1/models/gemini-2.5-flash`, trong khi model `gemini-2.5-flash` yêu cầu endpoint `/v1beta/`.
- **Giải pháp:** Đổi URL sang `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=...` ở cả block logic (`Screen1.bky`) và thuộc tính component `Web1` (`Screen1.scm`).

### 4. Fix BUG-7: Bổ sung xử lý lỗi HTTP cho API (Error Handling)
- **Vấn đề:** Khi API trả về mã lỗi HTTP (400, 404, 429, 500...), hàm decode JSON không tìm thấy key `"candidates"` nên trả về chuỗi `"not found"`. Lệnh truy xuất phần tử danh sách `candidates[1]` sau đó gây crash app (runtime error).
- **Giải pháp:** Bổ sung khối kiểm tra điều kiện `responseCode = 200`:
  - **Nếu thành công (200):** Tiến hành bóc tách kết quả câu trả lời bình thường.
  - **Nếu lỗi (!= 200):** Thông báo lỗi qua giọng nói (*"Sorry, there was an error. Please try again."*) và hiển thị thông báo trên giao diện, đồng thời mở lại nút Mic để người dùng thử lại.

### 5. Fix BUG-14: Reset trạng thái chọn Unit sau khi bấm nút Reset
- **Vấn đề:** Nút Reset (`Button3.Click`) chỉ reset các biến `questionIndex`, `currentUnit`, `history` nhưng quên đặt lại biến cờ `global first` về `FALSE`. Sau khi bấm Reset, `first` vẫn là `TRUE`, khiến tất cả các nhánh nhận diện Unit tiếp theo bị bỏ qua và app liên tục báo *"Sorry. Can you say again?"*.
- **Giải pháp:** Bổ sung lệnh gán `global first ← FALSE` vào luồng sự kiện `Button3.Click`.

---

## 🚀 Hướng dẫn Import vào MIT App Inventor

1. Đóng gói mã nguồn thành file `.aia` (thực chất là file zip của thư mục project).
2. Truy cập [MIT App Inventor](https://ai2.appinventor.mit.edu/).
3. Chọn menu **Projects** → **Import project (.aia) from my computer...**
4. Chọn file `Assisstant_Speech.aia` và tiến hành mở dự án.
