# Tech tin học

Kinh nghiệm hộp mạng nhà và tài khoản số. Đã bỏ tên mạng, serial, mail.

- Skill gọi file này: `tech-tin-hoc`
- Đọc khi: router, VPN, modem, tài khoản số, đăng bài
- Cập nhật: 2026-10-04

## Bốn lớp khách ra ngoài

Việc: máy khách không ra internet hoặc vẫn địa chỉ Việt.

Làm:
1. Khách phải nhận dải riêng, không chung DHCP với mạng nhà.
2. Luật hướng từ dải khách sang cổng VPN, và luật chiều về qua cổng thường.
3. Đường hầm phải lên. Thiếu luật chấp nhận trên cầu nối thì gói bị từ chối.
4. Sau khởi động lại phải dựng lại mạng khách. Không dựng thì máy rơi về dải nhà.

Không làm: tách VPN chỉ bằng tên Wi-Fi trên DHCP chung. Không cài lại gói khách khi chỉ thiếu luật chấp nhận.

Đã thấy: 2026-09-19, hộp Asus Merlin, khách một dải, nhà một dải, cổng OpenVPN. Hai đường WireGuard cùng địa chỉ nội bộ thì để một đường không nhận lưu lượng.

## DHCP không lẫn VPN

Việc: máy lạ dính VPN hoặc máy VPN bị tranh số.

Làm:
1. Pool tự động dừng trước dải gán tay.
2. Máy cần VPN đặt tĩnh hoặc gán theo MAC từ mốc đó.
3. Xóa lease cũ rồi khởi động lại dịch vụ phát số.

Không làm: nới pool đến cuối dải cho tiện.

Đã thấy: 2026-10-03, pool nhà kết thúc .199, từ .200 là VPN.

## Modem một cổng gigabit

Việc: Wi-Fi sát máy trần thấp dù gói quang cao.

Làm: cắm lần lượt từng cổng LAN. Chỉ cổng hiện 1 Gbps mới đưa sang router. Ba cổng còn lại trên một số hộp đời cũ là 100 Mbps.

Đã thấy: hộp Viettel đời cũ, một cổng GE. Đời Wi-Fi 6 mới có thể đủ bốn cổng GE. Thiếu dữ liệu nếu chưa đo từng cổng.

## IPv6 bị đẩy lại

Việc: đã tắt IPv6 mà sau mất điện lại có địa chỉ toàn cầu.

Làm: tắt trên cục điều khiển mesh, không tắt trên cục agent đã tắt DHCP. Mất điện xong thấy prefix trở lại thì làm lại. Không cần trả máy về mặc định.

## Cài Windows từ ISO

Việc: USB flash ghi bộ cài bị nóng, chậm, hỏng chip.

Làm: Rufus ghi ISO vào SSD rời. Giữ file ISO gốc. Ổ đích không phải ổ đang chạy hệ thống.

Đã thấy: 2026-10-03.

## Tài khoản số

Việc: tạo hoặc nối tài khoản.

Làm:
1. Mỗi dịch vụ một cửa. Không dùng Google làm cửa chung.
2. Không xác nhận gương mặt để đăng nhập Google.
3. Tài khoản Apple dùng mail riêng, không lấy Gmail hay Outlook.
4. Facebook tạo bằng Gmail có thể bị máy mới bắt đăng nhập Google. Coi là rủi ro gom dữ liệu.
5. Connector báo còn hoạt động hoặc không hoạt động. Không dùng chữ còn sống.

Không làm: ghi tên người và mail thật lên trang này.

Đã thấy: 2026-09-19. Tên chủ file Drive và OneDrive lệch nhau là chuyện tài khoản nối, không suy ra một người.
