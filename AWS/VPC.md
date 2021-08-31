### Khái niệm
- Cho bạn 1 môi trường ảo - private cloud - data center ảo nằm trên AWS. Tuy không hẳn bạn đc cấp 1 con Firewall, Server riêng biệt nhưng toàn bộ tài nguyên của bạn sẽ được cô lập/riêng biệt và chỉ có bạn mới có quyền connect tới môi trường đó.
- Đồng nghĩa với việc bạn sẽ có toàn quyền về các tài nguyên nằm bên trong VPC. Bạn có thể tạo 1 hay nhiều dải IP tùy bạn. Tuy nhiên, mỗi dải chỉ nằm trong 1 AZ
- Mặc định giữa các VPC sẽ không thể connect tới nhau (trừ khi 2 bên cho phép) Logically isolated from other VPCs on AWS
- Khi tạo VPC cần chỉ định region vì VPC không thể mở rộng quá khỏi region nhưng có thể nằm trên nhiều AZ.
![](https://docs.aws.amazon.com/vpc/latest/userguide/images/security-diagram.png)
### Components
-  **CIDR** viết tắt của Classless Inter-Domain Routing, là 1 IP address scheme, tức là việc phân bổ IPv4. Việc chia này làm giảm việc cạn kiệt IP v4. Ví dụ: 10.0.0.0/16
- **Router** (khi tạo subnet thì VPC sẽ khởi tạo 1 router) Chức năng là để định tuyến khá rõ rồi, giữa các subnet trong VPC, cũng như từ VPC đi ra internet.
- **Route tables** 1 bảng định tuyến bao gồm các rule gọi là "route", mỗi subnet trong VPC sẽ liên kết với 1 route table, route table sẽ quản lý vấn đề định tuyến trong subnet. 1 subnet chỉ liên kết với 1 route table tại 1 thời điểm, nhưng chiều ngược lại là 1 route table có thể chứa nhiều subnet

- **Internet gateway** để VPC kết nối ra internet hay các service khác của AWS.
- **Security Groups** virtual firewall, dùng để protect instances - Có thể hiểu ná ná như iptables apply trên network adapter của instance. Kiểm soát trafic in/out của các instance
- **Network Access Control List (N.ACLs)** Firewall dùng ở mức subnet level hay có thể hiểu là áp dụng lên đối tượng là subnet. Kiểm soát traffic in/out của các subnet
- **Virtual Private Gateway** Internet Gateway là để VPC của bạn đi ra internet, còn nếu bạn muốn từ công ty bạn connect vào các instance trên VPC chẳng hạn thì phải dùng thằng này.
- **NAT Gateway** dùng để giúp các resourcess trong private subnet có thể kết nối internet thông qua internet gateway, nhưng không cho phép giao tiếp từ internet tới các resources này
- **VPC Endpoint** cho phép 1 kết nối private tới những resource được support như S3 mà không thông qua internet gateway, VPC hay firewall proxies.

### IPv6
- Tất cả IPv6 đều public và AWS sẽ cấp khi được yêu cầu
### Router và route tables
1. **Implicit Router** Mục đích là trung tâm định tuyến trong VPC. Để connect giữa các AZ với nhau, để VPC connect tới Internet Gateway. Để inside VPC kết nối ra internet thì chỉ cần route `0.0.0.0/0` trỏ tới Internet Gateway. Nó sẽ ẩn đi mà không hiện hữu như 1 tab hay 1 device bạn có thể create, delete
2. **Route table** 
- Có thể lên tới 200 route tables trong 1 VPC
- Có thể có 50 routes entries trong một route table
- Mỗi subnet cần liên kết với 1 route table và chỉ 1 route table tại 1 thời điểm
- Nếu không chỉ định route tables khi tạo subnet, thì subnet sẽ liên kết với main/default VPC route table
- Có thể thay đổi subnet liên kết với route table khác
- Có thể sửa route của main route table nhưng không để xóa
- Có thể chuyển custom route table lên thành main (chỉ có 1 main tại 1 thời điểm). Lúc này thì thằng main cũ sẽ là custom và xóa ngon lành
- Mỗi 1 route table sẽ có 1 default route, và không thể edit được. Ví dụ khi tạo subnet 10.0.1.0/16 thì sẽ có route cho thằng này là đi trong local
- Không cho phép 2 CIDR block cùng range hoặc lớn hơn tồn tại trong 2 route table. Ví dụ là có 10.0.0.0/24 trỏ về VPG, thì chỉ có thể thêm route nhỏ hơn /24 mới đc phép tồn tại trong route.

### IP Address
- Khi bạn tạo VPC xong thì bạn không thể thay đổi CIDR range. Tức là khi tạo ra thì phải chọn dải IP mà đã chọn rồi thì ko sửa được nữa.
- Subnet nằm trong dải /16 đến /28
- Nếu cần tạo 1 CIDR khác thì tạo mới VPC
- Các subnet trong VPC không được overlap lẫn nhau (ví dụ đã tạo dải 10.0.0.0/24 thì ko được tạo 10.0.0.0.0/28)
- Có thể extent dải IP bằng việc thêm 1 CIDR nữa
- Reserved IP's in each subnet. Bạn không thể sử dụng 4 địa chỉ IP đầu tiên và địa chỉ ip cuối cùng trong mỗi subnet. Ví dụ 1 subnet 10.0.0.0/24 thì bạn không thể sử dụng, gán cho instance:
`10.0.0.0`: Network address
`10.0.0.1`: AWS dùng cho VPC router
`10.0.0.2`: reserved bởi AWS. phần này liên quan tới DNS server [AWS DNS](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html#AmazonDNS)
`10.0.0.3`: được AWS reserved để dùng trong mục đích sau này
`10.0.0.255`: Network broadcast
### Internet Gateway
- Khi tạo VPC, mặc định sẽ có 1 Internet GW (và chỉ 1 mà thôi, nhưng có thể tạo được 5 Internet GW trong 1 region)
- Là một thành phần để hoziontally scale, redundant, and HA
- Sử dụng cơ chế NAT (1:1) giữa hệ thống private và public. Từ trong ra sẽ được IG Nat đi ra ngoài, và không có chuyện IP public gắn thẳng lên instance được mà phải đi qua IG
- Support IPv4 và IPv6
### Subnet
- Public: Nếu subnet mà có route trỏ về internet gateway thì ta gọi subnet đó là subnet public. Instance trong dải này muốn từ internet truy cập thì có thể dùng thêm Elastic IP.
- Private: Nếu subnet mà không có route trỏ về internet gateway thì ta gọi đó là subnet private. Instance trong dải này nếu muốn ra internet bạn có thể sử dụng Nat instance/gateway ...
### VPC Types
**Default VPC**
- Được tạo tự động ở mỗi region khi tạo tài khoản AWS
- Mặc định có CIDR , SG, N ACL, and route table sẵn luôn
- Mặc định có IG sẵn luôn

**Custom VPC**
- Là VPC do customer tạo
- Tự tạo CIDR
- Mặc định có SG, NACL, Route tables sẵn luôn
- IG cần thì tạo

### Security Groups
- Là 1 virtual firewall
- Kiểm soát traffic ở lớp virtual server (EC2 instance)
- Có thể có 5 SG cho 1 Instance
- Là dạng **Stateful**, nghĩa là bạn allow cho chiều vào (in) thì mặc định lúc return (out) sẽ là cho phép. Ví dụ bạn muốn mở SSH từ ngoài vào instance. Bạn chỉ cần tạo 1 rule là cho port 22 đi vào instance là được
- Chỉ có permit rule, không có deny rule
- Rule mặc định là deny và nằm dưới cùng
- Khi tạo EC2 thì sẽ phải gán luôn với ít nhất 1 SG nào đó (Chưa có thì sẽ tạo mới)
- Hiệu lực tức thời khi change rule

### Network Access Control Lists (NACLs)
- Có thể coi là 1 funtion của Implied Router
- Là funtions ở subnet level (kiểm soát traffic in/out của subnet)
- Là dạng firewall **stateless**. Tức là nếu bạn muốn cho phép 1 giao thức in/out đều được thì bạn phải cấu hình 2 rule, 1 rule inbound + 1 rule outbound
- Có thể chọn "permit" hoặc "deny"
- NACL là 1 tập hợp các rule và mỗi rule sẽ có 1 number
- Có 1 rule dưới cùng là deny all, không thể xóa rule này
- NACL bao gồm nhiều rule và được xét theo thứ tự, bắt đầu với rule thấp nhất. Thêm nữa là khi rule được match allow/deny thì sẽ được thực hiện luôn. Có nghĩa là khi bạn có cả rule allow và deny với 1 request traffic thì rule nào với số nhỏ hơn thì sẽ được thực hiện và rule còn lại sẽ bị bơ (ignore)
- Mỗi subnet cần associated với 1 NACL, nếu không chỉ định nó sẽ liên kết với NACL default (NACL khi tạo VPC)

| Security group | Network ACL |
| -------- | -------- |
| hoạt động ở instance level  | hoạt động ở subnet level   |
| chỉ cho phép allow rules | cho phép cả allow rules và deny rules |
| stateful: traffic back được tự động cho phép bất kể rules nào | stateless: chỉ định rõ những traffic được allow |
| xét tất cả các rules trước khi quyết định rule nào đc cho phép | xét các rule theo thứ tự, bắt đầu từ rule có thứ tự nhỏ nhất và quyết định có allow traffic hay ko |
| Áp dụng với instance chỉ định | Tự động áp dụng với tất cả các instance trong cùng 1 subnet |

### NAT Instance
Ngoài cách trỏ Instance trong private subnet về Internet Gateway còn có một cách khác dùng con máy ảo Nat Instance.
- Nat instance sẽ nằm ở vùng public subnet. Mục đích là cho instance dải Private đi ra Internet.
- Từ internet sẽ không truy cập dc vào Private Subnet
- Cần assign với 1 SG
- Sử dụng OS được AWS build sẵn
- Nên siết policy chỉ cho phép 1 số IP ssh vào server từ internet
- Siết rule liên quan http và https (in/outbound to 0.0.0.0/0 (Internet) on ports 80 and 443)
- Cần disable source/destination check vì EC2 instance mặc định sẽ check  
Source và Destination khi gửi/nhận dữ liệu. Trong trường hợp làm NAT này, nó sẽ  thấy thằng Source/Destination không giống nhau.
![](https://miro.medium.com/max/652/1*s9i-UTOjPuH9gDlxabiE2Q.png)
### NAT Gateway
Một cách nữa để các Instance trong private subnet ra internet là trỏ về NAT Gateway
- Sẽ do AWS manage, nên không phải quan tâm về việc update patch ...
- Mục đích là cho instance dải Private đi ra Internet
- Từ internet sẽ không truy cập dc vào Private Subnet
- Không thể assign với 1 SG
- Chỉ làm việc với Elastic IP

Price:
- Tính tiền dựa vào số lượng data transfer qua GW: 0.062$/GB
- Và số giờ mà NAT GW avaiable: 0.062$/h
- Không mất tiền khi từ EC2 -> S3 ( Cùng region)
- Không mất tiền từ Nat GW -> EC2 cùng Avaiable Zone cùng sử dụng private IP
![](https://miro.medium.com/max/330/1*zf-v6VkNi9_PH6uuotmzrg.png)
### VPC Wizard
Hiểu đơn giản là Amazon sẽ đưa ra các recommend (template) cho bạn khi tạo VPC
- VPC with a Single Public Subnet
- VPC with Public and Private Subnet
- VPC with Public and Private Subnet and Hardware VPN Access
- VPC with a Single Public Subnet Only and hardware VPN Access
### VPC Peering
VPC tạo ra với mục đích cô lập toàn bộ tài nguyên trong đó ...
Giả sử như bạn có nhiều VPC, 1 VPC cho việc development + 1 VPC cho việc chạy production.

Giờ có 1 file cần share giữa 2 con instance ở trên 2 VPC khác nhau, mà các instance này chỉ dùng private subnet ==> sử dụng VPC peering

- Peering chỉ cho phép kết nối 1:1 (trong một kết nối peering chỉ cho phép 2 VPC, đồng thời giữa 2 VPC chỉ cho tạo 1 kết nối peering)
- Nếu CIDR blocks giữa 2 VPC bị overlap ==> ko thể peering
- Không cần lo về bottleneck hay SPOF
- Không thể dùng VPC peering để transit hay edge routing

![VPCs with matching IPv4 CIDR blocks](https://docs.aws.amazon.com/vpc/latest/peering/images/overlapping-cidrs-ipv6-diagram.png)
![Transitive peering](https://docs.aws.amazon.com/vpc/latest/peering/images/transitive-peering-diagram.png)
![Edge to edge routing through an internet gateway](https://docs.aws.amazon.com/vpc/latest/peering/images/edge-to-edge-igw-diagram.png)
![Edge to edge routing through a VPC endpoint](https://docs.aws.amazon.com/vpc/latest/peering/images/edge-to-edge-s3-diagram.png)

Refs: https://docs.aws.amazon.com/vpc/latest/peering/invalid-peering-configurations.html

### AWS transit gateway
Vẫn là bài toàn muốn giao tiếp, chia sẻ dữ liệu giữa các VPC, VPC với on-premise.

Nếu số lượng các VPC lên nhiều thì để các VPC có thể share được với nhau (full mesh) thì số peering sẽ rất lớn.

Để giải quyết vấn đề này, AWS có 1 dịch vụ là Transit gateway cho phép kết nối giữa các VPC với nhau, với On-primise chỉ với 1 gateway
### AWS Virtual Private Gateway
Là 1 dịch vụ cung cấp dịch vụ Virtual Private Network cho phép khách hàng tạo 1 tunnel từ site của họ với VPC

Hỗ trợ giao thức IPSec
### VPC Endpoint
![New – VPC Endpoints for DynamoDB | AWS News Blog](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2017/08/15/ddb-no-vpc-endpoint-1024x561.png)

- Như hình trên, ta thấy để EC2 giao tiếp với DynamoDB, traffic sẽ đi ra IG rồi ra ngoài internet, tiếp đó là đi tới DB.
=> Dữ liệu đi vòng vèo, latency cao, không bảo mật

Có 1 giải pháp là sử dụng VPC Endpoint (PrivateLink). Giải pháp này sẽ giúp traffic giữa VPC và các dịch vụ khác không phải đi vòng ra internet.

Instance không cần có IP public, không đi qua IG ...
![](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2017/08/15/ddb-yes-vpc-endpoint-1024x565.png)

Refs: https://aws.amazon.com/vi/blogs/aws/new-vpc-endpoints-for-dynamodb/

- Các Endpoint là virtual device. Cho phép scale, redundant, có tính high available cho phép kết nối giữa các instance trong VPC và các dịch vụ khác một kết nối an toàn và băng thông cao.
- Có 2 kiểu VPC Endpoint là interface endpoints và gateway endpoints. Khi tạo cần check xem service có support loại Endpoint này hay không? (Refs: https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)

**Interface endpoint**
Là 1 điểm gồm 1 IP private và 1 Elastic IP, đóng vai trò như 1 điểm chung chuyển lưu lượng tới các dịch vụ khác
![Using an interface endpoint to access Kinesis](https://docs.aws.amazon.com/vpc/latest/privatelink/images/vpc-endpoint-kinesis-private-dns-diagram.png)

**Gateway endpoint**
- chỉ cần trỏ route trong route table về gateway
- Có thể policy cho gateway để kiểm soát việc truyền dữ liệu
- Hỗ trợ trong cùng region. Ví dụ như S3 ở region khác thì Endpoint không thể hỗ trợ giao tiếp được
![Using a gateway endpoint to access Amazon S3](https://docs.aws.amazon.com/vpc/latest/privatelink/images/vpc-endpoint-s3-diagram.png)

Price
- Tính tiền theo giờ và lưu lượng transfer $0.014/h và $0.01/GB
### VPC Flow Logs
Là 1 tính năng cho phép capture và log data về network traffic trong VPC của bạn
- Troubleshoot các vấn đề như sao ko thể truy cập instance (giúp kiểm tra lại SG xem chuẩn chưa)
- Sử dụng như một security tool để giám sát traffic truy cập tới instance
- Có thể create Flow log cho 1 VPC hoặc 1 subnet hoặc 1 network interface
- Nếu tạo flow log ở mức subnet, thì mỗi subnet và network interface sẽ bị monitor
- Mỗi network interface sẽ có 1 log file
- Flow log có thể push data sang cho CloudWatch Logs và S3