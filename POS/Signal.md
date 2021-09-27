### Definition
Signal là cách đơn giản nhất để 2 tiến trình (process) trên cùng một máy có thể giao tiếp với nhau thông qua việc một process sẽ gửi đến cho process còn lại một signal. **Unix signals are form of soft interupt**, có nghĩa là một signal giống như một software interupt, mỗi signal sẽ có những ảnh hưởng, tác động riêng. Một process có thể khai báo một *signal handler* để override lại action mặc định của signal. Handler này là một function được thực thi bất đồng bộ (async).
-   System calls: là một chanel để giao tiếp giữa program và kernel
-   Signal: là một chanel dùng cho IPC (Inter-process communication)

Signals được thiết kế như một cách để OS thông báo tới các chương trình một lỗi hoặc một biến cố nào đó (critical event). Signal còn hữu dụng trong một số IPC (POSIX-standard signal bao gồm USR1 và USR2). Nó thường được hiểu là một custom signal, dùng cho những tác vụ chạy daemon. Nó là một cách để operator hoặc program khác đưa ra tín hiệu cho tác vụ chạy daemon cần khởi tạo lại (reinitialize), wake up hoặc ghi những thông tin internal-state/debugging vào một địa chỉ nào đó.
### Features
Một signal là một async event được gửi đến một process.
- Asyn nghĩa là một event có thể xảy ra ở bất kỳ thời điểm nào.  Có thể nó không liên quan đến sự thực thi của process, eg: enter Ctrl-C
- Unix support signal facility, có nghĩa là nó giống như 1 software version của một interrupt subsystem với một CPU
- Process có thể gửi signal cho process khác và Kernel có thể gửi signal cho một process
- Process có thể ignore/discard signal (trừ `SIGKILL` hoặc `SIGSTOP`). Nó cũng có thể thực thi một signal handler function và sau đó resume exec hoặc terminate, hay là thực thi default action của signal đó.


Một số signal thường gặp:
- **SIGINT**: là một program interrupt signal. Nó được gửi cho bất kỳ process nào đang được gán với bàn phím và người dùng enter interrupt character (thường là Ctrl-C). Nó sẽ terminate process nhưng nó cũng có thể bị ignored.
- **SIGTERM**: là một termination signal. Nó thường được dùng để kết thúc một process, signal này cũng có thể bị block, handle hoặc ignore. `kill` command mặc định sẽ sinh ra SIGTERM signal.
- **SIGKILL**: là một immediate termination signal. Nó không thể bị ignore bởi process. Ví dụ, khi một process không được kết thúc bởi Ctrl-C (SIGINT) thì cũng ta thường dùng command `kill -9 <PID>`
- **SIGTSTP**: suspend/stop process, thường dùng lệnh Ctrl-Z
- **SIGCONT**:  dùng lệnh `fg` để generate signal, dùng để wakeup (khôi phục) suspend process trước đó.
- **USR2**: custom signal, việc control và thực thi phụ thuộc vào process của bạn ví dụ như khởi tạo lại hoặc wake up. Ví dụ như chạy puma server ở chế độ daemon, thay vì chúng ta kill process với `SIGKILL` thì nên dùng `USR2` để reload/khởi tạo lại server.

Một technique thường được sử dụng với signal IPC đó là `pidfile`. Chương trình cần được signal thì sẽ ghi vào một file nhỏ ở một vị trí định trước (thường ở `/var/run` hoặc ở thư mục home của user) chứa PID. Những chương trình khác có thể đọc và xác định được PID đó. Pidfile có thể hoạt động như một lock file trong trường hợp chỉ có 1 instance daemon đang chạy ở thời điểm đó.
### COMMON SIGNAL NAMES AND NUMBERS
| Number | Name | Description | Used for |
| -------- | -------- | -------- | -------- |
| 0 | SIGNULL | Null | Check access to pid |
| 1 | SIGHUP | Hangup | Terminate; can be trapped |
| 2 | SIGINT | Interrupt | Terminate; can be trapped |
| 3 | SIGQUIT | Quit | Terminate with core dump; can be trapped |
| 9 | SIGKILL | Kill | Forced termination; cannot be trapped |
| 15 | SIGTERM | Terminate | Terminate; can be trapped |
| 24 | SIGSTOP | Stop | Pause the process; cannot be trapped |
| 25 | SIGTSTP | Terminal | stop Pause the process; can be trapped |
| 26 | SIGCONT | Continue | Run a stopped process |

### Refs
https://s905060.gitbooks.io/site-reliability-engineer-handbook/content/signals.html