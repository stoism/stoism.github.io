# Hai Wi‑Fi trên Asuswrt-Merlin: một mạng nhà, một mạng đi VPN

Hướng dẫn cho firmware **Asuswrt-Merlin 3004.388** (ví dụ RT-AX68U). Mục tiêu: điện thoại đổi SSID là đổi đường ra internet — mạng chính đi nhà mạng, mạng Guest đi VPN (bài này lấy OpenVPN, ví dụ máy chủ Mỹ).

Không phải hướng dẫn phá khóa, vượt tường lửa bất hợp pháp, hay lấy trộm tài khoản VPN. Cần firmware Merlin, quyền SSH admin trên **router của mình**, và tài khoản VPN hợp lệ.

## Ý tưởng bằng một hình

Router giống cái ngã ba. Wi‑Fi nhà đứng một nhánh, Wi‑Fi Guest đứng nhánh khác. VPN Director chỉ nhìn **dải IP**, không nhìn tên Wi‑Fi. Guest mặc định của Asus **chung DHCP** với mạng nhà → hai SSID vẫn cùng dải → không tách được VPN.

Cần thêm **YazFi** để Guest có dải riêng (ví dụ `192.168.51.0/24`). Rồi Director nói: dải đó đi tunnel OpenVPN.

YazFi mặc định chỉ cho Guest ra card WAN (`eth0`). Tunnel (`tun11`) bị coi là “không phải WAN” nên bị chặn. Thiếu lỗ firewall thì có IP Guest mà không có internet.

## Cần trước

- Router Merlin, chế độ **Wireless Router**, có dây WAN (hoặc USB 4G). Chế độ AP / Repeater không chia VPN.
- Administration → System: bật JFFS custom scripts.
- SSH vào router (user admin, IP LAN thường `192.168.50.1` hoặc `192.168.1.1`).
- Tài khoản VPN có file `.ovpn` và **service credentials** (không phải email đăng nhập app — đúng với Surfshark; nhà khác xem tài liệu họ).
- Guest 2.4 GHz đã bật trên web Asus, đặt tên SSID + mật khẩu riêng. Ghi nhớ interface: Guest 2.4 GHz số 1 thường là `wl0.1`.

Firmware **3006.x** và Guest Network Pro/VLAN là lối khác. YazFi nhánh AMTM-OSR **không hỗ trợ 3006.102**. AX68U Merlin 3004.388 thường **không có WISP**.

## Bước 1 — OpenVPN + VPN Director

1. VPN → OpenVPN Client 1: tải file `.ovpn`, điền user/pass dịch vụ.
2. **Redirect Internet traffic through tunnel** = **VPN Director**. Không để *No Internet traffic* (Director sẽ không kéo gói). Không để *All* nếu chỉ muốn Guest đi VPN.
3. Killswitch: tắt lúc mới dựng. Bật sau khi đã ổn.
4. Bật client. VPN Status phải Connected. SSH: `nvram get vpn_client1_state` ra `2`. Interface thường là `tun11`.

## Bước 2 — Cài YazFi

SSH:

```sh
/usr/sbin/curl -fsL --retry 3 "https://raw.githubusercontent.com/AMTM-OSR/YazFi/master/YazFi.sh" -o /jffs/scripts/YazFi && chmod 0755 /jffs/scripts/YazFi && /jffs/scripts/YazFi install
```

Hoặc `amtm` → danh sách script → YazFi.

Sửa config Guest 1 (`wl01`):

- `ENABLED=true`
- `IPADDR=192.168.51.1` — phải là **cổng `.1`**, không phải `.0`
- `ALLOWINTERNET=true`
- `REDIRECTALLTOVPN=false` (bài này để Director cầm OpenVPN; YazFi REDIRECT chỉ nhắm số client OpenVPN, dễ lệch)

Lệnh dựng mạng đúng là:

```sh
/jffs/scripts/YazFi runnow
```

Không có lệnh `YazFi apply`. `YazFi startup` chỉ gắn menu web, **không** tạo dải `192.168.51`.

## Bước 3 — Lỗ firewall (bắt buộc)

Tạo file đúng thư mục YazFi:

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
```

Guest 5 GHz số 1 đổi `wl0.1` thành `wl1.1`. OpenVPN client 2 thường là `tun12`.

Hai dòng `-i wl0.1` / `-o wl0.1` quan trọng: `wl0.1` hay còn nằm bridge `br0`, rule chỉ theo `-s 192.168.51.0/24` có thể **0 packet**.

Không đặt script vào `/jffs/scripts/YazFi_user/` — đó không phải hook YazFi.

Chạy lại:

```sh
/jffs/scripts/YazFi runnow
```

Log phải có `Executing user script: .../guest-ovpn.sh`.

## Bước 4 — Hai rule VPN Director

| Bật | Local IP | Remote IP | Interface |
|---|---|---|---|
| Có | `192.168.51.0/24` | (trống) | OpenVPN 1 |
| Có | (trống) | `192.168.51.0/24` | WAN |

Rule trên: Guest **đi** tunnel. Rule dưới: gói **về** dải Guest theo bảng main.

OpenVPN client phải để routing = VPN Director.

SSH mong đợi:

```text
from all to 192.168.51.0/24 lookup main
from 192.168.51.0/24 lookup ovpnc1
```

`prohibit` cùng dải là killswitch. Tunnel lệch + YazFi REJECT = mất mạng.

## Bước 5 — Sống sau reboot

`/jffs/scripts/services-start`:

```sh
#!/bin/sh
/jffs/scripts/YazFi startup & # YazFi
(sleep 30; /jffs/scripts/YazFi runnow) &
```

```sh
chmod +x /jffs/scripts/services-start
```

Giữ dòng `YazFi startup` nếu file đã có. Quan trọng là thêm `runnow` trễ ~30 giây.

Sau reboot: đợi khoảng 2 phút, trên điện thoại **Quên mạng** Guest rồi bắt lại. Lease cũ `192.168.50.x` không tự nhảy sang `192.168.51.x`.

## Kiểm trên máy

- IP: `192.168.51.x`, router `192.168.51.1`
- Trình duyệt `http://1.1.1.1` (không qua ô tìm Google)
- Trang xem IP công cộng phải ra quốc gia máy chủ VPN

iOS: cấu hình Automatic, tắt Private Wi‑Fi Address trên SSID đó nếu đang gán IP theo MAC. Ô DNS để trống khi xài DHCP. Gõ IP tay mà quên DNS thì “không kết nối được”.

## Chẩn nhanh (SSH)

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
| IP công cộng nhà mạng | Thiếu rule Director hoặc OVPN để *No Internet traffic* |

Lệnh `ip route get 1.1.1.1 from 192.168.51.x` trên busybox Merlin hay báo Invalid argument — bỏ.

## Tình huống hay gặp

**Hai WireGuard cùng một nhà VPN (Surfshark và một số nhà khác)**  
Nhiều file `.conf` cùng `Address = 10.14.0.2/16`. Hai client WGC trên một hộp làm bảng route chồng. Đổi thành phố hay tạo key mới **không** tách Address. Cách ổn: một WireGuard + một OpenVPN, hoặc chỉ một tunnel một lúc.

**YazFi `REDIRECTALLTOVPN=true` + Director**  
Dễ đánh hai đường. Với WireGuard thì REDIRECT của YazFi không thay Director. Để `false` rồi dùng Director.

**Guest có net khi tắt VPN, mất net khi bật Director**  
Đúng bệnh REJECT `wl0.1` không ra `eth0`. Thiếu userscript.

**Reboot ra IP nhà mạng**  
Thường máy còn lease `.50` hoặc `runnow` chưa chạy. Đợi, Quên mạng. Vẫn không có `.51.1` thì `services-start` thiếu `runnow`.

**OpenVPN Connected mà Guest không đi VPN**  
Client đang *No Internet traffic*. Đổi thành VPN Director rồi Apply.

**Muốn cắt dải trên cùng LAN `.50` (`.100`–`.149` một server, `.150+` server khác)**  
`/29` chỉ 8 địa chỉ và phải đúng biên mạng. `.100/29` sai nếu `.100` không phải địa chỉ mạng. Dải 50 IP không gói một prefix. Sạch hơn: Guest riêng `/24` như bài này, hoặc reservation từng máy.

**Gói CIDR gần đúng trên `.50` (tham khảo, dễ chồng)**  
`.96/27` = `.96`–`.127`; `.128/28` = `.128`–`.143`. Phần lẻ còn lại phải ghép thêm prefix hoặc gán tĩnh.

## Việc bot / người sửa hộ nên hỏi

1. Firmware và Operation mode?  
2. IP máy: `.50` hay `.51`?  
3. `YazFiFORWARD` có không?  
4. `ip rule` có `lookup ovpnc1` không?  
5. `vpn_client1_state`?  
6. Counter ACCEPT `wl0.1` / `.51` có tăng khi mở `http://1.1.1.1` không?

Vá đúng lớp. Đừng cài lại YazFi khi chỉ thiếu ACCEPT.

## Giới hạn

- YazFi không chạy trên node AiMesh / AP.  
- Jack Yaz không còn bảo trì gốc; dùng nhánh AMTM-OSR, tự chịu rủi ro.  
- OpenVPN chậm hơn WireGuard trên cùng hộp.  
- Bài không bao gồm chia sẻ tài khoản VPN hay lách kiểm duyệt.

## Nguồn kỹ thuật

- Wiki VPN Director (Asuswrt-Merlin)  
- YazFi AMTM-OSR trên GitHub  
- Các thread SNBForums về YazFi + WireGuard/VPN Director: firewall YazFi chặn interface tunnel, lỗ đặt tại `/jffs/addons/YazFi.d/userscripts.d/`, chmod 0755
