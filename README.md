# ESP32-C3_control_Panasonic-CS-223DF-W
Dưới đây là dự án riêng biệt cho ESP32-C3 điều khiển Panasonic CS-223DF-W qua UART → MAX485 → level shift 3.3V → cổng điều khiển nội unit (thường là cổng CN-CNT / terminal static trên máy trong Panasonic, không phải DIII-NET như Daikin).

1. Lưu ý phần cứng
thành phần	chi tiết
ESP32-C3 UART	TX = GPIO18, RX = GPIO19
MAX485 DE/RE	GPIO21 (điều khiển chuyển gửi/nhận)
Baud rate	Panasonic cổng điều khiển thường dùng 2400 bps, 8N1 (nếu không nhận được byte nào, thử đổi sang 9600)
Cổng máy trong	Tìm cổng điều khiển ngoài (thường nhãn CN-CNT, TERMINAL, hoặc cặp dây ngoài ra từ mainboard máy trong) – xem service manual model CS-223DF-W
GND	Phải nối chung ESP32, MAX485, level shift và máy lạnh

2. Firmware quét (sniffer) – nạp trước để lấy frame thực tế: Tên file: panasonic_sniffer.yaml
Cách dùng:
Nạp file sniffer vào ESP32-C3.
Mở log ESPHome.
Bấm từng nút trên remote Panasonic: Bật, Tắt, Làm mát, Sưởi, Giữ ấm, Quạt, Nhiệt độ 18/25/30, Gió tự động, Swing...
Ghi lại các dòng hex RX 0x.. in ra. Một bấm nút thường sinh ra một burst (nhiều byte liên tiếp).
Sau khi có đủ frame, dùng file bên dưới để điều khiển.

3. Firmware điều khiển (điền frame thực tế sau khi đã quét) vào Tên file: panasonic_control.yaml

4. Câu lệnh hoàn thiện sau khi quét
Nạp panasonic_sniffer.yaml, mở log, bấm remote → copy các dòng hex.
Thay tất cả {0xXX, 0xXX, 0xXX} trong panasonic_control.yaml bằng mảng byte thực tế.
Viết đoạn parser trong khối while (id(uart_panasonic).available()) theo cấu trúc frame Panasonic thật (đầu/cuối frame, byte địa chỉ, byte lệnh, byte dữ liệu).
Nạp và kiểm tra trên Home Assistant: switch, select, number, text_sensor sẽ xuất hiện và phản hồi 2 chiều khi máy trong gửi trạng thái ngược lại UART.
