### 🛠️ Bước 1: Cấu hình Port Forwarding trên pfSense

1.  Mở giao diện WebGUI của pfSense.
    
2.  Trên thanh menu, chọn **Firewall** -> **NAT**.
    
3.  Tại tab **Port Forward**, bấm nút **Add** (nút mũi tên hướng lên để thêm luật lên đầu).
    
4.  Cấu hình các thông số sau (các phần khác giữ nguyên mặc định):
    
    - **Interface:** `WAN` (Vì traffic từ Kali đi vào qua cổng này).
        
    - **Address Family:** `IPv4`.
        
    - **Protocol:** `TCP` (Dành cho Web HTTP/HTTPS).
        
    - **Destination:** Chọn `WAN address`.
        
    - **Destination port range:** Nếu Web Server chạy HTTP, chọn `HTTP` (Port 80) cho cả phần *From port* và *To port*.
        
    - **Redirect target IP:** Nhập địa chỉ IP tĩnh của máy Windows đang chạy Web Server.
        
    - **Redirect target port:** Chọn `HTTP` (Port 80).
        
    - **Description:** Ghi chú cho dễ nhớ (VD: `NAT to Windows Web Server`).
        
5.  Cuộn xuống dưới cùng, đảm bảo mục **Filter rule association** đang để là `Add associated filter rule`. (Điều này rất quan trọng, nó sẽ bảo pfSense tự động tạo một rule bên Firewall để cho phép traffic này đi qua).
    
6.  Bấm **Save** và sau đó bấm **Apply Changes**.
    

* * *

### 🛠️ Bước 2: Khắc phục "Bẫy ngầm" trên máy Windows

Đây là bước mà rất nhiều kỹ sư mạng hay quên! Bản thân máy Windows 10 có một tường lửa riêng là **Windows Defender Firewall**. Theo mặc định, nó thường chặn các kết nối đến từ một dải mạng lạ (không cùng subnet).

Vì vậy, dù pfSense đã chuyển hướng traffic thành công, máy Windows có thể vẫn "đóng cửa từ chối". Khoa cần xử lý trên máy Windows:

1.  Mở **Windows Defender Firewall with Advanced Security** trên máy Windows.
    
2.  Chọn **Inbound Rules** -> Bấm **New Rule...** ở cột bên phải.
    
3.  Chọn **Port** -> Next.
    
4.  Trỏ vào **TCP** và gõ số **80** vào ô *Specific local ports* -> Next.
    
5.  Chọn **Allow the connection** -> Next.
    
6.  Tích chọn cả 3 ô (Domain, Private, Public) -> Next.
    
7.  Đặt tên (VD: `Allow Web Server Port 80`) và bấm **Finish**.
    

* * *

### 🚀 Bước 3: Kiểm tra từ Kali Linux

Khi mọi thứ đã thiết lập xong, Khoa quay trở lại máy Kali Linux.

Mở trình duyệt trên Kali hoặc dùng lệnh `curl` và truy cập vào **IP WAN của pfSense** (ví dụ: `http://192.168.164.132`), chứ **KHÔNG PHẢI** IP của máy Windows.