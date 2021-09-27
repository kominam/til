### Definition
-   Mục đích là phân giải tên miền (human-readable website) thành IPv4 hoặc IPv6
-   Hoạt động trên cả 2 giao thức TCP và UDP và **port 53**
### Abbreviation
| Abbr | Full sentence |
| -------- | -------- |
| DNS | Domain Name System |
| ISP | Internet Service Provider |
| TLD | Top Level Domain |
| ICANN | Internet Corporation for Assigned Names and Numbers |
### How does it work
- **Step 1**: Vào trình duyệt gõ www.example.com
- **Step 2**: Query cache của trình duyệt. ví dụ: `chrome://net-internals/#dns`. Nếu có thì trả về IP của domain và thực hiện request, còn không thì sẽ query ở OS. OS sẽ check ở `/etc/hosts` file để tìm IP (có thể thay đổi trong file `/etc/nsswitch`), sau đó sẽ check ở ` /etc/resolv.conf` để tìm DNS server IP. Nếu không tìm thấy thì OS sẽ gọi đến resolver (The Resolver on ISP).
- **Step 3**: OS sẽ gửi request thông qua giao thức UDP và port 53 đến resolver. DNS server sẽ query trong database, nếu không có kết quả thì nó sẽ gọi đến root server.
- **Step 4** Root server (https://www.iana.org/domains/root/servers)
Chúng ta có **13 root servers** trên toàn thế giới (được đặt tên từ a - m eg: a.root-servers.net). Chúng không biết IP của domain đó là gì nhưng chúng biết có thể tìm ở đâu. Root server sẽ đọc thành phần đầu tiên từ phải sang trái, ở đây là `.com`. Gửi request (direct request) tới TLD name server của `.com`. ISP sẽ lưu trữ lại thông tin của TLD.
- **Step 5**:  Mỗi TLDs sẽ thuộc về ICANN tương ứng. TLD name server sẽ tìm **Authoritative nameservers** cho domain được request `example.com`. Authoritative nameservers biết mọi thông tin về một domain nào đó (nó được lưu ở DNS records).
- **Step 6**: Resolver nhận A record từ Authoritative nameservers và lưu lại ở local. Resolver sẽ trả lại A record (IP) cho OS
![DNS Heirarchy](https://www.cloudflare.com/img/learning/dns/glossary/dns-root-server/dns-root-server.png)
### References
- https://s905060.gitbooks.io/site-reliability-engineer-handbook/content/how_does_dns_server_work.html
- https://howdns.works/