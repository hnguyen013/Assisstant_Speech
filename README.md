# Assisstant_Speech

Ứng dụng trợ lý ảo luyện nói tiếng Anh tương tác bằng giọng nói xây dựng trên nền tảng **MIT App Inventor**, kết hợp với mô hình trí tuệ nhân tạo **Google Gemini AI** (`gemini-2.5-flash`).

---

## 📌 1. Giới thiệu & Tính năng chính

- **Nhận diện giọng nói (SpeechRecognizer):** Thu âm và chuyển giọng nói của học sinh thành văn bản tiếng Anh (`SpeechRecognizer1`).
- **Phát âm chuẩn (TextToSpeech):** Phát âm các câu hỏi, hướng dẫn và phản hồi từ AI (`TextToSpeech1`).
- **Tích hợp Gemini AI:** Tự động sửa lỗi phát âm/ngữ pháp và hỗ trợ học sinh lớp 4 thực hành hội thoại (`Web1`).
- **Luyện tập theo từng Unit:** Tự động nhận diện Unit học sinh chọn (Unit 1, Unit 11 đến Unit 20) và nạp ngân hàng câu hỏi tương ứng.
- **Hội thoại có ngữ cảnh (Multi-turn Context):** Lưu trữ lịch sử câu hỏi và câu trả lời của các lượt trước vào `global history` để AI hiểu trọn vẹn mạch đối thoại.
- **Reset phiên học:** Cho phép bắt đầu lại buổi học bất kỳ lúc nào (`Button3`).

---

## 🛠️ 2. Các lỗi đã được khắc phục (Fixed Bugs)

| Mã lỗi | Mức độ | Vị trí | Mô tả vấn đề & Cách khắc phục | Trạng thái |
|---|---|---|---|---|
| **BUG-1** | 🔴 Cao | `SpeechRecognizer1.AfterGettingText` | Nút Mic (`Button1`) hiển thị lại quá sớm khi request đang gửi, dẫn đến việc bấm liên tục gây trùng request. **Đã fix:** Ẩn nút Mic (`Button1.Visible ← FALSE`) trong lúc gọi API, chỉ bật lại khi nhận response hoặc Reset. | ✅ Đã sửa |
| **BUG-2** | 🔴 Cao | `SpeechRecognizer1.AfterGettingText` | Payload gọi Gemini API thiếu trường `"role": "user"` trong `contents`, gây lỗi **HTTP 400 Bad Request**. **Đã fix:** Thêm `"role": "user"` vào payload JSON. | ✅ Đã sửa |
| **BUG-3** | 🔴 Cao | `Web1.Url` (`Screen1.bky` & `Screen1.scm`) | Endpoint API sử dụng `/v1/`, nhưng model `gemini-2.5-flash` yêu cầu endpoint `/v1beta/`, dẫn đến lỗi **HTTP 404**. **Đã fix:** Chuyển endpoint sang `v1beta`. | ✅ Đã sửa |
| **BUG-7** | 🔴 Cao | `Web1.GotText` | Không có khối kiểm tra mã HTTP; khi API lỗi (400, 404, 429...), app truy xuất `candidatesList[1]` gây crash runtime. **Đã fix:** Thêm rẽ nhánh `if responseCode = 200`, có thông báo giọng nói & hiển thị lỗi khi gặp sự cố. | ✅ Đã sửa |
| **BUG-14** | 🔴 Cao | `Button3.Click` (Reset) | Nút Reset không đặt lại biến cờ `global first` về `FALSE`. Sau khi Reset, `first` vẫn là `TRUE` khiến app bỏ qua việc nhận diện Unit mới và luôn báo *"Sorry. Can you say again?"*. **Đã fix:** Thêm `global first ← FALSE` vào sự kiện Reset. | ✅ Đã sửa |
| **BUG-15** | 🟡 Trung bình | `Button3.Click` (Reset) | Nút Reset không đặt lại `global check` về `FALSE`. **Đã fix:** Thêm `global check ← FALSE` vào `Button3.Click`. | ✅ Đã sửa |
| **BUG-16** | 🟡 Trung bình | `Button3.Click` (Reset) | Nút Reset không làm rỗng `global currentVob`. **Đã fix:** Thêm `global currentVob ← create empty list` vào `Button3.Click`. | ✅ Đã sửa |
| **BUG-4 & BUG-13** | 🟡 Trung bình | `StartUnit` / Prompt | Biến `history` bị ghi đè chỉ chứa câu hỏi hiện tại, không lưu câu trả lời trước đó của học sinh khiến ngữ cảnh AI bị đứt gãy. **Đã fix:** Cập nhật `StartUnit` để tích lũy toàn bộ mạch đối thoại (`[Câu hỏi cũ] -> Student: [Câu trả lời] -> Question: [Câu hỏi mới]`), giữ trọn vẹn ngữ cảnh hội thoại đa lượt cho Gemini AI. | ✅ Đã sửa |

---

## 🔍 3. Rà soát các lỗi & vấn đề còn tồn đọng (Remaining Issues & Backlog)

Dưới đây là danh sách các lỗi logic, trải nghiệm người dùng (UX) và tối ưu mã nguồn còn lại được ghi nhận:

### 3.1. Nhóm lỗi Logic & Trải nghiệm (Cần lưu ý)

- **BUG-6 (🟡 Trung bình) — `Label2` ("You said: ...") không bao giờ hiển thị:**
  - *Hiện trạng:* Trong `Screen1.scm`, `Label2` có thuộc tính `Visible: False`. Trong `Screen1.bky`, app có gán nội dung câu nói của học sinh (`Label2.Text ← "You said: " + global userText`), nhưng lại **không bao giờ gọi lệnh hiển thị** `Label2.Visible ← True`.
  - *Ảnh hưởng:* Học sinh không xem được câu nói mà AI nhận diện/sửa lại trên màn hình.
  - *Đề xuất fix:* Bổ sung khối lệnh `set Label2.Visible to TRUE` bên trong `Web1.GotText` (hoặc đặt mặc định `Visible: True` trong Designer).

- **BUG-9 (🟡 Trung bình) — Nhận diện nhầm Unit do `VobUnit1` chứa từ `"1"`:**
  - *Hiện trạng:* Danh sách từ vựng `VobUnit1` có chứa chuỗi `"1"`.
  - *Ảnh hưởng:* Khi học sinh nói chọn *"Unit 11"*, chuỗi `"unit 11"` có chứa ký tự `"1"`, khiến vòng lặp kiểm tra `VobUnit1` khớp trước và tự động chọn nhầm Unit 1 thay vì Unit 11.
  - *Đề xuất fix:* Bỏ phần tử `"1"` hoặc chuẩn hóa từ khóa nhận diện Unit thành từ đầy đủ (ví dụ: `"unit 1"`, `"unit one"` thay vì số đơn lẻ).

- **BUG-12 (🟢 Thấp) — Trạng thái sau khi hoàn thành Unit ("Great job!"):**
  - *Hiện trạng:* Khi hoàn thành hết câu hỏi của một Unit, app phát câu chúc mừng *"Great job! You finished Unit speaking practice"*. Tuy nhiên app không tự động chuyển trạng thái để chọn Unit mới. Nếu học sinh tiếp tục bấm Mic nói, app sẽ tiếp tục lặp lại câu chúc mừng này cho đến khi bấm Reset thủ công.
  - *Đề xuất fix:* Tự động gọi hàm Reset hoặc đặt `first ← FALSE` khi kết thúc Unit để học sinh có thể chuyển sang Unit khác ngay.

- **BUG-10 (🟢 Thấp) — Khối lệnh `if (first = FALSE)` bị lồng trùng lặp (Dead Code):**
  - *Hiện trạng:* Trong `Web1.GotText`, bên trong nhánh `DO0` của khối `controls_if (first = FALSE)` đầu tiên, có một khối con `controls_if (first = FALSE)` giống hệt bị lồng vào trong (do lỗi thao tác khi kéo thả block).
  - *Ảnh hưởng:* Không gây lỗi app nhưng làm rác mã nguồn.

### 3.2. Nhóm bảo mật & Dọn dẹp tài nguyên (Code Hygiene & Security)

- **BUG-5 (🟡 Bảo mật) — Gemini API Key bị lộ dạng Plain Text:**
  - *Hiện trạng:* API Key (`AIzaSy...`) được đính kèm trực tiếp trong khối văn bản ở `Screen1.bky`.
  - *Khuyến cáo:* Bất kỳ ai mở file `.aia` đều đọc được API key. Nên giới hạn quyền hạn của Key trên Google Cloud Console (HTTP referrers / Android apps) hoặc chuyển sang backend trung gian nếu đưa vào sản xuất.

- **Tài nguyên thừa (Unused Components & Extensions):**
  - Component `SpeechRecognizer2`: Đã khai báo trong `Screen1.scm` nhưng không có bất kỳ khối lệnh nào sử dụng trong `Screen1.bky`.
  - Biến toàn cục `isWaiting`: Khai báo sẵn nhưng không dùng ở đâu.
  - Extension `tcbgeminibot.com` trong thư mục `assets/external_comps/`: Đã import nhưng màn hình không sử dụng (dùng component `Web` tiêu chuẩn). Có thể xóa để giảm dung lượng file `.aia`.

---

## 🚀 4. Hướng dẫn sử dụng & Đóng gói (.aia)

1. File `.aia` hoàn chỉnh đã được đóng gói sẵn tại thư mục gốc: `Assisstant_Speech.aia` (kích thước ~3.45 MB).
2. Truy cập [MIT App Inventor](https://ai2.appinventor.mit.edu/).
3. Chọn menu **Projects** → **Import project (.aia) from my computer...**
4. Chọn file `Assisstant_Speech.aia` để tiến hành lập trình hoặc xuất file cài đặt Android (`.apk`).
