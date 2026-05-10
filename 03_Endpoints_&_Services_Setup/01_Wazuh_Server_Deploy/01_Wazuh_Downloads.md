Thực hiện tải Wazuh Manager trên máy chủ Ubuntu Server

`curl -o wazuh-install.sh https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash wazuh-install.sh -a`

![2363698ee35a05fa7ef03a77c7ff726c.png](../../_resources/2363698ee35a05fa7ef03a77c7ff726c.png)

![9273545f21a7729aa3512148643d2e68.png](../../_resources/9273545f21a7729aa3512148643d2e68.png)truy cập vào trang chủ wazuh và bắt đầu cấu hình

## Thay đổi pass của Admin

login root:

`sudo su`

`cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/`

Thay đổi mặt khẩu của tài khoản admin:

`./wazuh-passwords-tool.sh -u admin -p yourpassword`