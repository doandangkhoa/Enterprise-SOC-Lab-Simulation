### Kịch bản 1: "Máy chém" Tự Động (Endpoint Malware Eradication)

**Bối cảnh:** Kẻ tấn công bằng cách nào đó đã lọt qua được tường lửa, tải thành công một Payload (mã độc) xuống máy Windows 10 trong mạng LAN và giấu vào thư mục `C:\Users\Public`.

- **Tín hiệu Kích hoạt (Trigger):**
    
    - **Cảm biến:** File Integrity Monitoring (FIM / Syscheck) trên Windows 10.
        
    - **Điều kiện:** FIM phát hiện file mới, trích xuất Hash và gửi lên Wazuh Manager. Tích hợp VirusTotal trả về kết quả quét dương tính (Nổ Rule Level 12).
        
- **Hành động Phản hồi (Action):**
    
    - Wazuh Manager kích hoạt script cấu hình sẵn (ví dụ: `ossec-rm.cmd` hoặc `remove-threat.exe`) đẩy trực tiếp lệnh xuống Wazuh Agent trên máy Windows.
        
    - *Lệnh thực thi ngầm:* Xóa bỏ vĩnh viễn file vật lý tại đường dẫn vừa được FIM báo cáo.
        
- **Kết quả đạt được:** Payload độc hại bị bốc hơi khỏi ổ cứng chỉ trong vòng 10-15 giây sau khi chạm vào đĩa, hoàn toàn tự động, chặn đứng nguy cơ mã độc được thực thi.
    
- **Điểm nhấn kỹ thuật:** Chứng minh khả năng tích hợp Cyber Threat Intelligence (VirusTotal) để đưa ra quyết định xử lý tự động với độ chính xác 100%, không gây False Positive xóa nhầm file hệ thống.
    

* * *

### Kịch bản 2: Cách ly Y tế Cục bộ (Lateral Movement Quarantine)

**Bối cảnh:** Hacker đã lừa được người dùng chạy mã độc. Thay vì phá hoại ngay, chúng sử dụng các công cụ ẩn mình (như lấy cắp mật khẩu bằng `procdump` từ LSASS) để chuẩn bị leo thang đặc quyền và lây lan sang máy tính khác, hoặc nhảy sang tấn công Web Server đang chạy Spring Boot trong vùng DMZ.

- **Tín hiệu Kích hoạt (Trigger):**
    
    - **Cảm biến:** Sysmon trên Windows 10.
        
    - **Điều kiện:** Sysmon bắt được Event ID 1 và Wazuh Manager nổ Rule 100302 (Phát hiện công cụ kết xuất thông tin xác thực) hoặc phát hiện PowerShell thực thi lệnh bất thường.
        
- **Hành động Phản hồi (Action):**
    
    - Wazuh Manager lập tức gọi script `route-null` hoặc một đoạn script PowerShell thao tác với Windows Defender Firewall trên máy Victim.
        
    - *Lệnh thực thi ngầm:* Thêm các rule chặn toàn bộ Inbound/Outbound traffic ở Layer 3, **NGOẠI TRỪ** cổng 1514/1515 UDP/TCP (cổng giao tiếp với Wazuh Manager).
        
- **Kết quả đạt được:** Máy Windows 10 bị cô lập hoàn toàn khỏi mạng LAN và DMZ. Hacker mất kết nối điều khiển từ xa (C2/RDP), nhưng đội ngũ SOC vẫn giữ được kết nối với Wazuh Agent để thu thập bằng chứng pháp y (Forensics) trên RAM.
    
- **Điểm nhấn kỹ thuật:** Thể hiện tư duy Ứng phó sự cố (Incident Response) ở bậc cao. Không ngắt nguồn thiết bị (tránh mất dữ liệu bay hơi) mà dùng cơ chế Network Isolation để bảo vệ hệ thống nền tảng.
    

* * *

### Kịch bản 3: Lá chắn Tiên phong Layer 7 -> Layer 3 (Cross-Layer Defense)

**Bối cảnh:** Một IP lạ từ Internet liên tục thực hiện rà quét lỗ hổng hoặc ném các payload SQL Injection/XSS vào Web Server nằm trong vùng DMZ. pfSense mặc định phải mở Port 80/443 nên traffic này đi qua trót lọt và tới thẳng máy chủ Web.

- **Tín hiệu Kích hoạt (Trigger):**
    
    - **Cảm biến:** Suricata (NIDS) giám sát lưu lượng trong DMZ.
        
    - **Điều kiện:** Suricata đọc được nội dung gói tin HTTP (Layer 7), nhận diện chuỗi tấn công Web, tạo log cảnh báo và đẩy về Wazuh Manager qua Rsyslog.
        
- **Hành động Phản hồi (Action):**
    
    - Wazuh Manager phân tích log từ Suricata. Nếu một IP công cộng vi phạm rule tấn công Web với tần suất cao (ví dụ: 5 lần trong 1 phút), Manager sẽ kích hoạt một script tùy chỉnh (gọi qua API hoặc SSH).
        
    - *Lệnh thực thi ngầm:* Wazuh kết nối ngược lên pfSense, đẩy IP Public của kẻ tấn công vào một Alias (ví dụ: `Wazuh_Auto_Block`), được gắn với rule DROP trên giao diện WAN của firewall.
        
- **Kết quả đạt được:** Kẻ tấn công lập tức bị ngắt kết nối từ vòng gửi xe (Layer 3). Web Server trong DMZ được an toàn, không còn phải gánh chịu tải từ các request tấn công.
    
- **Điểm nhấn kỹ thuật:** Đây là mảnh ghép SOAR hoàn hảo. Sự phối hợp đa chiều: Phân tích sâu ở tầng Ứng dụng (Suricata) -> Điều phối tập trung (Wazuh) -> Chặn đứng ở ranh giới Mạng (pfSense).