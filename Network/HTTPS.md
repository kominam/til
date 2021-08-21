### Một số khái niệm
- **SSL là gì?**: viết tắt của từ Secure Sockets Layer. Đây là một tiêu chuẩn an ninh công nghệ toàn cầu tạo ra một liên kết giữa máy chủ web và trình duyệt. SSL được thay thế bởi TLS
- **Certificate Authority (CA)**: Là nhà cung cấp chứng chỉ **SSL** cho **server** sử dụng
- **Cipher suites**: Là tập hợp các thuật toán mã hoá và giải mã mà **client** và **server** sẽ sử dụng.
### Các bước TLS handshake
**1. Client gửi tới server "hello" mesage** để bắt đầu tiến trình handshake. Message này chứa một vài thông tin như phiên bản SSL, phiên bản cipher suites và **client random**
**2. Sẻver gửi "hello" message** Trong message này có chứa **SSL certificate** của server, cipher suite được dùng, và 1 chuỗi bytes ngẫu nhiên nữa được gọi là **server random**
**3. Authentication** Client xác thực SSL certificate mà server gửi với CA. 
**4. The premaster secret** client sẽ gửi tiếp lên server một chuỗi bytes nữa, gọi là **premaster secret**. premaster secret này đã được mã hoá bằng **public key** mà server gửi trước đó.
**5. Private key used** server giải mã premaster secret với private key
**6. Session keys created** tạo **session key** từ 3 keys trước đó: `client random, server random và premaster secret`
**7. Client is ready** client gửi message "finished" được mã hóa bắng session key
**8. Server is ready** server gửi message "finished" được mã hóa bằng session key
**9. Secure symmetric encryption achieved** handshake hoàn tất và từ giờ các message được mã hóa bằng session key vừa tạo.
![[Pasted image 20210820171507.png]]