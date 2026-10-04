# Tech lập trình

Kinh nghiệm script đã gãy trên Windows.

- Skill gọi file này: `tech-lap-trinh`
- Đọc khi: AutoIt, cmd, PowerShell, hotspot
- Cập nhật: 2026-10-04

## AutoIt gọi PowerShell

Việc: bọc PowerShell trong AutoIt rồi file kết quả rỗng.

Làm:
1. Dấu chuyển hướng đầu ra là của cmd. Phải gọi qua bộ xử lý lệnh, không gắn thẳng vào tệp chạy.
2. AutoIt 32-bit trên Windows 64-bit phải gọi PowerShell qua Sysnative, không qua System32.
3. In một dòng có tiền tố rồi mới đọc, đừng tin chuỗi rỗng từ host không console.

Không làm: kết PowerShell hỏng trước khi kiểm tra chuyển hướng và bản 32/64.

Đã thấy: 2026-09-19, bật phát sóng Windows.

## Cmd gãy vì ngoặc

Việc: trong khối `if (` có lệnh echo chứa dấu `)` của PowerShell.

Làm: để script PowerShell thành file tĩnh. Bắt buộc echo thì thoát dấu ngoặc. Bắt mã thoát ngay dưới lệnh, trước echo.

Không làm: tin mã thoát sau echo hoặc type.

## Phát sóng báo bật nhưng màn hình tắt

Việc: trạng thái vận hành trả về đang bật, trang cài đặt vẫn tắt.

Làm: dừng rồi bật lại nếu cần sóng thật. Đóng trang cài đặt rồi mở, không chỉ tải lại. Tắt IPv6 thì bỏ qua loopback và card ảo phát sóng.

Đã thấy: 2026-09-19. API bật không bằng nút trên giao diện.
