**Bước 1: Tải và cài đặt Sysmon trên máy Windows**

1.  **Tải Sysmon:** Vào máy Windows 10, mở trình duyệt và tải Sysmon trực tiếp từ Microsoft Sysinternals: [https://download.sysinternals.com/files/Sysmon.zip](https://www.google.com/search?q=https://download.sysinternals.com/files/Sysmon.zip)
    
    - Giải nén (Extract All), đặt đường dẫn là **C:\\Sysmon** để dễ quản lí.
2.  Tải một file cấu hình mẫu chuẩn (giúp lọc bớt log rác), mình khuyên dùng bộ của **SwiftOnSecurity**: [sysmonconfig-export.xml](https://www.google.com/search?q=https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml).
    
    - Tải file cấu hình Sysmon từ GitHub:
    - mở **PowerShell (Admin)** trên máy Windows 10
        - ```powershell
              # Di chuyển vào thư mục C:\Sysmon vừa tạo
              cd C:\Sysmon
                      
              # Tải file cấu hình từ GitHub của SwiftOnSecurity
              Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "sysmonconfig-export.xml"
                  
            ```
            
3.  Mở PowerShell (Admin) và chạy lệnh:
    
    PowerShell
    
    ```
    .\Sysmon64.exe -i .\sysmonconfig-export.xml -accept-eula
    ```
    
    ![e1f88862249840551321d5f89d12fd26.png](../../_resources/e1f88862249840551321d5f89d12fd26.png)
    
4.  Để xem Sysmon logs:
    
    - ```
        Event Viewer --> Applications And Services Logs --> Microsoft --> Windows --> Sysmon
        ```
        
    - <img src="../../_resources/ec5ba84c1ad991804ba0a0dcdf2d15d3.png" alt="ec5ba84c1ad991804ba0a0dcdf2d15d3.png" width="1114" height="588" class="jop-noMdConv">

**Bước 2: Bảo Wazuh Agent hãy "đọc" log của Sysmon**

1.  Mở Notepad (Run as Administrator) --> `Ctrl O`.
    
2.  Mở file cấu hình của Agent: `C:\Program Files (x86)\ossec-agent\ossec.conf`.
    
3.  Thêm đoạn này vào trong mục `<ossec_config>`:
    
    - &nbsp;
        
        ```xml
        <localfile>
          <location>Microsoft-Windows-Sysmon/Operational</location>
          <log_format>eventchannel</log_format>
        </localfile>
        ```
        
    - <img src="../../_resources/7d2c0da449519a17cf20bd7ddaa25db7.png" alt="7d2c0da449519a17cf20bd7ddaa25db7.png" width="476" height="441" class="jop-noMdConv">
4.  Restart lại service Wazuh bằng lệnh trên Windows `NET STOP Wazuh` và `NET START Wazuh` hoặc `Restart-Service WahzuSvc`.
    
5.  Restart lại Wazuh Manager trên Ubuntu bằng lệnh: `sudo systemctl restart wazuh-manager`
    

&nbsp;