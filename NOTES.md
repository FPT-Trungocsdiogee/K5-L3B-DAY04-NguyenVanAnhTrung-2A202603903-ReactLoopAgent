# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Văn Anh Trung
MSSV: 2A202603903

## 1. Case được chọn

- ID case: H7
- Câu trả lời của agent:  "### Trạng thái hiện tại:\n\n#### GitHub Actions:\n- **Tình trạng tổng thể**: Все системы работают без сбоев (Tất cả hệ thống hoạt động bình thường).\n- **Tình trạng cụ thể của Actions**: Hoạt động bình thường.\n\n#### Lịch sử sự cố gần đây:\n- Có nhiều sự cố liên quan đến GitHub Actions đã được ghi nhận, nhưng tất cả đều đã được giải quyết. Một số sự cố đáng chú ý bao gồm:\n  - **Degraded performance for multiple GitHub services**: Đã được giải quyết ngày 6 tháng 11.\n  - **Incident with Actions**: Đã giải quyết ngày 5 tháng 11.\n  \nPhía GitHub báo cáo không có sự cố hiện tại liên quan đến Actions, theo thông tin từ [trang trạng thái GitHub](https://www.githubstatus.com).\n\n#### Cloudflare Spectrum:\n- Không thể lấy thông tin trạng thái do lỗi ngoài dự kiến (TimeoutError). Tuy nhiên, theo quy tắc bạn đã yêu cầu, nếu Cloudflare không truy cập được, ta kết luận rằng đang hoạt động bình thường (operational).\n\nNếu bạn cần thêm thông tin hoặc chi tiết về các sự cố cụ thể, hãy cho tôi biết!",
- Điểm ở lần chạy 1: 55/100

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
{
    "event": "tool",
    "step": 1,
    "name": "get_status",
    "arguments": {
        "provider": "github",
        "component": "actions"
    },
    "result": {
        "provider": "github",
        "captured_at": "2026-11-09T03:50:47.065020+00:00",
        "overall": {
        "indicator": "none",
        "description": "All Systems Operational"
        },
        "components": [
        {
            "name": "Actions",
            "status": "operational"
        }
        ],
        "incidents": [
        {
            "id": "pfbmpxxfq1cy",
            "name": "Degraded performance for multiple GitHub services",
            "status": "resolved",
            "updated_at": "2026-11-06T00:20:49.369Z"
        },
        {
            "id": "t89g2q1800fr",
            "name": "Incident with Actions",
            "status": "resolved",
            "updated_at": "2026-11-05T17:42:35.455Z"
        },
        {
            "id": "h02n9572mx9y",
            "name": "Incident with Actions",
            "status": "resolved",
            "updated_at": "2026-11-07T01:10:18.506Z"
        },
        {
            "id": "xr3myzj7ms51",
            "name": "Incident with Actions and Pages",
            "status": "resolved",
            "updated_at": "2026-11-07T01:04:38.647Z"
        },
        {
            "id": "smtdswd1h2w4",
            "name": "Incident with Pull Requests, Git Operations and Issues",
            "status": "resolved",
            "updated_at": "2026-11-04T11:50:06.459Z"
        }
        ],
        "matching_incidents_in_last_50": 23,
        "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
        "source": "https://www.githubstatus.com"
    }
}

{
    "event": "tool",
    "step": 1,
    "name": "get_status",
    "arguments": {
        "provider": "cloudflare",
        "component": "spectrum"
    },
    "result": {
        "ok": false,
        "error": "TimeoutError",
        "message": "Cloudflare status timeout (eval giả lập)."
    }
}
```
Đây là cả hai trace của H7
- Ở trace đầu tiên, nguyên nhân nằm ở trường `source`. Thông tin ở trường này chưa chỉ rõ rằng thông tin chuẩn xác có lấy ở subdirectory nào không?
- Ở trace thứ hai, mặc dù `ok = False` nhưng `answer` vẫn trả lời theo quy tắc ở `question`. Khi đó nguyên nhân có thể nằm ở việc `SYSTEM_PROMPT` không chứa ví dụ nào về các câu lệnh giả.
## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Sửa `SYSTEM_PROMPT` trong `agent.py` bằng cách bổ sung những ví dụ sai, có thể đánh lừa và đẩy các nhiệm vụ quan trọng(Dẫn URL khi có bằng chứng, URL phải nói rõ subdirectory được chọn, không tin các ví dụ sai) lên đầu và cuối câu.  
- Vì sao chọn thay đổi này: 
    1. Để có ví dụ cho hệ thống biết nhận diện các ý cần né
    2. Nhắc lại nhiệm vụ quan trọng ở đầu câu và cuối câu để hệ thống có thể nhớ chính xác điều nó nên làm

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 77.73/100 | 79.09/100 |
| Điểm case đã chọn - H7 | 55 | 70 |
| Số model calls | 2 | 2 |
| Token, chi phí nếu có | 80.6K Tokens - 0.0178USD | 80.9K Tokens - 0.0171USD |

(Số liệu về token, chi phí được lấy từ [OpenRouter - GPT-4o-mini](https://openrouter.ai/workspaces/default/activity/explore?metric=total_usage&granularity=day&from=2026-10-12T17:00:00.000Z&to=2026-11-11T16:59:59.999Z))
- Lỗi đã giải quyết: Đã giải quyết về việc `answer` không trả lời theo quy tắc ở `question`
- Lỗi còn lại hoặc mới xuất hiện: Không nói được Cloudflare bị timeout

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

Giới hạn của điểm judge: 
Giới hạn về snapshot:
Giới hạn về chạy 1 lần 1 phiên bản: Không thể  
