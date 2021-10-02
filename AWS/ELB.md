### Tổng quan
- Là 1 dịch vụ cho phép tự động phân phối lưu lượng truy cập (incoming traffic) đến nhiều targets
- có 3 loại: Application Load Balancers, Network Load Balancers, và Classic Load Balancers

### Application Load Balancer

![[ALB.png]]

- cân bằng tải trafifc HTTP và HTTPS (Layer 7 load balancer)
- ALB route traffic tới resources bên trong VPC dựa trên của request: URL, Host name, Query params
- Có thể lắng nghe nhiều port và route traffic tới 1 instance

### Network Load Balancer

![[NLB.png]]

- cân bằng tải các giao thức TCP, UDP, TLS (Layer 4)
- route traffic tới các thành phần bên trong VPC dựa trên giao thức và port
- Cross-zone Load Balancing

### Classic Load Balancer
- Hỗ trợ cả request level hoặc connection level (Layer 4 hoặc Layer 7)
- Cross-zone Load Balancing

### Traffic rules
- LB sẽ tiếp nhận incoming traffic và điều phối request tới registered targets trên 1 hoặc nhiều AZ

- LB sẽ giám sát trạng thái của mỗi target để đảm bảo traffic được điều hướng tới healthy target

- Nếu xác định có target bị lỗi, LB sẽ không đẩy traffic về target lỗi

- cơ chế heath check để kiểm tra trạng thái mỗi target

- ALB yêu cầu instance có ở trên tối thiểu 2 AZ (1 subnet/AZ)

- ELB có thể public (internet facing) để nhận request từ ngoài internet vào, hoặc chỉ dùng để chạy trong internal

### Cross-Zone Load Balancing
- ELB sẽ tạo ở mỗi AZ một load balancer node

- Mỗi Node sẽ nhận tương ứng 1/N số traffic. N là số node

- Khi enable CZ lên thì sẽ cho phép Node route traffic tới các instance ở AZ khác

- ALB luôn luôn enable CZ

- NLB và CLB thì có thể tùy chọn

**Interface facing**
- ELB node sẽ có Public IP
- Backend instance chỉ cần IP Private
- ELB Node ở public subnet

**Internal ELB**
- ELB node sẽ có IP Private
- Backend instance chỉ cần IP Private

**Note**:

- ELB sẽ route traffic tới primary network card của instance (eth0)
- Nếu interface có nhiều IP thì sẽ route tới Private address
- ELB cũng sẽ cần cấu hình Security Group và NACL
- Tất cả instances unhealthy thì sẽ đẩy request tới tất cả

