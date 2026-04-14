# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Xuân Mong  
**MSSV:** 2A202600246
**Vai trò trong nhóm:** Worker Owner (Policy Tool)  
**Ngày nộp:** 14/04/2026

---

## 1. Tôi phụ trách phần nào? (100–150 từ)

Tôi chịu trách nhiệm thiết kế và triển khai **Policy Tool Worker** (`workers/policy_tool.py`). Đây là module then chốt giúp hệ thống đưa ra quyết định dựa trên chính sách nghiệp vụ.

Các công việc tôi đã hoàn thành:
- Triển khai hàm `analyze_policy` phối hợp giữa Rule-based (nhanh, chính xác cho case cứng) và LLM Reasoning (sâu, linh hoạt cho case phức tạp).
- Tích hợp thành công **NVIDIA Foundation API (model gpt-oss-120b)** để xử lý suy luận chính sách.
- Xây dựng cơ chế bắt và hiển thị `reasoning_content` giúp minh bạch hóa quá trình xử lý của AI.
- Đảm bảo Worker tuân thủ đầy đủ Contract về Input/Output trong hệ thống Multi-Agent.

**Bằng chứng:** Code đã được verify qua 3 kịch bản test (Flash Sale, Digital License, Standard Refund) và đã push lên repository (Commit `3df49c9`).

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

**Quyết định:** Tôi sử dụng mô hình **Hybrid Analysis** (Luật cứng + Suy luận AI) và giữ nguyên khả năng hiển thị chuỗi suy luận (Reasoning process) của model gpt-oss-120b.

**Lý do:** 
Trong các hệ thống pháp chế, việc đưa ra kết quả Đúng/Sai là chưa đủ, mà cần phải giải trình được tại sao. Việc chọn model `gpt-oss-120b` của NVIDIA cho phép tôi lấy được `reasoning_content`. Qua chạy thử, tôi thấy AI có khả năng tự so sánh các con số (VD: khách yêu cầu hoàn tiền trong 5 ngày, AI tự đối chiếu với luật 7 ngày để kết luận là hợp lệ).

Việc kết hợp Rule-based giúp chặn ngay các case vi phạm hiển nhiên (Flash Sale), trong khi AI hỗ trợ việc viết lời giải thích (`explanation`) mềm mỏng và thuyết phục cho khách hàng.

**Trade-off:** Việc hiển thị Reasoning trực tiếp ra console làm log dài hơn, nhưng đổi lại tính minh bạch của hệ thống tăng lên đáng kể.

**Bằng chứng:** 
```text
[Reasoning]: AI tự phân tích: "Customer requests refund within 5 days, product defective... So it complies with 7-day policy."
Result Policy Applies: True
```

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

**Lỗi:** "Silent Fail" trong việc ghi nhật ký hệ thống và Lỗi xác thực 401.

**Symptom:** Worker trả về kết quả nhưng không ghi lại lịch sử vào `history` và `worker_io_logs`. Đồng thời, ban đầu hệ thống báo lỗi `Authentication failed` mặc dù đã có file mẫu `.env.example`.

**Root cause:** 
1. `policy_tool.py` thiếu lệnh `load_dotenv()` nên không đọc được API Key.
2. Lệnh `return state` trong hàm `run` bị đặt sai vị trí (đặt trước phần ghi log), dẫn đến code ghi log trở thành "unreachable code".

**Cách sửa:** 
Tôi đã bổ sung `load_dotenv()` ở đầu file và tái cấu trúc lại hàm `run`, đưa lệnh `return` xuống cuối cùng để đảm bảo mọi bước đi của Worker đều được ghi nhận vào State của hệ thống.

**Bằng chứng:** Sau khi sửa, lệnh `python3 policy_tool.py` đã in ra đầy đủ kết quả của cả 3 Case Test thay vì bị ngắt quãng giữa chừng.

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

**Tôi làm tốt nhất ở điểm nào?**
Tôi đã hoàn thành Worker có độ tin cậy cao, vượt qua mọi kịch bản test biên. Phần giải thích của AI do tôi thiết kế rất chi tiết và bám sát tài liệu `policy_refund_v4.txt`.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Tôi còn lúng túng trong việc quản lý biến môi trường dẫn đến lỗi xác thực ban đầu.

**Nhóm phụ thuộc vào tôi ở đâu?**
Nếu không có Policy Worker, hệ thống sẽ thiếu đi bộ lọc quan trọng về tính hợp lệ của yêu cầu, dẫn đến việc synthesis worker có thể trả lời sai cam kết của công ty.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Dựa trên kết quả test Case 3 (Hợp lệ), tôi thấy AI giải thích rất tốt nhưng chưa trích dẫn chính xác số dòng (Line number) trong tài liệu. Nếu có thêm 2 giờ, tôi sẽ cải tiến Prompt để AI trả về chính xác số thứ tự dòng trong văn bản gốc giúp tăng tính pháp lý cho câu trả lời.

---
*Người viết: Nguyễn Xuân Mong*
