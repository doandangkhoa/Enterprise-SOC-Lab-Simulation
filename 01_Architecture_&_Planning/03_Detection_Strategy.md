Chiến lược giám sát được xây dựng theo mô hình Phòng thủ chiều sâu (Defense-in-Depth), kết hợp cả **Giám sát mạng (Network-based)** và **Giám sát máy trạm (Host-based)**.

**1\. Data Ingestion Pipeline (Luồng thu thập dữ liệu):**

- **Network-Level (NIDS):** Suricata bắt các chữ ký mạng độc hại (Malicious Signatures) tại pfSense. Log cảnh báo (`eve.json`) được đẩy về Wazuh Server thông qua giao thức **Syslog** (Port `514/UDP`).
    
- **Host-Level (HIDS):** Wazuh Agent giám sát tính toàn vẹn của tệp (FIM), các lệnh thực thi lạ và đọc log từ **Sysmon** (System Monitor) để bắt các hành vi như leo thang đặc quyền, tạo tiến trình con bất thường trên máy Windows.
    

**2\. SIEM Correlation & Alerting (Phân tích tương quan trên Wazuh):**

- Thay vì để cảnh báo rác làm tràn ngập màn hình (Alert Fatigue), hệ thống sử dụng các bộ luật XML tùy biến.
    
- **Ví dụ:** Nếu Suricata phát hiện nhiều lỗi HTTP 404 (Reconnaissance) và Sysmon phát hiện mật khẩu đăng nhập sai liên tục trên Windows, Wazuh sẽ tổng hợp tần suất (`frequency`) và thời gian (`timeframe`) để đưa ra một cảnh báo Critical duy nhất (Level 12) ánh xạ theo **MITRE ATT&CK** (ví dụ: *T1110 - Brute Force*).
    

**3\. Threat Intelligence Enrichment (Làm giàu dữ liệu tự động):**

- Wazuh Server được tích hợp module API gọi trực tiếp đến **VirusTotal**.
    
- Khi Sysmon ghi nhận một file thực thi mới (mã Hash) hoặc một kết nối đến IP lạ, Wazuh sẽ lấy Hash/IP đó gửi qua API Request đến VirusTotal. Nếu kết quả trả về là Malicious, hệ thống sẽ tự động nâng mức cảnh báo, giúp Analyst tiết kiệm thời gian điều tra thủ công.