# Telecom Secure Campus Network

![Network](https://img.shields.io/badge/Network-Campus-blue)
![Firewall](https://img.shields.io/badge/Firewall-OPNsense-orange)
![IDS/IPS](https://img.shields.io/badge/IDS%2FIPS-Suricata-red)
![Monitoring](https://img.shields.io/badge/Monitoring-Zabbix-green)
![Platform](https://img.shields.io/badge/Lab-PNETLab-lightgrey)

## 📌 Giới thiệu

Dự án xây dựng mô hình **mạng Campus an toàn cho doanh nghiệp viễn thông**, được triển khai trên môi trường PNETLab.

Hệ thống sử dụng kiến trúc mạng phân cấp, kết hợp các công nghệ dự phòng và tối ưu đường truyền nhằm bảo đảm tính sẵn sàng cao. Bên cạnh đó, dự án tích hợp:

* **OPNsense** làm tường lửa và gateway biên.
* **Suricata** hoạt động ở chế độ IPS Inline để phát hiện và ngăn chặn xâm nhập.
* **Zabbix** giám sát tập trung trạng thái thiết bị, tài nguyên và sự kiện bảo mật.
* **Discord Webhook** để gửi cảnh báo sự cố theo thời gian thực.

Dự án được thực hiện với mục tiêu mô phỏng một hệ thống mạng doanh nghiệp có khả năng vận hành ổn định, giám sát tập trung và chủ động phòng chống các hành vi truy cập trái phép.

---

## 🗺️ Sơ đồ hệ thống

### Sơ đồ logic

![Logical Topology](topology/logical-topology.png)

### Sơ đồ triển khai trên PNETLab

![PNETLab Topology](topology/pnetlab-topology.png)

---

## 🏗️ Kiến trúc hệ thống

Hệ thống được thiết kế theo mô hình mạng phân cấp ba lớp:

### Core Layer

* Chuyển tiếp lưu lượng tốc độ cao.
* Kết nối giữa các thiết bị Distribution.
* Sử dụng OSPF để trao đổi thông tin định tuyến.

### Distribution Layer

* Thực hiện Inter-VLAN Routing.
* Cung cấp Default Gateway cho các VLAN.
* Triển khai HSRP để dự phòng gateway.
* Áp dụng ACL để kiểm soát truy cập giữa các vùng mạng.

### Access Layer

* Cung cấp kết nối cho thiết bị đầu cuối.
* Phân chia người dùng theo VLAN.
* Kết nối lên Distribution thông qua trunk và EtherChannel.

---

## ⚙️ Công nghệ sử dụng

| Nhóm              | Công nghệ                                |
| ----------------- | ---------------------------------------- |
| Mô phỏng          | PNETLab                                  |
| Switching         | VLAN, 802.1Q Trunking, STP, EtherChannel |
| Routing           | Inter-VLAN Routing, OSPF                 |
| High Availability | HSRP, EtherChannel                       |
| Security          | ACL, Stateful Firewall, NAT              |
| Firewall          | OPNsense                                 |
| IDS/IPS           | Suricata Inline IPS                      |
| VPN               | WireGuard                                |
| Monitoring        | Zabbix                                   |
| Data Collection   | SNMP, Zabbix Agent                       |
| Alerting          | Discord Webhook                          |
| Services          | Kea DHCP, Unbound DNS                    |

---

## 🔐 Phân vùng mạng

Hệ thống được chia thành nhiều vùng mạng riêng biệt nhằm hạn chế phạm vi ảnh hưởng khi xảy ra sự cố.

| Vùng mạng   | Vai trò                            |
| ----------- | ---------------------------------- |
| User VLAN   | Mạng dành cho người dùng nội bộ    |
| Server VLAN | Mạng dành cho các máy chủ nội bộ   |
| DMZ         | Chứa các dịch vụ công khai         |
| Management  | Quản trị thiết bị mạng và hệ thống |
| WAN         | Kết nối ra mạng bên ngoài          |
| VPN         | Truy cập từ xa thông qua WireGuard |

Việc phân vùng giúp kiểm soát lưu lượng, hạn chế truy cập trái phép và tăng khả năng giám sát hệ thống.

---

## 🛡️ OPNsense Firewall

OPNsense được triển khai tại vùng biên và đảm nhiệm các chức năng:

* Định tuyến giữa mạng nội bộ và mạng bên ngoài.
* Network Address Translation.
* Destination NAT để công bố dịch vụ trong vùng DMZ.
* Kiểm soát truy cập giữa LAN, DMZ, Management và WAN.
* Cung cấp DHCP thông qua Kea DHCP.
* Phân giải tên miền nội bộ thông qua Unbound DNS.
* Thiết lập VPN truy cập từ xa bằng WireGuard.
* Tích hợp Suricata IPS.

---

## 🚨 Suricata IPS

Suricata được cấu hình ở chế độ **Inline IPS**, cho phép hệ thống chủ động loại bỏ các gói tin phù hợp với luật phát hiện.

Các chức năng đã triển khai:

* Phân tích sâu nội dung gói tin.
* Sử dụng bộ luật Emerging Threats.
* Xây dựng custom rules.
* Phát hiện hoạt động dò quét mạng.
* Phát hiện truy cập trái phép vào vùng Management.
* Ngăn chặn kết nối SSH không hợp lệ.
* Ghi log sự kiện để gửi sang hệ thống Zabbix.

Ví dụ custom rule:

```text
drop tcp any any -> 192.168.100.0/24 22 (
    msg:"Unauthorized SSH access to Management Network";
    flow:to_server;
    sid:1000001;
    rev:1;
)
```

Các luật tùy chỉnh được lưu tại:

```text
suricata-rules/local.rules
```

---

## 📊 Zabbix Monitoring

Zabbix được sử dụng làm nền tảng giám sát tập trung cho toàn bộ hệ thống.

### Các đối tượng được giám sát

* Core Switch.
* Distribution Switch.
* Access Switch.
* OPNsense Firewall.
* Máy chủ Linux.
* Web Server trong vùng DMZ.
* Trạng thái dịch vụ Suricata.

### Các thông số giám sát

* Trạng thái thiết bị.
* Trạng thái interface.
* Lưu lượng mạng.
* Mức sử dụng CPU và RAM.
* Khả năng phản hồi ICMP.
* Log cảnh báo Suricata.
* Trạng thái các tiến trình hệ thống.

### Dashboard

![Zabbix Dashboard](zabbix/dashboard.png)

### Topology Map

![Zabbix Topology](zabbix/topology-map.png)

### Discord Alert

![Discord Alert](zabbix/discord-alert.png)

---

## 🧪 Kịch bản kiểm thử

### 1. Kiểm thử EtherChannel và OSPF

Mô phỏng sự cố một đường truyền vật lý trong Port-Channel.

**Kết quả mong đợi:**

* Port-Channel vẫn duy trì hoạt động.
* Lưu lượng được chuyển qua liên kết còn lại.
* OSPF cập nhật đường đi khi topology thay đổi.
* Zabbix phát hiện interface bị mất kết nối.

---

### 2. Kiểm thử HSRP Failover

Tắt thiết bị Distribution đang giữ vai trò Active.

**Kết quả mong đợi:**

* Distribution Switch dự phòng chuyển sang trạng thái Active.
* Virtual IP vẫn được duy trì.
* Người dùng chỉ bị gián đoạn kết nối trong thời gian ngắn.
* Zabbix gửi cảnh báo thiết bị mất phản hồi.

---

### 3. Kiểm thử Firewall và NAT

Thực hiện truy cập Web Server trong vùng DMZ thông qua địa chỉ được NAT.

**Kết quả mong đợi:**

* Người dùng bên ngoài truy cập được dịch vụ được công bố.
* Các dịch vụ không được cho phép vẫn bị chặn.
* Mạng User không thể truy cập trực tiếp vào vùng Management.

---

### 4. Kiểm thử Suricata IPS

Sử dụng Kali Linux để thực hiện dò quét và kết nối SSH trái phép.

**Kết quả mong đợi:**

* Suricata phát hiện hành vi bất thường.
* Gói tin vi phạm bị drop.
* Sự kiện được ghi vào log.
* Zabbix thu thập sự kiện và hiển thị cảnh báo.

---

## ✅ Kết quả đạt được

Dự án đã hoàn thành các mục tiêu chính:

* Xây dựng kiến trúc mạng Campus phân cấp.
* Phân chia VLAN và các vùng mạng riêng biệt.
* Triển khai EtherChannel để tăng băng thông và khả năng dự phòng.
* Triển khai HSRP để dự phòng Default Gateway.
* Triển khai OSPF để định tuyến động.
* Thiết lập ACL kiểm soát truy cập giữa các VLAN.
* Triển khai OPNsense Firewall, NAT, DHCP, DNS và VPN.
* Tích hợp Suricata IPS để phát hiện và ngăn chặn xâm nhập.
* Triển khai Zabbix giám sát tập trung qua SNMP và Agent.
* Gửi cảnh báo sự cố tự động qua Discord Webhook.
* Thực hiện các kịch bản kiểm thử sự cố và tấn công mạng.

---

## 📁 Cấu trúc repository

```text
telecom-secure-campus-network/
├── README.md
├── topology/
│   ├── logical-topology.png
│   └── pnetlab-topology.png
├── configs/
│   ├── core-sw1.txt
│   ├── core-sw2.txt
│   ├── dist-sw1.txt
│   ├── dist-sw2.txt
│   ├── access-sw1.txt
│   └── opnsense-notes.md
├── suricata-rules/
│   └── local.rules
├── zabbix/
│   ├── dashboard.png
│   ├── topology-map.png
│   └── discord-alert.png
├── test-cases/
│   ├── hsrp-failover.md
│   ├── ospf-link-failure.md
│   ├── firewall-access-control.md
│   └── suricata-ips-test.md
└── screenshots/
    ├── etherchannel-status.png
    ├── ospf-neighbor.png
    ├── hsrp-status.png
    └── suricata-alert.png
```

---

## 🔍 Một số lệnh kiểm tra

### EtherChannel

```bash
show etherchannel summary
show interfaces port-channel
```

### HSRP

```bash
show standby brief
show standby
```

### OSPF

```bash
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```

### VLAN và Trunk

```bash
show vlan brief
show interfaces trunk
```

### Kiểm tra định tuyến

```bash
show ip route
ping <destination-ip>
traceroute <destination-ip>
```

---

## 🔒 Lưu ý bảo mật

Các thông tin nhạy cảm đã được loại bỏ hoặc thay thế trước khi đưa lên repository:

* Password.
* Enable secret.
* SNMP community.
* Discord Webhook URL.
* API Token.
* WireGuard Private Key.
* Public IP thật.
* Thông tin đăng nhập hệ thống.

Ví dụ:

```text
username admin secret <REDACTED>
snmp-server community <REDACTED> RO
```

Repository này chỉ phục vụ mục đích học tập, nghiên cứu và trình bày portfolio.

---

## 🎥 Video demo

Video demo quá trình vận hành và kiểm thử hệ thống:

```text
Đang cập nhật
```

---

## 🚀 Hướng phát triển

Một số hướng có thể tiếp tục triển khai:

* Xây dựng hệ thống Firewall High Availability.
* Tích hợp SIEM để quản lý log tập trung.
* Triển khai Wazuh hoặc Elastic Stack.
* Tích hợp NetFlow để phân tích lưu lượng.
* Tự động phản ứng khi phát hiện tấn công.
* Triển khai hệ thống trên thiết bị vật lý.
* Kiểm thử tải và đánh giá hiệu năng IPS.
* Tích hợp cảnh báo qua Telegram hoặc Email.

---

## 👨‍💻 Tác giả

**Lê Hoàng Thiện**

Sinh viên chuyên ngành Truyền thông và Mạng máy tính
Trường Đại học Nha Trang

### Project

**Thiết kế và triển khai hệ thống mạng an toàn cho công ty viễn thông, tích hợp ngăn chặn xâm nhập Suricata và giám sát tập trung Zabbix.**

---

## 📄 License

Dự án được chia sẻ phục vụ mục đích học tập và nghiên cứu.

Vui lòng ghi rõ nguồn khi sử dụng nội dung hoặc cấu hình từ repository này.
