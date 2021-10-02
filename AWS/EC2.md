### EC2 là gì?
Amazon Elastic Compute Cloud (EC2) là một web service cung cấp tài nguyên hạ tầng linh hoạt trên cloud. EC2 giảm thời gian thiết lập và boot mới một server chỉ còn vài phút và cho phép bạn scale nhanh chóng tùy theo requirements.

**Note**: Mỗi account chỉ chạy được tối đa 20 instance. Muốn chạy nhiều hơn phải request Amazon
### EC2 options
##### On Demand
Cho phép bạn thanh toán một mức giá cố định (fixed rate) theo giờ (hour or second) và ko ràng buộc cam kết gì lâu dài (no commitment). Linh hoạt và chi phí rẻ (no upfront payment). Phù hợp với các ứng dụng ngắn (short-term), unpredictable workloads.

##### Reserved
Cho phép bạn trả trước capacity (pay upfront) và được offer một mức giá discount trong mỗi giờ theo 1 instance (trong 1 hoặc 3 năm) để giảm tổng chi phí. Phù hợp với những ứng dụng cần sự ổn định và chạy lâu dài. Một số options khác:

-   Standard RI: discount lớn lên tới 72% so với giá của On Demand.
-   Convertible RI: discount lên tới 54% so với On-Demand (tính linh hoạt cho việc sử dụng các instance khác nhau)
-   Scheduled RI: cho phép bạn đặt lịch theo nhu cầu mà mình đã dự đoán trc (ví dụ trong ngày Black Friday)

Khi bạn stop Reserved Instances thì nó vẫn được tính bill cho đến khi hết term

##### Spot
Cho phép bạn bid dung lượng (capacity) của instance với một mức giá bạn cho là hợp lý. Phù hợp với ứng dụng có start time và end time linh hoạt. Và nếu khi giá lên cao và AWS terminate EC2 này của bạn thì bạn sẽ ko phải trả phí cho số giờ mình sử dụng. Nếu bạn tự terminate thì bạn sẽ phải chi trả cho số giờ mà ứng dụng đã chạy.

##### Dedicated host
là physical servers cho phép bạn sử dụng các software licenses như VMWare of Oracle. Khi những license đó ko support cho việc multi-ternant hoặc cloud deployment.

### EC2 types
![[ec2-type.png]]

Bạn có thể nhớ theo câu sau: **Dr. McGift PX** tương đương:

-   **D**: Density.
-   **R**: RAM.
-   **M**: Main choice for general purpose applications.
-   **C**: Compute.
-   **G**: Graphics.
-   **I**: IOPS.
-   **F**: Field Programmable Gate Array.
-   **T**: "Tiny", cheap general purpose.
-   **P**: Graphics (think Pics)
-   **X**: Extreme memory.

### EBS là gì?
Elastic Block Store (EBS) cho phép bạn tạo storage volumes và gán nó vào một EC2 instance (giống như filesystem hoặc database). EBS được đặt ở 1 availability zone (AZ) cụ thể và có thể tự replicate.
### EBS volume types

-   **General purpose SSD (GP2)**: sử dụng SSD, phù hợp làm boot volume và database nhỏ. Max IOPS 16k/volume. Volume size 1GB -> 16TiB.
-   **Provisioned IOPS SSD (IO1)** sử dụng SSD, dành do những app I/O lớn ví dụ như SQL/NoSQL database. Max IOPS 64k/volume. Volume size 4 GiB -> 16 TiB
-   **Throughput Optimized HDD (ST1)** phù hợp với big data, data warehouse, log processing. Không thể là boot volume. Volume size 125 GiB - 16 TiB. Max IOPS 500/volume
-   **Cold HDD (SC1)** giá rẻ nhất , phù hợp với việc đọc ghi ko thường xuyên và cũng ko thể là boot volume. Volume size 125 GiB - 16 TiB
-   **Magnetic Standard** giá rẻ nhất tính theo từng gigabyte (Gb) trong các loại EBS volumes mà có thể dùng làm boot volume (bootable).

### Block Device Mapping
-   Bạn ko thể mount 1 **instance store** vào 1 instance sau khi nó đã chạy. Không phải tất cả EC2 instance đều support instance store.
-   Instance store thì **ko đảm bảo** tính toàn vẹn khi reboot, còn EBS thì đảm bảo tính toàn vẹn cả khi reboot, stop, terminate.
-   Có thể mapping với 1 EBS khi instance đang runnning
-   Chỉ có thể mapping với instance-store khi lauch instance
-   Encrypted EBS không phải được support trên toàn bộ các loại EC2 instance
-   Không thể giảm EBS volume size
- Với root volume, chỉ có thể size up
-   EBS volumes chỉ có thể attach với EC2 instance trong cùng AZ
-   Không thể mount 1 EBS volume với nhiều EC2 instances thay vì đó dùng EFS
-   Termination protection mặc định là off.

### Snapshot vs Volume

-   Volume thì nằm trong EBS (virtual hard-disk) còn snapshot thì ở trên S3
-   Snapshot của 1 volume sẽ được lưu trên S3
-   Snapshot của 1 encrypted volume cũng sẽ được tự động encrypt
-   Volume được restore từ 1 encrypted snapshot cũng sẽ được encrypt
-   Snapshot có thể share nếu nó ko encrypt
-   Volume tạo từ 1 snapshot phải có kích thước = hoặc > kích thước của snapshot
-   Muốn snapshot root volume thì cần stop instance trước khi thực hiện
### Placement Groups
Là một logical group của các EC2 instance trong cùng AZ hoặc # AZ, mục đích để giao tiếp giữa các instance là low latency, high network throughput, hay có tính chịu lỗi cao (fault-tolerant)

PG chỉ ra cách mà các instance được phân bố trên phần cứng như thế nào

Partition: Tập hợp các rack vật lý lại tạo thành 1 partition

Có 3 loại: Cluster, Partition, Spread

**Cluster**

![[cluster-pg.png]]

- AWS sẽ đưa các instance vào chạy trên cùng host/rack để có được tốc độ cao nhất có thể, low-latency và cùng 1 AZ

- Cannot span across AZ (chỉ trong AZ cụ thể)

- Băng thông lên tới 10Gbps

- Phù hợp với các ứng dụng cần HPC (High Performance Computing)

- Chỉ hỗ trợ các instance type có enhanced networking (Các instance nên chọn cùng instance type)

**Partition**

![[partition-pg.png]]

- Phân bổ các instance vào logical partition và không dùng chung với các instance khác.

- Can span across AZ

- Tối đa được 7 Partition trong 1 AZ, không giới hạn số instance trong một Partition

- Phù hợp với các ứng dụng phân tán như HDFS, Cassandra

**Spread**

![[spread-pg.png]]

- Mỗi instance sẽ được phân bổ vào 1 partition

- Can span across AZ

- Tối đa 7 instance trong 1 AZ, nếu 2 AZ thì sẽ là max 14 instance

- Phù hợp với các ứng dụng nhỏ mà yêu cầu phải chạy tách biệt nhau

## EC2 Placement Groups - Certification Tips

-   **Insufficient capacity error** can happen when:
    -   New instances are added in (OR)
    -   More than one instance type is used (OR)
    -   An instance in placement group is stopped and started
-   capacity error:
    -   Stop and start all instances in the placement group (OR)
    -   Try to launch the placement group again
    -   Result: Instances may be migrated to a rack that has capacity for all the requested instances
-   **Recommendation**:
    -   Have only one instance type in a launch request AND
    -   Launch all instances in a single launch request together

### AMI

Amazon Machine Image (AMI) là master image để khởi tạo một EC2 instance. Nó giống như một templates được cài đặt sẵn hệ điều hành (OS) và một vài phần mềm mặc định khác. AMI types được chia theo region, OS, system architecture, launch permissions và nó được lưu trữ bằng EBS hay Instance store.

-   AMI là được chia theo region, bạn chỉ có thể launch 1 AMI trong region mà nó được lưu trữ.
-   Tuy nhiên bạn có thể copy sang region khác thông qua console, commandline hoặc EC2 API

### Network & Security

#### Security Groups

Nó là một virtual firewall. Tất cả các traffic tới EC2 instance đều phải qua đây. Bạn có thể chỉ định những ports nào được mở để gửi và nhận data qua port đó. Chúng ta có thể có nhiều EC2 instance dùng chung 1 SG

Rules là **stateful** tức là nếu bạn cho phép traffic pass qua inbound rule thì nó cũng tự động được phép pass qua outbound rule. Mặc định thì inbound rule sẽ block tất cả và outbound rule thì allow cả.

#### Key Pairs

- Dùng để login vào instance thông qua SSH. AWS sẽ lưu public key còn bạn sẽ lưu private key của mình. Nếu create instance mà không có key pair --> không thể ssh/rdp vào server
- Khi bạn tạo key pair trên AWS, bạn chỉ có thể download private 1 lần duy nhất

#### Role lab

Roles sẽ bảo mật hơn so với việc lưu trữ access key và secret key trên từng EC2 instance. Role sẽ cho phép EC2 access tới AWS resources khác như S3.

-   Dễ quản lý: thay vì revoke hoặc thay key cho 100 server thì bạn chỉ cần update role cho nó
-   role có thể assign cho EC2 instance sau khi EC2 instance đã được provisioned thông qua console hoặc command line.

### States

![[Instance-lifecycle.png]]

Khi launch instance, trạng thái sẽ chuyển từ pending -> running

Khi chuyển sang trạng thái running nghĩa là instance đã bắt đầu vào quá trình boot

Sau khi instance nhận Private DNS, và có thể public DNS nếu dùng Public IP

**Stopped**

- Khi bấm stop instance, AWS sẽ thực hiện shutdown nó

- Instance-store backed instance không thể stop

- không tính tiền nữa (tính tiền EBS volume)

- Có thể detach/re-attach EBS volume(kể cả root volume)

Khi thực hiện shutdown 1 EBS-Backed EC2 instance: Private IP vẫn sẽ giữ Public IP thì sẽ mất

**Rebooting**

Việc sử dụng EC2 reboot (không dùng tiến trình reboot trong OS) có điểm lợi là: AWS sẽ thực hiện reboot mềm, và đợi 4 phút, nếu không reboot thì AWS sẽ reboot cứng và tạo log trong cloud trailtrail

**Terminated**

Khi thực hiện terminate 1 instance đang running trạng thái sẽ từ Running -> Shutting down -> Terminated

### Meta data
Hay gọi là **tags**. Bạn có thể tạo và gán nó vào EC2. Bạn có thể lấy thông tin EC2 hiện tại của mình thông qua curl ở địa chỉ sau: `curl http://169.254.169.254/latest/meta-data/` . Và lấy thông tin của user thông qua `curl http://169.254.169.254/latest/user-data/`

Để lấy địa chỉ public IP thì bạn chạy curl sau: `curl http://169.254.169.254/latest/meta-data/public-ipv4`

### User data
- thực hiện script khi instance boot
- tối đa là 16KB
- Có thể change user data khi instance stoped. Instance -> actions -> instance-setting -> View/Change user data
### Bastion Host
- Là 1 EC2 instance dùng để làm nơi truy cập tập trung, giúp bản kiểm soát các truy cập từ bên ngoài vào VPC của bạn. Từ bastion bạn có thể kết nối tiếp tới các instance bên trong. Bastion sẽ ở public subnet.

Nên gán Elastic IP cho bastion kết hợp với Security Group để tăng security cho hệ thống như giới hạn IP truy cập

HA cho Bastion có thể tạo 1 Auto Scaling group với capacity 1, min 1, max 1 để nếu basion fail hay terminate thì sẽ tự launch 1 con khác

### Elastic Network Interface (ENI)
Có thể hiểu nó là 1 card mạng ảo

Mặc định khi launch instance linux sẽ có Eth0 (là Primary network interface) và không thể xóa hay detach nó

Có thể thêm nhiều interface vào 1 EC2 instance

ENI sẽ phải nằm trong 1 AZ (tương ứng với subnet)

Public IPv4 không thể đặt trực tiếp cho eth mà bạn sử dụng Elastic IPs để map với private IP

Interface mà tạo trong quá trình launch thì khi terminate instance sẽ xóa cùng luôn

### Source/Destination check flag
Mặc định EC2 instance sẽ kiểm tra địa chỉ nguồn/đích của bất kì traffic nào đi qua. Nếu không khớp sẽ drop

Đối với Nat Instance thì cần disable tính năng này đi