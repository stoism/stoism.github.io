# Trải nghiệm MacOS 15.6  

## Lưu ý
- Cân nhắc thật kỹ nhu cầu, vì đã mua mày thì phải mua thêm phần mềm bản quyền.
- Các ứng dụng công việc tại Việt Nam đa số chạy trên Windows, MacOS không chạy được gì nhiều ngoài duyệt web. Có thể cài Windows ARM (bản thiết kế riêng cho dòng chip Apple này) nhưng khi chạy ứng dụng x86, x64 thông thường trên PC dễ gây nóng máy (dễ thấy tiếc).

## Định nghĩa  
- Vẫn có thể cài phần mềm ngoài Appstore.  
- Một số phần mềm lớn vẫn có thuốc.  
- Chip (CPU) M1, M2, M3... là chip cấu trúc ARM64, tức nó có thể chạy ứng dụng đơn giản như trên điện thoại. Phần mềm trên máy tính Windows thông thường có thể chạy qua giả lập bằng phần mềm Parallels. Trên Parallels có thể cài Windows 11 ARM, từ Windows 11 ARM chạy các ứng dụng 32bit (x86) và 64 bit (x64). Tuy nhiên, chạy ứng dụng 64 bit làm nặng máy nhất, chạy AutoCAD 2018 64 bit có thể làm CPU nóng đến 90 độ C.  
- Có thể xài chuột và bàn phím không phải của Apple  
- File *.dmg là file nén, khi mở lên, kéo icon ứng dụng vào Application tức là cài app.  

## Thao tác  
Xài xong gập máy lại là xong.  


### Phím tắt  
Phím Command (⌘) là "chìa khóa" chính trên macOS, tương tự như phím Control (Ctrl) trên Windows.  
Command (⌘) + C / V / X: Sao chép / Dán / Cắt.  
Command (⌘) + A: Chọn tất cả.  
Command (⌘) + Z: Hoàn tác (Undo).  
Command (⌘) + Q: Thoát hẳn ứng dụng đang mở. Đây là phím tắt bạn nên ghi nhớ để không chỉ đóng cửa sổ mà còn tắt ứng dụng hoàn toàn.  
Command (⌘) + Option + Esc: Buộc thoát ứng dụng (Force Quit), hữu ích khi một ứng dụng bị treo.  
Command (⌘) + W: Đóng cửa sổ đang mở, nhưng ứng dụng vẫn chạy ngầm.  
Command (⌘) + Space (phím cách): Mở Spotlight Search, công cụ tìm kiếm nhanh bất kỳ file, ứng dụng hoặc thông tin nào trên máy.  
Command (⌘) + Tab: Chuyển đổi qua lại giữa các ứng dụng đang mở.  
Command (⌘) + Shift + 3: Chụp toàn bộ màn hình.  
Command (⌘) + Shift + 4: Chụp một vùng màn hình tùy chọn.  
Command (⌘) + Option (⌥) + Delete: xóa vĩnh viễn file  
Command (⌘) + Delete: bỏ thùng rác  
Vuốt 4 ngón ra > Desktop  
Vuốt 4 ngón vào > App list  

### Chỉnh Trackpad  
1. Bấm nhẹ: Tap to click > ON  
2. Natural scrolling > OFF  

### Kéo thả bằng 3 ngón tay (không dùng lực)  
1. Mở System Settings (Cài đặt hệ thống).  
2. Chọn Accessibility (Trợ năng) ở thanh bên trái.  
3. Chọn Pointer Control (Điều khiển con trỏ) ở phía bên phải.  
4. Nhấn vào nút Trackpad Options... (Tùy chọn Bàn di chuột...).  
5. Bật tùy chọn Use trackpad for dragging (Sử dụng bàn di chuột để kéo).  
6. Trong menu thả xuống bên cạnh Dragging style (Kiểu kéo), chọn Three Finger Drag (Kéo bằng ba ngón tay).  
7. Nhấn OK để lưu lại.

### Thao tác khác  



## Cài Homebrew (chạy code)  
Spotlight Search (nhấn Command (⌘) + Spacebar) và gõ Terminal  
```xcode-select --install```  
Một cửa sổ pop-up sẽ hiện ra. Bạn chỉ cần làm theo hướng dẫn trên màn hình để cài đặt.  
```/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"```  
Với MacBook dùng chip Apple Silicon (M1, M2, M3), thiết lập biến môi trường (PATH) giúp bạn có thể sử dụng lệnh brew từ bất kỳ thư mục nào:  
```(echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> ~/.zprofile```  
```eval "$(/opt/homebrew/bin/brew shellenv)"```  
Kiểm tra lại việc cài đặt: ```brew --version```  

Một số app nên cài  
```brew install --cask evkey```  
```brew install --cask onedrive```  
```brew install --cask surfshark```  
```brew install --cask parsec```  
```brew install --cask parallels```  
```brew install --cask rectangle``` (phím tắt chia cửa số như trên Windows)  
```brew install --cask vlc```  
```brew install --cask istat-menus``` (giống TaskManager, hiện ở thanh menu)  
```brew install --cask Karabiner-Elements``` (thêm phím tắt)  
```brew install --cask motrix``` (quản lý download)   
```brew install --cask keka``` (nén file)   
```brew install --cask fuse-t``` -> cài https://www.veracrypt.io/en/Downloads.html  


### Cài EVkey  
1. Tải https://evkeyvn.com  
2. Setting > Privacy & Security > trong phần Security bấm "Open Anyway" EVKey (cho phép EVKey chạy)  
3. Cấp quyền bàn phím: Setting > Privacy & Security > Accessibility > bật EVKey  
4. Sau khi EVKey chạy, chọn thiết lập từ icon góc phải  
  
### Tùy chỉnh vị trí profile trình duyệt Brave
1. Cấp quyền cho nhu liệu  
```sudo chown -R $(whoami):staff "/Volumes/DISK/Program/ProfileA"```  
```chmod -R 755 "/Volumes/DISK/Program/ProfileA"```  

2. Chạy lệnh Teminal hoặc dùng nhu liệu "Automator" → New → Application → thêm Action "Run Shell Script" → paste lệnh trên vào → Save thành app riêng (ví dụ "Brave ProfileA.app")  
```"/Applications/Brave Browser.app/Contents/MacOS/Brave Browser" --user-data-dir="/Volumes/DISK/Program/ProfileA" --no-default-browser-check```  
hoặc  
```open -a "Brave Browser" --args --user-data-dir="/Volumes/DISK/Program/ProfileA" --no-default-browser-check```  
  
  
## Kiểm tra máy mới mua  
System Report (Báo cáo hệ thống): Apple > System Settings (Cài đặt hệ thống) > General (Chung) > nhấp vào nút System Report (Báo cáo hệ thống)  

### Kiểm tra phần cứng chung  
1. Tắt hoàn toàn MacBook M1 của bạn.  
2. Bật lại máy và nhấn giữ phím nguồn. Tiếp tục giữ cho đến khi bạn thấy màn hình "Loading startup options" (Đang tải các tùy chọn khởi động).  
3. Sau đó, bạn sẽ thấy một biểu tượng bánh răng cưa có nhãn "Options" (Tùy chọn). Không chọn, nhấn giữ Command (⌘) + D để khởi động Apple   Diagnostics.  
4. Quá trình kiểm tra sẽ tự động bắt đầu và mất vài phút. Một thanh tiến trình sẽ hiển thị trên màn hình.  
5. Để thoát, bạn có thể chọn "Restart" (Khởi động lại) hoặc "Shut Down" (Tắt máy).  








[edit](https://github.com/stoism/stoism.github.io/blob/main/pages/trainghiemmacos.md)
