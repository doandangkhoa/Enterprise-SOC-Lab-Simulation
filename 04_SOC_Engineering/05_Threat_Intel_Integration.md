# Tích hợp VirusTotal (Threat Intelligence) vào Manager

Chúng ta sẽ cho Wazuh Manager tự động bắt các file `.exe` khả nghi (thông qua mã Hash do Sysmon gửi về) và đem lên VirusTotal kiểm tra.

**1\. Lấy API Key của VirusTotal (Miễn phí):**

- Truy cập trang chủ [VirusTotal](https://www.virustotal.com/) và tạo một tài khoản (nếu chưa có).
    
- Đăng nhập, bấm vào Avatar góc trên bên phải -> Chọn **API Key**.
    
- Copy đoạn mã API Key (một chuỗi ký tự rất dài).
    

**2\. Cấu hình trên Wazuh Manager:** Mở file cấu hình lõi của Wazuh trên máy Ubuntu:

```
nano /var/ossec/etc/ossec.conf
```

*Sử dụng phím mũi tên kéo xuống dưới cùng (hoặc tìm đến gần thẻ `</ossec_config>`), bạn hãy dán khối cấu hình này vào và thay thế `YOUR_API_KEY` bằng mã bạn vừa copy:*

```
  <integration>
    <name>virustotal</name>
    <api_key>YOUR_API_KEY_Ở_ĐÂY</api_key>
    <group>syscheck</group>
    <alert_format>json</alert_format>
  </integration>
```

Lưu file lại (`Ctrl+O`, `Enter`, `Ctrl+X`).

**3\. Khởi động lại hệ thống** Chạy lệnh này để Wazuh đọc cấu hình mới:

```
systemctl restart wazuh-manager
```

```
systemctl status wazuh-manager
```

**4\. Bật tính năng FIM (Trên Windows 10)**

1.  Bạn vào đường dẫn `C:\Program Files (x86)\ossec-agent\` và mở file `ossec.conf` (nhớ mở Notepad bằng quyền Admin).
    
2.  Tìm đến phần `<syscheck>`. Bạn thêm một dòng giám sát thư mục `Public` vào đó, yêu cầu nó phải tính toán mã Hash theo thời gian thực:
    

```
<directories check_all="yes" realtime="yes">C:\Users\Public</directories>
```

3.  Lưu file lại và **Restart** dịch vụ Wazuh Agent bằng CMD (quyền Admin):

```cmd
net stop WazuhSvc && net start WazuhSvc
```