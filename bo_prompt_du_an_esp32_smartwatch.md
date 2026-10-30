# Bộ Prompt Chi Tiết Cho Gemini: Dự Án Smartwatch ESP32 (OLED 0.96" I2C + Wi-Fi Portal + App Android BLE)

Tài liệu này cung cấp bộ câu lệnh (prompt) được tối ưu hóa để bạn sao chép và gửi trực tiếp cho Gemini. Để AI viết code chính xác, không bị tràn bộ nhớ hoặc cắt bớt logic, bạn nên gửi lần lượt từng phần theo thứ tự từ **Phần 1** đến **Phần 3** trong cùng một cuộc trò chuyện.

## Tổng Quan Thông Số & Giao Thức Thống Nhất Giữa Các Phần

Bảng dưới đây định nghĩa các thông số kỹ thuật cố định đã được nhúng sẵn vào cả 3 prompt để đảm bảo code ESP32 và code App Android khớp hoàn toàn với nhau khi ghép nối:

| Thành phần | Thông số kỹ thuật quy chuẩn trong Prompt |
| :--- | :--- |
| **Vi điều khiển & Màn hình** | ESP32 DevKit V1 kết nối OLED 0.96 inch I2C (`SSD1306`, độ phân giải `128x64`, địa chỉ `0x3C`, chân `SDA = GPIO 21`, `SCL = GPIO 22`) |
| **Thư viện đồ họa đề xuất** | `U8g2` hoặc `Adafruit_SSD1306` + `Adafruit_GFX` (kết hợp vẽ frame-buffer chống nhấp nháy, đạt tốc độ khung hình ~30 FPS) |
| **Cấu hình Wi-Fi Web Portal** | Chế độ `WIFI_AP_STA`, phát Wi-Fi tên `ESP32-SmartWatch`, IP tĩnh `192.168.4.1`, quét mạng Wi-Fi xung quanh và lưu vào `Preferences` (NVS Flash) |
| **Nguồn dữ liệu Thời tiết & Giờ** | Đồng bộ giờ qua `NTP` (`GMT+7`) và lấy thời tiết địa phương qua API miễn phí `Open-Meteo` (không cần API key phức tạp) hoặc `OpenWeatherMap` |
| **Giao thức Bluetooth** | Bluetooth Low Energy (`NimBLE-Arduino` để tiết kiệm RAM khi chạy song song với Wi-Fi), định dạng gói tin truyền nhận là chuỗi `JSON` UTF-8 |
| **Cấu trúc JSON truyền qua BLE** | Thông báo: `{"type":"notif","app":"Zalo","sender":"Tên","msg":"Nội dung"}` \| Chữ & Icon riêng: `{"type":"custom","text":"Tên ai đó","icon":"heart"}` |

## Phần 1: Prompt Thiết Kế GUI Smartwatch & Hiệu Ứng Chuyển Động (OLED 0.96" I2C)

> **Cách dùng:** Sao chép toàn bộ nội dung trong khung dưới đây và gửi cho Gemini để nhận mã nguồn xử lý giao diện, bitmap icon và các hiệu ứng hoạt ảnh trên màn hình.

```text
Hãy đóng vai một kỹ sư hệ thống nhúng (Embedded Systems Engineer) và chuyên gia thiết kế giao diện vi điều khiển (Microcontroller UI/UX Designer). Tôi đang làm một dự án thiết bị hiển thị thông minh giống đồng hồ thông minh (Smartwatch không cảm ứng, chỉ hiển thị) sử dụng vi điều khiển ESP32 kết nối với màn hình OLED 0.96 inch (128x64 pixel, giao tiếp I2C: SDA = GPIO 21, SCL = GPIO 22, driver SSD1306, địa chỉ 0x3C).

Hãy viết code C++ (Arduino Framework, ưu tiên thư viện U8g2 hoặc Adafruit_SSD1306 với cơ chế vẽ full-buffer không dùng delay() gây đứng hình) để tạo một giao diện GUI mượt mà, tinh tế và đầy đủ các tính năng sau:

1. BỐ CỤC MÀN HÌNH CHÍNH (HOME SCREEN - 128x64):
- Thanh trạng thái (Status Bar - 12px trên cùng):
  + Góc trái: Icon trạng thái kết nối Wi-Fi và icon Bluetooth (BLE) khi có điện thoại kết nối.
  + Góc phải: Hiển thị dung lượng pin còn lại dưới dạng biểu tượng viên pin kèm số phần trăm (%) hoặc điện áp (ví dụ: "85%" hoặc "3.9V") đọc từ chân ADC (GPIO 34) qua cầu phân áp.
- Khu vực trung tâm & bên trái:
  + Hiển thị đồng hồ số (Giờ:Phút cỡ lớn, Giây cỡ nhỏ) và Ngày/Tháng/Thứ ở phía dưới với font chữ hiện đại, rõ nét.
- Khu vực bên phải (Widget Thời tiết):
  + Hiển thị nhiệt độ địa phương (ví dụ: "29°C") và trạng thái thời tiết lấy từ Wi-Fi.
  + HIỆU ỨNG ĐỘNG (Weather Animations): Viết hàm vẽ hoạt ảnh mượt mà theo thời gian thực (dùng millis(), tốc độ ~25-30 FPS):
    * Trời nắng/quang: Hiệu ứng mặt trời có các tia sáng xoay tròn hoặc nhấp nháy nhẹ.
    * Trời nhiều mây/mưa: Hiệu ứng đám mây trôi bồng bềnh qua lại theo chiều ngang và hạt mưa rơi xuống.

2. HIỆU ỨNG THÔNG BÁO "3D CORNER POPUP" GIỐNG iOS (DYNAMIC BANNER):
- Khi có thông báo mới từ điện thoại gửi xuống (Zalo, Messenger, SMS...), đừng chỉ hiện chữ đột ngột. Hãy lập trình hiệu ứng chuyển động mượt mà kiểu 3D/iOS:
  + Giai đoạn 1 (Expand): Một khung bo góc (Rounded Box có đổ bóng giả lập 3D bằng đường viền dày ở cạnh dưới và cạnh phải) phóng to dần từ góc trên màn hình (từ kích thước 10x6 pixel bung mượt ra kích thước 124x54 pixel) sử dụng hàm nội suy mượt (ease-out).
  + Giai đoạn 2 (Display & Auto-Scroll): Hiển thị bên trong khung thông báo gồm:
    * Góc trên trái: Logo bitmap (12x12 hoặc 16x16 pixel) tương ứng với tên ứng dụng (tạo sẵn mảng PROGMEM XBM/bitmap cho logo Zalo, Messenger, cuộc gọi/tin nhắn chung).
    * Tiêu đề: Tên ứng dụng + Tên người gửi (in đậm hoặc đảo màu nền).
    * Nội dung tin nhắn: Nếu nội dung tin nhắn dài quá 1 dòng hoặc vượt quá chiều cao khung hiển thị, sau 1.5 giây đứng yên để người dùng đọc dòng đầu, GUI tự động cuộn dọc (smooth vertical scrolling từng pixel từ dưới lên) với tốc độ chậm vừa phải để người xem đọc hết toàn bộ tin nhắn.
  + Giai đoạn 3 (Collapse): Sau khi cuộn hết tin nhắn (hoặc sau 6-8 giây), khung thông báo thu nhỏ mượt mà trở lại về góc màn hình rồi biến mất, trả lại màn hình chính (Home Screen).

3. CHẾ ĐỘ HIỂN THỊ CHỮ TÙY CHỈNH & ICON DỄ THƯƠNG (CUSTOM MESSAGE & CUTE ICONS):
- Tạo một chế độ hiển thị đặc biệt khi nhận lệnh từ điện thoại (ví dụ hiển thị tên một người đặc biệt hoặc lời nhắn ngắn giữa màn hình) kèm theo các icon hoạt họa dễ thương (tạo sẵn mảng bitmap 24x24 pixel có hiệu ứng nhấp nháy/đập nhịp cho các icon: Trái tim yêu thương, Mặt cười tít mắt, Mặt khóc nhè, Ngôi sao lấp lánh).

YÊU CẦU ĐẦU RA:
- Cung cấp mã nguồn đầy đủ, chia rõ các mảng dữ liệu Bitmap (PROGMEM) cho logo (Zalo, Messenger), thời tiết (Mây, Mặt trời, Mưa) và bộ icon dễ thương (Cười, Tim, Yêu, Khóc).
- Viết các hàm quản lý trạng thái UI (State Machine) không chặn (non-blocking) bằng millis().
```

## Phần 2: Prompt Lập Trình Firmware ESP32 Hoàn Chỉnh (Wi-Fi Captive Portal + Weather + BLE)

> **Cách dùng:** Sau khi Gemini tạo xong phần giao diện ở Phần 1, hãy gửi tiếp prompt dưới đây để Gemini tích hợp phần mạng Wi-Fi (phát AP `192.168.4.1`, trang web quét và chọn Wi-Fi), lấy dữ liệu thời tiết/ngày giờ thực tế và nhận dữ liệu Bluetooth từ điện thoại.

```text
Tiếp tục dự án ESP32 Smartwatch ở trên, bây giờ hãy viết toàn bộ phần Firmware hoàn chỉnh cho ESP32 kết hợp giao diện GUI ở Phần 1 với hệ thống kết nối Wi-Fi Web Portal và Bluetooth Low Energy (BLE). Hãy tuân thủ chính xác luồng hoạt động sau:

1. CƠ CHẾ KHỞI ĐỘNG & PHÁT WI-FI CẤU HÌNH (WEB PORTAL 192.168.4.1):
- Khi ESP32 mới khởi động, kiểm tra thông tin Wi-Fi (SSID, Password) đã lưu trong bộ nhớ Flash (sử dụng thư viện Preferences).
- Nếu chưa có Wi-Fi hoặc kết nối thất bại sau 10 giây:
  + ESP32 chuyển sang phát một mạng Wi-Fi riêng (SoftAP) tên là "ESP32-SmartWatch" (không mật khẩu hoặc mật khẩu tùy chọn), gán địa chỉ IP tĩnh là 192.168.4.1.
  + Trên màn hình OLED 0.96 inch hiển thị hướng dẫn trực quan: Tên Wi-Fi phát ra và địa chỉ IP "192.168.4.1" để người dùng biết và kết nối.
  + Khởi tạo một Web Server nhẹ tại địa chỉ http://192.168.4.1:
    * Khi tôi dùng điện thoại kết nối vào Wi-Fi của ESP32 và truy cập địa chỉ 192.168.4.1 trên trình duyệt, ESP32 sẽ tự động quét (WiFi.scanNetworks()) toàn bộ các mạng Wi-Fi xung quanh.
    * Trang web có giao diện HTML/CSS gọn gàng, đẹp mắt, tối ưu cho màn hình điện thoại: Hiển thị danh sách các mạng Wi-Fi vừa quét được (kèm cường độ sóng RSSI). Khi bấm chọn tên một mạng Wi-Fi trong danh sách, tên mạng tự điền vào ô SSID, người dùng chỉ cần nhập mật khẩu và bấm "Kết nối". Có thêm nút "Quét lại mạng".
    * Sau khi nhận SSID và mật khẩu, ESP32 lưu vào bộ nhớ Preferences, kết nối vào mạng Wi-Fi nhà và thông báo thành công trên trang web.

2. ĐỒNG BỘ DỮ LIỆU QUA WI-FI (NGÀY GIỜ & THỜI TIẾT ĐỊA PHƯƠNG):
- Khi đã kết nối Wi-Fi thành công:
  + Tự động đồng bộ thời gian thực qua NTP Server (múi giờ Việt Nam GMT+7: pool.ntp.org).
  + Định kỳ mỗi 15-30 phút gọi HTTP GET tới API thời tiết (sử dụng Open-Meteo API theo tọa độ địa phương hoặc OpenWeatherMap) để lấy nhiệt độ hiện tại (°C) và mã trạng thái thời tiết (nắng, mây, mưa), sau đó cập nhật lên GUI ở Phần 1.

3. KẾT NỐI BLUETOOTH (BLE SERVER) VỚI APP ANDROID:
- Ngay sau khi kết nối Wi-Fi thành công, khởi tạo BLE Server (ưu tiên dùng thư viện NimBLE-Arduino hoặc cấu hình phân vùng Huge APP để Wi-Fi và BLE hoạt động mượt mà cùng lúc không bị thiếu RAM).
- Đặt tên thiết bị Bluetooth là "ESP32-SmartWatch-BLE", khai báo rõ Service UUID và Characteristic UUID (hỗ trợ thuộc tính WRITE).
- Nhận dữ liệu định dạng chuỗi JSON từ App Android gửi sang với 2 chế độ:
  + Chế độ 1 - Thông báo điện thoại:
    JSON mẫu: {"type":"notif","app":"Zalo","sender":"Nguyễn Văn A","msg":"Tối nay đi cà phê nhé bạn ơi!"}
    -> Kích hoạt hiệu ứng 3D Corner Popup từ góc màn hình, hiển thị logo tương ứng (Zalo/Messenger/...), tên người gửi và tự động cuộn dọc nếu tin nhắn dài.
  + Chế độ 2 - Hiển thị chữ tùy chỉnh & Icon dễ thương:
    JSON mẫu: {"type":"custom","text":"Yêu em nhiều!","icon":"heart","duration":15}
    -> Chuyển màn hình sang chế độ hiển thị thông điệp riêng kèm icon hoạt họa (smile, heart, love, cry...) theo yêu cầu từ App.

YÊU CẦU ĐẦU RA:
- Cung cấp trọn bộ code Arduino (.ino) ghép nối hoàn chỉnh cả Web Server cấu hình Wi-Fi, NTP, Weather HTTP Client, BLE Server (ArduinoJson) và bộ khung GUI của Phần 1.
- Hướng dẫn rõ danh sách các thư viện cần cài đặt trong Arduino IDE / PlatformIO và cách chọn Partition Scheme ("Huge APP (3MB No OTA/1MB SPIFFS)") để tránh lỗi đầy bộ nhớ khi dùng chung Wi-Fi và BLE.
```

## Phần 3: Prompt Lập Trình App Android (Bắt Thông Báo Zalo/Messenger, Gửi Chữ/Icon & Xuất APK)

> **Cách dùng:** Sau khi hoàn tất code cho ESP32, hãy gửi prompt thứ 3 này để Gemini viết mã nguồn ứng dụng Android và hướng dẫn bạn đóng gói ra file `.apk` cài đặt lên điện thoại.

```text
Bây giờ hãy đóng vai một Senior Android Developer. Hãy thiết kế và viết mã nguồn hoàn chỉnh cho ứng dụng Android kết nối với thiết bị "ESP32-SmartWatch-BLE" ở Phần 2 qua Bluetooth Low Energy (BLE). Ứng dụng cần có giao diện hiện đại, tinh tế, dễ sử dụng và xuất ra được file .apk để cài đặt trực tiếp.

Hãy sử dụng Kotlin + Jetpack Compose (Material Design 3) hoặc Flutter (hãy chọn giải pháp giúp tôi dễ dàng build ra file APK nhất và cung cấp đầy đủ các file mã nguồn chính):

1. GIAO DIỆN NGƯỜI DÙNG (UI/UX HIỆN ĐẠI & TINH TẾ):
- Thiết kế theo phong cách Smartwatch Companion App hiện đại (bo góc mềm mại, bố cục chia thẻ Card rõ ràng, hỗ trợ Dark/Light mode):
  + Thẻ Trạng thái kết nối (Connection Card): Hiển thị trạng thái kết nối BLE với ESP32 ("Đã kết nối" / "Đang tìm kiếm..."), nút Quét & Kết nối nhanh tới thiết bị "ESP32-SmartWatch-BLE" với đúng Service UUID và Characteristic UUID đã định nghĩa ở Phần 2.
  + Thẻ Quản lý Thông báo (Notification Sync Card):
    * Nút kiểm tra và mở nhanh màn hình cấp quyền truy cập thông báo của Android (Notification Listener Settings).
    * Danh sách công tắc bật/tắt (Switch) cho phép chọn lọc ứng dụng muốn đẩy thông báo lên màn hình ESP32: Zalo (com.zing.zalo), Messenger (com.facebook.orca), Facebook (com.facebook.katana), SMS/Tin nhắn mặc định, Telegram...
  + Thẻ Tính năng Đặc biệt - Viết Chữ & Gửi Icon Dễ Thương (Custom Display Studio):
    * Một ô nhập văn bản (TextField) cho phép người dùng gõ tên một ai đó hoặc lời nhắn ngắn muốn hiển thị riêng trên màn hình OLED của ESP32.
    * Bộ chọn Icon dễ thương trực quan (Grid/Row các nút chọn icon): Cười (smile), Trái tim (heart), Yêu thương (love), Khóc (cry), Ngôi sao (star)...
    * Tuỳ chọn chế độ hiển thị (hiển thị tạm thời trong vài giây hoặc ghim cố định trên màn hình đồng hồ) và nút "Gửi lên màn hình ESP32".

2. XỬ LÝ NGẦM BẮT THÔNG BÁO ĐIỆN THOẠI (NOTIFICATION LISTENER SERVICE):
- Viết class kế thừa `NotificationListenerService` của Android để lắng nghe thời gian thực mọi thông báo đến từ hệ thống.
- Khi có thông báo mới từ các ứng dụng được bật (như Zalo, Messenger):
  + Trích xuất chính xác: Tên ứng dụng (ví dụ: `com.zing.zalo` -> `"Zalo"`, `com.facebook.orca` -> `"Messenger"`), Tên người gửi (`android.title`) và Nội dung tin nhắn (`android.text`).
  + Lọc bỏ các thông báo hệ thống rác hoặc thông báo trùng lặp ("Đang chạy nền", "Chat heads active"...).
  + Đóng gói thành chuỗi JSON chuẩn: `{"type":"notif","app":"Zalo","sender":"...","msg":"..."}` (hỗ trợ xử lý tiếng Việt có dấu hoặc hàm chuyển đổi tiếng Việt/chia nhỏ gói tin BLE MTU 256-512 bytes để không bị đứt đoạn tin nhắn dài) và tự động gửi xuống ESP32 qua BLE.

3. CẤU HÌNH QUYỀN & HƯỚNG DẪN XUẤT FILE APK:
- Cung cấp đầy đủ file `AndroidManifest.xml` với tất cả các quyền cần thiết cho Android 12/13/14+ (`BLUETOOTH_SCAN`, `BLUETOOTH_CONNECT`, `ACCESS_FINE_LOCATION`, `BIND_NOTIFICATION_LISTENER_SERVICE`, `FOREGROUND_SERVICE`).
- Hướng dẫn từng bước chi tiết để tôi tạo project trong Android Studio, dán code vào và bấm `Build -> Build Bundle(s) / APK(s) -> Build APK(s)` để lấy file `.apk` cài vào điện thoại.
```

## Mẹo Nhỏ Khi Làm Việc Với Gemini Để Dự Án Chạy Mượt Nhất

* **Chuyển đổi tiếng Việt trên màn hình OLED 0.96 inch:** Màn hình OLED 128x64 có bộ nhớ font giới hạn. Nếu bạn muốn hiển thị đầy đủ tiếng Việt có dấu từ Zalo/Messenger, khi gửi **Phần 1** và **Phần 2**, bạn có thể nhắc Gemini sử dụng bộ font `u8g2_font_unifont_t_vietnamese1` hoặc `u8g2_font_cu12_t_vietnamese1` của thư viện `U8g2`. Nếu muốn chữ to và sắc nét hơn nữa, bạn có thể yêu cầu App Android ở **Phần 3** tự động chuyển tiếng Việt có dấu sang không dấu trước khi gửi qua Bluetooth.
* **Xử lý tràn bộ nhớ ESP32 khi bật cả Wi-Fi và Bluetooth:** Cả thư viện Wi-Fi và Bluetooth mặc định của ESP32 đều khá nặng. Trong **Phần 2**, prompt đã chỉ định rõ cho AI sử dụng thư viện **`NimBLE-Arduino`** (nhẹ hơn thư viện BLE gốc tới 60% RAM) và hướng dẫn chọn phân vùng **`Huge APP (3MB No OTA)`** trong Arduino IDE để bạn nạp code không bao giờ gặp lỗi thiếu bộ nhớ.
* **Đo phần trăm Pin chính xác:** Nếu bạn dùng pin Li-ion 3.7V (đầy là 4.2V), bạn chỉ cần nối cực dương của pin qua 2 điện trở giống nhau (ví dụ 2 điện trở `100kΩ` mắc nối tiếp tạo cầu phân áp chia đôi điện áp) rồi đưa điểm giữa vào chân `GPIO 34` của ESP32 như đã mô tả sẵn trong **Phần 1**.
