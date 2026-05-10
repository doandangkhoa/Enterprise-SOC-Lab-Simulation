**Vị trí:** Ngay trên giao diện Web của pfSense (Windows 10).

**Cách làm:** Vào **System** ➡️ **Package Manager** ➡️ **Available Packages** ➡️ Tìm **Suricata** ➡️ **Install**.

<img src="../../_resources/5ac48f2588ad65f4dee4601a2b650778.png" alt="5ac48f2588ad65f4dee4601a2b650778.png" width="1069" height="503" class="jop-noMdConv">

**Bấm Install và chờ cho tới chi Suricata được cài đặt xog.**

**<img src="../../_resources/0db0748cc0dec42c870daba4fbca48a2.png" alt="0db0748cc0dec42c870daba4fbca48a2.png" width="1073" height="505" class="jop-noMdConv">**

**Đến đây, tạm thời chúng ta đã hoàn thành việc cấu hình Suricata. Tuy nhiên, sẽ chưa có luật nào được áp dụng cho tới khi chung ta cấu hình cho nó.**

### 🛠️ Việc cần làm để Suricata bắt đầu hoạt động:

**Bước 1: Tải "Danh sách tội phạm" (Global Rules)**

1.  Trên menu chính của pfSense, vào **Services** ➡️ **Suricata**.
    
2.  Chọn tab **Global Settings**.
    
3.  Kéo xuống mục **Install ETOpen Free Emerging Threats rules**, tích vào ô **`Enable`**. (Đây là bộ luật miễn phí cực xịn cho giới SOC).
    
4.  Bạn có thể tích thêm **`Enable Snort VRT`** nếu có code, nhưng tạm thời cứ dùng **ETOpen** là đủ phê rồi.
    
5.  Kéo xuống dưới cùng bấm **Save**.
    

**Bước 2: Cập nhật Rules (Đổ dữ liệu vào máy)**

1.  Chuyển sang tab **Updates**.
    
2.  Bấm vào nút **`Update`**. Đợi một lát để nó tải hàng nghìn luật nhận diện mã độc về máy. Khi nào dòng *Status* báo "Success" là xong.
    

**Bước 3: Chọn cổng mạng để canh gác**

1.  Chuyển sang tab **Interfaces**.
    
2.  Bấm nút **`Add`** (hình dấu cộng).
    
3.  Ở mục **Interface**, chọn **LAN** (để giám sát lưu lượng đi vào mạng nội bộ của bạn).
    
4.  Ở mục **Description**, ghi: `LAN Security Monitoring`.
    
5.  Tích chọn thêm ô `Send Alerts to System Log`
    
    - ![6fc7429f277292698b9503f0fc307f0b.png](../../_resources/6fc7429f277292698b9503f0fc307f0b.png)
    - **Log Facility (LOCAL1):** Đây là "kênh" để Suricata đẩy log vào hệ thống FreeBSD của pfSense. `LOCAL1` là giá trị tiêu chuẩn, không cần thay đổi trừ khi bạn có một hệ thống ghi log cực kỳ phức tạp.
    - **Log Priority (NOTICE):** Mức độ ưu tiên này giúp lọc các thông báo. `NOTICE` là mức vừa phải, đủ để ghi lại các sự kiện quan trọng mà không làm rác file log.
6.  Bấm **Save**.
    

<img src="../../_resources/2f414397a05f895e507177c4e434b9a5.png" alt="2f414397a05f895e507177c4e434b9a5.png" width="970" height="441" class="jop-noMdConv">

Đến đây là hoàn thành việc cài đặt Suricata.

## Testing

Máy chủ kali thực hiện : nmap -sV 192.168.10.1

Kết quả trên suricata:

<img src="../../_resources/c14d83bbc1c323d5a9d82229723ffded.png" alt="c14d83bbc1c323d5a9d82229723ffded.png" width="921" height="453" class="jop-noMdConv">

&nbsp;

# 🛠️ Cách cấu hình Linux tự động cấp quyền Promiscuous Mode khi khởi động:

Để làm điều này, chúng ta sẽ tạo một "kịch bản" nhỏ (script) để Linux tự động chạy lệnh `chmod a+rw /dev/vmnet*` mỗi khi hệ thống khởi động lên.

**Bước 1: Tạo file cấu hình (Service) mới** Gõ lệnh này để tạo và mở một file service mới tên là `vmware-promiscuous.service`: 

```
sudo nano /etc/systemd/system/vmware-promiscuous.service
```

**Bước 2: Viết "kịch bản" tự động** Khi màn hình Nano hiện ra,  copy và paste đoạn code sau vào:

```
[Unit]
Description=Enable VMware Promiscuous Mode
After=vmware.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c '/bin/chmod a+rw /dev/vmnet*'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

- Nhấn `Ctrl + O`, sau đó nhấn `Enter` để lưu file.
    
- Nhấn `Ctrl + X` để thoát khỏi Nano.
    

**Bước 3: Kích hoạt Service để nó chạy cùng hệ thống** Gõ 2 lệnh này để "nạp" kịch bản vào hệ thống và cho phép nó tự động chạy khi bật máy:

```
sudo systemctl daemon-reload
sudo systemctl enable vmware-promiscuous.service
```

*(Nếu nó báo `Created symlink...` là thành công).*

**Bước 4: Chạy thử Service ngay lúc này** Gõ lệnh này để hệ thống thực thi ngay lập tức cái script bạn vừa viết (để xem nó có lỗi gì không):

```
sudo systemctl start vmware-promiscuous.service
```

**Bước 5:** Kiểm tra

```
sudo systemctl start vmware-promiscuous.service
```