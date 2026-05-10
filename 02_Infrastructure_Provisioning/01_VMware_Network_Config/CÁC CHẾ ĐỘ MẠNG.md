# CÁC CHẾ ĐỘ MẠNG

| **Chế độ mạng trên VMware** | **Bản chất hoạt động (Đóng vai trò gì?)** | **Khả năng ra Internet** | **Áp dụng vào Đồ án của bạn (Vị trí cắm)** | **Đánh giá độ tối ưu** |
| --- | --- | --- | --- | --- |
| **Bridged** | Nối thẳng vào Router Wi-Fi vật lý của nhà/trường học. Máy ảo có IP ngang hàng với laptop thật. | ✅ Có (Trực tiếp) | **Không sử dụng.** (Để tránh nguy cơ mã độc lây lan ngược ra mạng thật). | ❌ Tránh dùng |
| **NAT** *(Mặc định là VMnet8)* | "Núp bóng" máy thật. Dùng IP của máy thật làm Gateway để đi ra ngoài. | ✅ Có (Gián tiếp) | 1\. Máy tấn công **Kali Linux**.  <br><br/>2\. **Cổng WAN** của Firewall pfSense. | 🌟 Rất tốt cho vùng ngoài |
| **Host-Only** *(Mặc định là VMnet1)* | Mạng nội bộ cô lập, giao tiếp được với laptop thật, nhưng bị VMware tự động nhồi sẵn dịch vụ cấp IP (DHCP). | ❌ Không | **Hạn chế dùng.** (Sẽ gây xung đột IP với tính năng DHCP mà bạn tự cấu hình trên pfSense). | ⚠️ Dễ gây lỗi xung đột |
| **Custom** *(Ví dụ: chọn VMnet2)* | Một Switch ảo trắng tinh, hoàn toàn độc lập. Bạn có toàn quyền thiết lập luật lệ và dịch vụ (như DHCP) bên trong nó. | ❌ Không (Phải đi qua pfSense) | 1\. **Cổng LAN** của Firewall pfSense.  <br><br/>2\. Máy chủ **Ubuntu (Wazuh)**.  <br><br/>3\. Máy trạm **Windows 10**. | 🏆 Tuyệt đối bắt buộc cho mạng LAN |

&nbsp;