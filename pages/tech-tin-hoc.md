# Tech tin học

Khung kinh nghiệm. Grok không mở file này mỗi lần gọi skill.

- Skill gọi file này: `tech-tin-hoc`
- Đọc khi: việc quá khó, skill Grok không đủ dữ liệu
- Cập nhật: 2026-10-09
- Gom từ `pages/huong-dan-merlin-yazfi-vpn-guest.md` ngày 2026-10-09. File hướng dẫn gốc đã xóa.

`#` và `##` là một nhóm. `###` nằm trong nhóm phía trên. Không cần mục lục.

## Hai Wi-Fi Merlin: mạng nhà và mạng Guest đi VPN

Việc: điện thoại đổi tên Wi-Fi là đổi đường ra internet. Mạng chính đi nhà mạng. Mạng Guest đi VPN. Bài này lấy OpenVPN, ví dụ máy chủ Mỹ, firmware Asuswrt-Merlin 3004.388 (ví dụ RT-AX68U).

Không phải hướng dẫn phá khóa, vượt tường lửa bất hợp pháp, hay lấy trộm tài khoản VPN. Cần firmware Merlin, quyền SSH admin trên router của mình, và tài khoản VPN hợp lệ.

Làm:
1. Router để chế độ Wireless Router, có dây WAN (hoặc USB 4G). Bật JFFS custom scripts. Guest 2.4 GHz số 1 đã có tên và mật khẩu riêng. Interface thường là `wl0.1`.
2. OpenVPN Client 1: nạp file `.ovpn`, điền user/pass dịch vụ (Surfshark dùng service credentials, không phải email đăng nhập app). Redirect Internet traffic = VPN Director. Không để No Internet traffic. Không để All nếu chỉ Guest đi VPN. Killswitch tắt lúc mới dựng, bật sau khi đã ổn. Client Connected thì `nvram get vpn_client1_state` ra `2`. Interface thường `tun11`.
3. Cài YazFi bằng `amtm`, hoặc:

```sh
/usr/sbin/curl -fsL --retry 3 "https://raw.githubusercontent.com/AMTM-OSR/YazFi/master/YazFi.sh" -o /jffs/scripts/YazFi && chmod 0755 /jffs/scripts/YazFi && /jffs/scripts/YazFi install
```

4. Config Guest 1 (`wl01`): `ENABLED=true`, `IPADDR=192.168.51.1` (cổng `.1`, không phải `.0`), `ALLOWINTERNET=true`, `REDIRECTALLTOVPN=false`. Director cầm OpenVPN. YazFi REDIRECT chỉ nhắm số client OpenVPN, dễ lệch. Lệnh dựng mạng là `/jffs/scripts/YazFi runnow`. Không có lệnh `YazFi apply`. `YazFi startup` chỉ gắn menu web, không tạo dải `192.168.51`.
5. Lỗ firewall bắt buộc, đúng thư mục YazFi, không đặt vào `/jffs/scripts/YazFi_user/`:

```sh
mkdir -p /jffs/addons/YazFi.d/userscripts.d
cat > /jffs/addons/YazFi.d/userscripts.d/guest-ovpn.sh << 'EOF'
#!/bin/sh
# Guest 2.4 GHz số 1 = wl0.1 ; dải 192.168.51.0/24 ; OpenVPN client 1 = tun11
iptables -I YazFiFORWARD -i wl0.1 -j ACCEPT
iptables -I YazFiFORWARD -o wl0.1 -j ACCEPT
iptables -I YazFiFORWARD -s 192.168.51.0/24 -o tun11 -j ACCEPT
iptables -I YazFiFORWARD -i tun11 -d 192.168.51.0/24 -j ACCEPT
iptables -I YazFiFORWARD -s 192.168.51.0/24 -j ACCEPT
iptables -I FORWARD -s 192.168.51.0/24 -j ACCEPT
iptables -I OVPNCF -s 192.168.51.0/24 -j ACCEPT
iptables -I OVPNCF -d 192.168.51.0/24 -j ACCEPT
EOF
chmod 0755 /jffs/addons/YazFi.d/userscripts.d/guest-ovpn.sh
/jffs/scripts/YazFi runnow
```

Log phải có `Executing user script: .../guest-ovpn.sh`. Guest 5 GHz số 1 đổi `wl0.1` thành `wl1.1`. OpenVPN client 2 thường là `tun12`. Hai dòng `-i wl0.1` / `-o wl0.1` quan trọng: `wl0.1` hay còn nằm bridge `br0`, rule chỉ theo dải có thể 0 packet.
6. Hai rule VPN Director, cả hai bật. Rule đi: Local IP `192.168.51.0/24`, Remote trống, Interface OpenVPN 1. Rule về: Local trống, Remote `192.168.51.0/24`, Interface WAN. SSH mong đợi `from all to 192.168.51.0/24 lookup main` và `from 192.168.51.0/24 lookup ovpnc1`. `prohibit` cùng dải là killswitch.
7. Sống sau reboot, trong `/jffs/scripts/services-start` (giữ dòng `YazFi startup` nếu đã có):

```sh
#!/bin/sh
/jffs/scripts/YazFi startup & # YazFi
(sleep 30; /jffs/scripts/YazFi runnow) &
```

`chmod +x /jffs/scripts/services-start`. Sau reboot đợi khoảng 2 phút, trên điện thoại Quên mạng Guest rồi bắt lại. Lease cũ `192.168.50.x` không tự nhảy sang `192.168.51.x`.
8. Kiểm: IP `192.168.51.x`, router `192.168.51.1`, trình duyệt `http://1.1.1.1` (không qua ô tìm), trang xem IP công cộng ra quốc gia máy chủ VPN.

Không làm: tách VPN chỉ bằng tên Wi-Fi trên DHCP chung. Guest mặc định Asus chung DHCP với mạng nhà, Director chỉ nhìn dải IP. Không cài lại YazFi khi chỉ thiếu ACCEPT. Không bật `REDIRECTALLTOVPN=true` cùng Director. Không chạy hai WireGuard cùng `Address = 10.14.0.2/16`. Không dùng `/29` để cắt dải 50 IP trên LAN `.50`. Không dùng `ip route get 1.1.1.1 from 192.168.51.x` trên busybox Merlin (hay báo Invalid argument).

Đã thấy: 2026-09-19, hộp Asus Merlin 3004.388, khách một dải, nhà một dải, cổng OpenVPN. YazFi mặc định chỉ cho Guest ra card WAN (`eth0`). Tunnel (`tun11`) bị coi là không phải WAN nên bị chặn. Thiếu lỗ firewall thì có IP Guest mà không có internet.

### Bốn lớp

| Lớp | Sống | Chết |
|---|---|---|
| YazFi | `.51.1` trên `wl0.1`, có `YazFiFORWARD` | Máy `.50` |
| Director | `from .51/24 lookup ovpnc1` và `to .51/24 lookup main` | `.51` mà IP công cộng nhà mạng |
| Tunnel | `vpn_client1_state=2`, `tun11` UP | Connecting |
| ACCEPT | `-i wl0.1` / `-o wl0.1` trên REJECT | `.51` + tunnel, không vào `1.1.1.1` |

`1.1.1.1` được mà web chết = DNS.

### Chẩn nhanh

```sh
ip addr | grep -E "192.168.51|tun11"
ip rule | grep -E "51|ovpn|prohibit"
nvram get vpn_client1_state
iptables -L YazFiFORWARD -n -v | head -8
```

| Thấy gì | Nghĩa |
|---|---|
| Không có `192.168.51.1`, không có chain `YazFiFORWARD` | Chưa `runnow` |
| Máy `192.168.50.x` | Đang ở LAN chính, Director `.51` không dính |
| Có `.51` + `lookup ovpnc1` + `state=2` nhưng không ra `1.1.1.1` | Firewall YazFi / OVPNCF |
| `1.1.1.1` được, web chết | DNS |
| IP công cộng nhà mạng | Thiếu rule Director hoặc OpenVPN để No Internet traffic |

Hỏi trước khi vá: firmware và Operation mode; IP máy `.50` hay `.51`; `YazFiFORWARD` có không; `ip rule` có `lookup ovpnc1` không; `vpn_client1_state`; counter ACCEPT `wl0.1` / `.51` có tăng khi mở `http://1.1.1.1` không. Vá đúng lớp.

### Tình huống hay gặp

- Hai WireGuard cùng một nhà VPN: nhiều file `.conf` cùng `Address = 10.14.0.2/16`. Đổi thành phố hay tạo key mới không tách Address. Cách ổn: một WireGuard + một OpenVPN, hoặc chỉ một tunnel một lúc.
- Guest có net khi tắt VPN, mất net khi bật Director: đúng bệnh REJECT `wl0.1` không ra `eth0`. Thiếu userscript.
- Reboot ra IP nhà mạng: máy còn lease `.50` hoặc `runnow` chưa chạy. Đợi, Quên mạng. Vẫn không có `.51.1` thì `services-start` thiếu `runnow`.
- OpenVPN Connected mà Guest không đi VPN: client đang No Internet traffic. Đổi thành VPN Director rồi Apply.
- Muốn cắt dải trên cùng LAN `.50`: `/29` chỉ 8 địa chỉ và phải đúng biên mạng. `.100/29` sai nếu `.100` không phải địa chỉ mạng. Dải 50 IP không gói một prefix. Sạch hơn là Guest riêng `/24`. Gần đúng: `.96/27` = `.96`–`.127`; `.128/28` = `.128`–`.143`. Phần lẻ ghép thêm prefix hoặc gán tĩnh.
- iOS: tắt Private Wi-Fi Address trên SSID đó nếu đang gán IP theo MAC. Ô DNS để trống khi xài DHCP. Gõ IP tay mà quên DNS thì không kết nối được.

### Giới hạn

- Chế độ AP / Repeater không chia VPN. YazFi không chạy trên node AiMesh / AP. Firmware 3006.x và Guest Network Pro/VLAN là lối khác. YazFi nhánh AMTM-OSR không hỗ trợ 3006.102. AX68U Merlin 3004.388 thường không có WISP.
- Jack Yaz không còn bảo trì gốc; dùng nhánh AMTM-OSR, tự chịu rủi ro.
- OpenVPN chậm hơn WireGuard trên cùng hộp.
- Không gồm chia sẻ tài khoản VPN hay lách kiểm duyệt.

Thiếu dữ liệu: chưa đo lại trên firmware 3006.

## Sửa lỗi ghi hình trên máy tính bị màng hình đen

Việc: Sửa lỗi ghi hình trên máy tính bị màng hình đen

Làm:
1. Cần thiết bị bộ chia HDMI tách HDCP 1 vào 2 ra, kèm theo 2 dây HDMI mới.
2. Cần card ghi hình HDMI, ví dụ như Ugreen CM489 40189.
3. Từ cổng HDMI ra của card đồ họa -> kết nối bộ chia HDMI -> ..   
   -> cổng 1 vào màng hình thông thường  
   -> cổng 2 kết nối card ghi hình HDMI -> kết nối tới cổng USB máy tính  -> dùng phần mềm như OBS Studio ghi hình & âm thanh từ nguồn thiết bị là "card ghi hình"  
 
Lợi ích khác: Ngoài ra, có thể kết nối theo thứ tự sau để dùng thiết bị thứ 2 phát âm thanh & hình ảnh: Từ cổng HDMI thứ 2 ra của card đồ họa -> card ghi hình HDMI -> thiết bị 2 -> dùng phần mềm (VLC) mở chức năng thu hình & âm thanh từ "card ghi hình HDMI"  



## Example

Việc: một câu việc đã gặp.

Làm:
1. Bước đã làm.
2. Bước tiếp.

Không làm: việc đã thử và hỏng.

Đã thấy: ngày, hoàn cảnh. Cắt tên riêng, số hợp đồng, mail.
Thiếu dữ liệu: chỗ chưa đo.

Thêm nhóm mới bằng `##`. Giữ mục Example này làm khung, hoặc xóa khi đã có nhóm thật.

[edit](https://github.com/stoism/stoism.github.io/blob/main/pages/tech-tin-hoc.md)
