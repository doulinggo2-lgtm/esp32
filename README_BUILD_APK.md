# Hướng Dẫn Xuất & Cài Đặt File APK Cho Dự Án ESP32 Smartwatch BLE

Gói nén này chứa toàn bộ mã nguồn ứng dụng Android gốc (**Kotlin + Jetpack Compose + NotificationListenerService + BLE GATT Client**) của dự án **ESP32 Smartwatch OLED 0.96" I2C**.

## Cách 1: Cài đặt trực tiếp App Di Động ngay trên điện thoại (Nhanh nhất - Không cần máy tính)
1. Mở đường dẫn Web App của dự án trên trình duyệt **Chrome** (Android) hoặc **Safari** (iOS).
2. Bấm nút **"Cài App Lên Điện Thoại"** trên thanh công cụ (hoặc chọn **Thêm vào màn hình chính / Cài đặt ứng dụng**).
3. Ứng dụng sẽ xuất hiện ngoài màn hình chính điện thoại như một App độc lập, hỗ trợ **Kết nối Bluetooth BLE trực tiếp tới ESP32** (qua Web Bluetooth API trên Chrome Android) để gửi thông báo và viết chữ + icon dễ thương lên màn hình OLED!

## Cách 2: Tự động nén ra file `.apk` bằng GitHub Actions (Không cần cài Android Studio)
1. Tạo 1 repository mới trên GitHub và tải toàn bộ các file trong gói ZIP này lên.
2. Vào tab **Actions** trên GitHub -> chọn workflow **Build ESP32 Smartwatch Android APK** -> bấm **Run workflow**.
3. Sau khoảng 2 phút, tải file **`app-debug.apk`** trong mục **Artifacts** về điện thoại và bấm cài đặt!

## Cách 3: Xuất file `.apk` bằng Android Studio trên máy tính
1. Giải nén file ZIP này và mở thư mục bằng **Android Studio**.
2. Trên thanh menu, chọn **Build -> Build Bundle(s) / APK(s) -> Build APK(s)**.
3. Lấy file `.apk` tại `app/build/outputs/apk/debug/app-debug.apk` và chép vào điện thoại Android để cài đặt.
