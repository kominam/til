### Một số khái niệm
- **SSL là gì?**: viết tắt của từ Secure Sockets Layer. Đây là một tiêu chuẩn an ninh công nghệ toàn cầu tạo ra một liên kết giữa máy chủ web và trình duyệt. SSL được thay thế bởi TLS
- **Certificate Authority (CA)**: Là nhà cung cấp chứng chỉ **SSL** cho **server** sử dụng
- **Cipher suites**: Là tập hợp các thuật toán mã hoá và giải mã mà **client** và **server** sẽ sử dụng.
### Các bước TLS handshake
**1. Client gửi tới server "hello" mesage** để bắt đầu tiến trình handshake. Message này chứa một vài thông tin như phiên bản SSL, phiên bản cipher suites và **client random**
**2. Server gửi "hello" message** Trong message này có chứa **SSL certificate** của server, cipher suite được dùng, và 1 chuỗi bytes ngẫu nhiên nữa được gọi là **server random**
**3. Authentication** Client xác thực SSL certificate mà server gửi với CA. 
**4. The premaster secret** client sẽ gửi tiếp lên server một chuỗi bytes nữa, gọi là **premaster secret**. premaster secret này đã được mã hoá bằng **public key** mà server gửi trước đó.
**5. Private key used** server giải mã premaster secret với private key
**6. Session keys created** tạo **session key** từ 3 keys trước đó: `client random, server random và premaster secret`
**7. Client is ready** client gửi message "finished" được mã hóa bắng session key
**8. Server is ready** server gửi message "finished" được mã hóa bằng session key
**9. Secure symmetric encryption achieved** handshake hoàn tất và từ giờ các message được mã hóa bằng session key vừa tạo.

![[SSL_handshake.png]]

Khi nói là server trả về SSL certificate không có nghĩa là server sẽ chỉ trả về một certificate. Thực tế thì server trả về một chuỗi certificate (certificate chain).
![[certificate_chain.png]]

**Note:** để có thể SSL/TLS handshake thì chỉ cần trust root certificate là được.

Như vậy thì system chỉ cần trust root certificate còn không quan tâm là certificate cuối cùng trong certificate chain có phải là của google site hay không.

Điều này dẫn đến issue là hacker có thể tự cài đặt root certificate vào hệ thống. (MITM attack)

![[SSL_certificate_MITM_attack.png]]

### SSL pinning là gì ?
SSL pinning là một kỹ thuật (technique) sử dụng ở phía client bằng cách validate certificate của server (certificate cuối cùng trong certificate chain) một lần nữa sau khi SSL handshake. Có thể hiểu là chúng ta ghim (pin) một list trustful certificates ở phía client app và dùng để so sánh với certificate của server trong quá trình chạy (runtime). Nếu validate không trùng khớp thì kết nối sẽ bị ngắt (disrupted).

Có hai cách để implement đó là:
**certificate pinning**: tức là pin cả certificate ở phía client, dùng sau khi handshake thì sẽ validate certificate mà phía server trả về với certificate đã pin ở phía client
**public key pinning**: khi lấy được thông tin certificate phía server trả về thì tách và lưu lại public key. Sau khi handshake thì so sánh public key trong certificate và public key đã lưu xem có match nhau không.

**Note:** với certificate pinning , khi một certificate đã được ghim bị hết hạn, phải nhớ update lại ở phía client application. Nếu không validate giữa server certificate mới và client pinned certificate sẽ luôn luôn không trùng khớp.