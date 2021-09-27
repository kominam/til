![](https://s905060.gitbooks.io/site-reliability-engineer-handbook/content/linux-boot-process.png)

### BIOS
BIOS là viết tắt của **Basic Input/Output System**. 
Nó thực hiện tích hợp và kiểm tra một vài chức năng cơ bản như search, load và thực thi boot loader các chương trình.  Nó sẽ tìm các boot loader ở trong ổ đĩa mềm (floppy), CD-ROM hoặc ổ cứng. Hiểu đơn giản thì khi bật máy BIOS sẽ wake up và kiểm tra các driver và thiết bị ngoại vi, các trật tự ổ cứng (MBR boot loader) rồi chuyển việc kiểm soát còn lại cho OS. Bạn có thể ấn nút để có thể vào trình quản lý BIOS (tùy thuộc vào loại máy và OS, thường là F2 hoặc F12).
### MBR
MBR là viết tắt của **Master BOOT Record**.
Nó nằm ở đầu của một ổ đĩa chứa boot. Thường là trong `/dev/hda` hoặc `/dev/sda` MBR sẽ nhỏ hơn 512 bytes. Nó sẽ có 3 components:
- Primary boot loader: chứa thông tin của 446 bytes đầu tiên
- Partition table: chứa thông tin 64 bytes tiếp theo (16 bytes với 4 entries)
- Boot signature: MBR validation check 2 bytes cuối

![MBR construction](https://www.partitionwizard.com/images/uploads/articles/2019/07/windows-8-mbr-fix/windows-8-mbr-fix-1.png)

Nó sẽ chứa thông tin của GRUB (hoặc LILO ở những hệ thống cũ).
Hiểu đơn giản thì MBR sẽ load và khởi động GRUB boot loader.
### GRUB
GRUB viết tắt của **Grand Unified Bootloader**.
Nếu bạn cài đặt nhiều kernel images thì bạn có thể chọn được kernel nào sẽ được thực thi.
GRUB sẽ hiển thị màn hình splash trong vài giây và nếu bạn không chọn gì thì nó sẽ load default kernel được chỉ định trong GRUB config file.
GRUB config file nằm ở `/boot/grub/grub.cfg` (`/etc/default/grub` link tới file này). Nên hiểu đơn giản thì GRUB sẽ load và thực thi kernel và initrd images
### Kernel
Mount root file được chỉ định trong `root=` ở file `grub.cfg`, kenel sẽ thực thi `/sbin/init`, bởi vì nó là chương trình được khởi chạy đầu tiên nên sẽ có PID là 1. Bạn có thể kiểm tra bằng command: `ps -ef | grep init`

Initrd là viết tắt của **Initial RAM Disk**. Nó được sử dụng như là root file (temporary) cho đến khi kernel được boot xong và root file được mount. Nó chứa driver được compile để access các phần vùng ổ cứng và các phần cứng khác.
### Init
##### Unix System V
Với những version hệ điều hành trước đây, bạn có thể xem Linux runlevel ở file `/etc/inittab`.  Init sẽ dựa vào đó để load tất cả những chương trình theo script tương ứng với runlevel (gồm runlevel từ 0 - 6).  Bạn có thể thay đổi default run level là 3 (không có GUI) hoặc 5 (có GUI).

Các run level:
-   Runlevel 0 - /etc/rc.d/rc0.d : Level Shutdown hệ thống (`shutdown -h`).
-   Runlevel 1 - /etc/rc.d/rc1.d: Level chỉ dùng cho 1 người dùng để sửa lỗi hệ thống tập tin.
-   Runlevel 2 - /etc/rc.d/rc2.d: Không sử dụng.
-   Runlevel 3 - /etc/rc.d/rc3.d: Level dùng cho nhiều người dùng nhưng không có giao diện.
-   Runlevel 4 - /etc/rc.d/rc4.d: Không sử dụng.
-   Runlevel 5 - /etc/rc.d/rc5.d: Level dùng cho nhiều người dùng và được cung cấp giao diện đồ họa.
-   Runlevel 6: Level Reboot hệ thống (`reboot`).

**Notes:**
- Có symlink `/etc/rc0.d` tới `/etc/rc.d/rc0.d`.
- Trong folder `/etc/rc.d/rc*.d/` script bắt đầu bằng `S` là những chương trình sẽ chạy khi startup (S = startup). `K` là những chương trình chạy khi shutdown (K = kill).
##### Systemd
Đối với các bản Linux gần đây thì init và runlevel được thay thế bởi systemd và cũng thực hiện nhiệm vụ tương ứng. Systemd cũng giống như init là tiến trình chạy đầu tiên trên hệ thống với ID = 1.

Systemd sẽ đọc `/etc/systemd/system/default.target` để xác định default target. Nó sẽ bắt đầu với `/etc/systemd/system/basic.target` trước khi chạy multi-user service.

Target trong systemd tương ứng với Run level ở Unix System V. Trong đó có 2 target chính là: `multi-user.target` và `graphical.target` lần lượt ứng với run level 3 và 5.

| Runlevel | Target |
| -------- | -------- |
| 0  | poweroff.target   |
| 1  | rescue.target   |
| 2, 3, 4  | multi-user.target   |
| 5  | graphical.target   |
| 6  | reboot.target   |

Để xem default target:
``` bash
systemctl get-default # graphical.target
```
Thay đổi default target:
``` bash
systemctl enable <target>.target
systemctl set-default <target>.target
```