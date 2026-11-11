# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Hoàng Tùng
MSSV: 2A202603481

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: Agent nói rằng không có incident Github Actions
  trong năm 2025, sau đó liệt kê năm incident thuộc năm 2026.
- Điểm ở lần chạy 1: 35/100

## 2. Bằng chứng trong trace

Ở model step 1, agent gọi `get_status` với GitHub Actions. Tool trả về các
incident có `updated_at` trong tháng 11/2026, ví dụ:

```text
pfbmpxxfq1cy: updated_at = 2026-11-06T00:20:49.369Z
t89g2q1800fr: updated_at = 2026-11-05T17:42:35.455Z
h02n9572mx9y: updated_at = 2026-11-07T01:10:18.506Z
```

Tool cũng trả giới hạn dữ liệu:

```text
matching_incidents_in_last_50 = 23
Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh
không có sự cố.
```

Những dữ liệu này không bao phủ năm 2025. Tuy nhiên, câu trả lời cuối của agent
lại kết luận:

```text
Tính đến thời điểm kiểm tra, không có sự kiện nào liên quan đến GitHub Actions
diễn ra trong năm 2025.
```

Ở model step 2, agent còn gọi `get_incident_details` cho năm incident thuộc năm
2026 và đưa chúng vào câu trả lời, dù người dùng yêu cầu danh sách năm 2025.

Agent loop và tools vẫn hoạt động đúng: tool trả dữ liệu thành công, trace có đủ
tool calls và run kết thúc với `completed`. Lỗi nằm ở cách model suy luận từ dữ
liệu giới hạn. `SYSTEM_PROMPT` có nhắc đến giới hạn bằng chứng nhưng chưa có quy
tắc rõ ràng cho yêu cầu "danh sách đầy đủ" trong một khoảng thời gian không được
snapshot bao phủ. Vì vậy model đã biến "không có dữ liệu năm 2025" thành kết luận
không hợp lệ rằng "năm 2025 không có incident".

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Bổ sung quy tắc vào `SYSTEM_PROMPT` trong `agent.py`. Quy tắc
  yêu cầu model kiểm tra độ bao phủ thời gian, nói rõ khi dữ liệu chỉ là một phần,
  không xem kết quả tìm kiếm rỗng là bằng chứng không tồn tại, và không liệt kê
  incident ngoài khoảng thời gian nếu chúng không giúp trả lời câu hỏi.
- Vì sao chọn thay đổi này: Đây là thay đổi tổng quát cho mọi yêu cầu lịch sử hoặc
  yêu cầu danh sách đầy đủ, không hard-code case A5, năm 2025 hay một incident ID.
  Nó xử lý đúng nguyên nhân đã thấy trong trace mà không thay đổi contract của tool
  hoặc agent loop.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng |  |  |
| Điểm case đã chọn |  |  |
| Số model calls |  |  |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết:
- Lỗi còn lại hoặc mới xuất hiện:

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.
