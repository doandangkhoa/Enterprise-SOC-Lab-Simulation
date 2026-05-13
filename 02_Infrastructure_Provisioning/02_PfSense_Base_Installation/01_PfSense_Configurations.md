# Cấu hình pfSense qua Console

**Bước 1: Bật máy ảo pfSense lên.**

1.  Khoa nhấn nút Play (Start up this guest operating system).
    
2.  Chờ nó chạy xong đống chữ trắng trên nền đen. Khi nào nó hiện ra **Menu gồm các số từ 0 đến 16**, thì chuyển sang Bước 2.
    

**Bước 2: Gán cổng mạng (Assign Interfaces)**

1.  Tại Menu chính, Khoa gõ phím **`1`** (Assign Interfaces) rồi nhấn Enter.
    
2.  Hệ thống hỏi `Should VLANs be set up now?` -> Gõ **`n`** rồi Enter.
    
3.  Nó sẽ liệt kê 3 tên card mạng (thường là `em0`, `em1`, `em2` hoặc `le0`, `le1`, `le2`).
    
4.  Nó hỏi `Enter the WAN interface name` -> Bạn gõ tên card mạng đầu tiên (ví dụ **`em0`**) -> Enter.
    
5.  Nó hỏi `Enter the LAN interface name` -> Bạn gõ tên card mạng thứ hai (ví dụ **`em1`**) -> Enter.
    
6.  Nó hỏi `Enter the Optional 1 interface name` (Đây chính là DMZ) -> Bạn gõ tên card mạng thứ ba (ví dụ **`em2`**) -> Enter.
    
7.  Nó hỏi lại thông tin vừa nhập, bạn gõ **`y`** để xác nhận lưu.
    

**Bước 3: Đặt địa chỉ IP tĩnh cho từng cổng** Khi đã quay lại Menu chính, bạn gõ phím **`2`** (Set interface(s) IP address).

- **👉 Đặt IP cho LAN (vmnet2):**
    
    1.  Chọn số tương ứng với **LAN** (Thường là phím số `2`).
        
    2.  `Configure IPv4 address LAN interface via DHCP?` -> Gõ **`n`**.
        
    3.  `Enter the new LAN IPv4 address` -> Gõ **`192.168.10.1`** (Vì bạn đang giữ dải cũ cho Wazuh).
        
    4.  `Enter the new LAN IPv4 subnet bit count` -> Gõ **`24`**.
        
    5.  `For a WAN, enter the new LAN IPv4 upstream gateway address` -> **Nhấn Enter luôn** (bỏ trống).
        
    6.  Các câu hỏi về IPv6 Khoa cứ nhấn Enter bỏ qua.
        
    7.  `Do you want to enable the DHCP server on LAN?` -> Gõ **`y`**, nhập dải từ `192.168.10.100` đến `192.168.10.200` (Để lỡ máy ảo khác cần IP).
        
    8.  `Do you want to revert to HTTP as the webConfigurator protocol?` -> Gõ **`n`**.
        
- **👉 Đặt IP cho DMZ (OPT1 - vmnet3):**
    
    1.  Ở Menu chính, gõ lại phím **`2`**.
        
    2.  Chọn số tương ứng với cổng **OPT1** (Thường là phím số `3`).
        
    3.  `Configure IPv4 address OPT1 interface via DHCP?` -> Gõ **`n`**.
        
    4.  `Enter the new OPT1 IPv4 address` -> Gõ **`172.16.10.1`**.
        
    5.  `Enter the new OPT1 IPv4 subnet bit count` -> Gõ **`24`**.
        
    6.  Bỏ trống IPv4 Gateway và IPv6 như trên.
        
    7.  `Do you want to enable the DHCP server on OPT1?` -> Gõ **`y`**, nhập dải từ `172.16.10.100` đến `172.16.10.200`.
        

# Cấu hình Pfsense trên giao diện Web

### Bật pfSense Firewall rồi mở Windows Host, truy cập vào Internet rồi Search : `https://192.168.10.1` sẽ hiện ra trang web để cấu hình firewall

<img src="../../_resources/925321fe98e06c3fadc54e2b5e819bfe.png" alt="925321fe98e06c3fadc54e2b5e819bfe.png" width="1027" height="482" class="jop-noMdConv">

### **Thiết lập Timezone** và đảm bảo các thiết bị khác cũng có cùng TimeZone này.

<img src="../../_resources/66bf4a043e81945f64c159b0c136542a.png" alt="66bf4a043e81945f64c159b0c136542a.png" width="977" height="230">

### **Tiếp theo tới phần Setup cho WAN Interface**

<img src="../../_resources/d63a7247c9781a984503878286959311.png" alt="d63a7247c9781a984503878286959311.png" width="973" height="213">

bỏ tích 2 ô này:

**Lý do:**

- **Block private networks:** Chặn tất cả các địa chỉ IP thuộc dải nội bộ (như `10.x.x.x`, `172.16.x.x`, `192.168.x.x`) đi vào cổng WAN.
    
    - *Thực tế:* Trên Internet thật, không bao giờ có chuyện một gói tin từ dải IP nội bộ lại "lạc" vào cổng WAN của bạn, trừ khi đó là một cuộc tấn công giả mạo (Spoofing). Nên pfSense mặc định chặn để bảo mật.
- **Block bogon networks:** Chặn các IP "ma" (chưa được đăng ký hoặc không tồn tại trên bản đồ Internet thế giới).
    

Tuy nhiên để thuận tiện cho việc thử nghiệm, giả sử như các điều trên sẽ mặc định được thỏa mãn.

### **Trong phần System / Advanced / Networking**

Đảm bảo tích chọn 3 ô này:

**<img src="../../_resources/b94428a4e4ac4c87b2499a317c6fdc54.png" alt="b94428a4e4ac4c87b2499a317c6fdc54.png" width="884" height="242">**

### **Trong phần Status / System Logs / Settings**

Đảm bảo chọn `Enable Remote Logging` để pfsense tiến hành gửi **syslog** tới wazuh manager trên địa chỉ đã chọn:

**<img src="../../_resources/e8782dc50004d208b138c2f723943e2d.png" alt="e8782dc50004d208b138c2f723943e2d.png" width="858" height="575">**

# Setup rules cho vùng DMZ

`Firewall --> Rule --> OTP1`

## Setup 3 Rule cho vùng DMZ (Theo thứ tự Top-Down)

#### 🟢 Rule 1: Mở đường cho Wazuh (Nằm trên cùng)

1.  Nhấn nút **Add** (nút có mũi tên hướng lên ⬆️).
    
2.  **Action:** Pass
    
3.  **Interface:** OPT1
    
4.  **Address Family:** IPv4
    
5.  **Protocol:** TCP/UDP
    
6.  **Source:** Chọn `OPT1 subnets`
    
7.  **Destination:** Chọn `Address or alias` -> Gõ ô bên cạnh: **`192.168.10.10`** (IP Ubuntu).
    
8.  **Destination Port Range:**
    
    - Từ (From): `(other)` -> Gõ **`1514`**
        
    - Đến (To): `(other)` -> Gõ **`1515`**
        
9.  **Description:** Gõ "Allow Wazuh Agent" (để sau này dễ nhớ).
    
10. Nhấn **Save**.
    
    - <img src="../../_resources/b4cb81cb2a5b6e38e8b3c911dc152267.png" alt="b4cb81cb2a5b6e38e8b3c911dc152267.png" width="830" height="721" class="jop-noMdConv">

#### 🔴 Rule 2: Cấm DMZ vào LAN (Nằm giữa)

1.  Nhấn nút **Add** (nút có mũi tên hướng xuống ⬇️, để nó xếp dưới Rule 1).
    
2.  **Action:** **Block.**
    
3.  **Interface:** OPT1
    
4.  **Address Family:** IPv4
    
5.  **Protocol:** Any
    
6.  **Source:** Chọn `OPT1 subnets`
    
7.  **Destination:** Chọn `Network` -> Gõ ô bên cạnh: **`192.168.10.0`** / chọn **`24`** ở ô nhỏ (Hoặc nếu mục Destination có thả xuống chọn được `LAN subnets` thì chọn luôn).
    
8.  **Description:** Gõ "Block DMZ to LAN".
    
9.  Nhấn **Save**.
    
    - <img src="../../_resources/aa8a0c23f8d1bc8517204e41dc1b4073.png" alt="aa8a0c23f8d1bc8517204e41dc1b4073.png" width="832" height="622" class="jop-noMdConv">

#### 🟢 Rule 3: Cho phép máy Windows lướt Web (Nằm dưới cùng)

1.  Nhấn nút **Add** (nút có mũi tên hướng xuống ⬇️, để nó xếp dưới Rule 2).
    
2.  **Action:** Pass
    
3.  **Interface:** OPT1
    
4.  **Address Family:** IPv4
    
5.  **Protocol:** Any
    
6.  **Source:** Chọn `OPT1 subnets`
    
7.  **Destination:** Any
    
8.  **Description:** Gõ "Allow DMZ to Internet".
    
9.  Nhấn **Save**.
    
    - <img src="../../_resources/a6d8d1291a04bb146284e51cc0212cb5.png" alt="a6d8d1291a04bb146284e51cc0212cb5.png" width="818" height="611" class="jop-noMdConv">

### Kết quả

- - <img src="../../_resources/2a79977bea280cf0c7ef8e53829d738e.png" alt="2a79977bea280cf0c7ef8e53829d738e.png" width="856" height="335" class="jop-noMdConv">

# Setup Rules cho vùng LAN

- - <img src="../../_resources/9377642c8e8f76887c4440ae68ad6a66.png" alt="9377642c8e8f76887c4440ae68ad6a66.png" width="858" height="297" class="jop-noMdConv">

# Setup Rules cho vùng WAN

- - <img src="../../_resources/f034d036b1344ca4a56c831803570396.png" alt="f034d036b1344ca4a56c831803570396.png" width="877" height="231" class="jop-noMdConv"> - <img src="../../_resources/da00b38df148b8753b394bb1042a5a65.png" alt="da00b38df148b8753b394bb1042a5a65.png" width="878" height="852" class="jop-noMdConv">