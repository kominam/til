#Docker #Tips #Performance #bestPractice
### Tối ưu image
+ Mỗi step của Dockerfile sẽ tạo ra 1 layer, từ đó tăng kích thước image => ta phải giảm số lượng layer đi, và cũng phải đảm bảo kích thước của nó không quá lớn. Để làm được điều này, hãy chú ý:

  + (1) Gộp những câu lệnh như COPY, RUN.
  + (2) Bỏ đi những dependencies không cần thiết khi chạy container.
  + (3) Giảm dung lượng base image: image -> image alpine.

#### Gộp câu lệnh
+ Mỗi lệnh trong Dockerfile sẽ tạo ra 1 layer mới, nó sẽ làm tăng kích thước image sau khi build.

+ Một số lệnh có khả năng cache

  + `yarn install` sử dụng `yarn.lock` và `package.json`
  + `bundle install` sử dụng `Gemfile.lock` và `Gemfile`

+ Ta nên copy những file này vào image trước khi những lệnh này, nó sẽ sử dụng cache thay vi cài đặt lại từ đầu

  ```Dockerfile
  COPY Gemfile* ./
	COPY package.json yarn.lock ./
  RUN bundle install
  ```

+ Nên gộp những lệnh RUN lại với nhau:

    ```Dockerfile
    RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
      curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
      apt-get update && \
      apt-get install -y build-essential mysql-client libv8-dev nodejs yarn
    ```

#### Bỏ dependencies không cần thiết
  + Từ Docker 17.05 trở đi, Docker đã cho ra mắt **Multi-stage builds** để cứu rỗi linh hồn của những con chiên lạc lối.

    `Dockerfile`

    ```Dockerfile
    # builder
    FROM golang:1.7.3 as builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html
    COPY app.go    .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

    # main image
    FROM alpine:latest
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"]
    ```

  + Khi build thì ta chỉ cần

    ```shell
    docker build -t alexellis2/href-counter:latest .
    ```

  + Magic ở đâu?

    ```Dockerfile
    FROM golang:1.7.3 as builder
    ...
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    ```

  + Chính nó, **FROM xyz as builder** và **COPY --from=builder** (builder chỉ là example, bạn ghi tên mình vào đó cũng được)

  + Trong khi build image chính của ta Docker đã build thêm một image nữa, chính là phần builder ta đã khai báo.

  + Sau khi build, ta có thể dùng COPY với tham số `--from=stage_name` để copy bất cứ thứ gì trong image trên. Không dừng lại ở đó, cậu lệnh này còn cho phép ta copy file từ image (đã build) ở ngoài, thậm chí là remote image.

    ```Dockerfile
    COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
    ```

  + Do ta chỉ copy một phần nhỏ của image builder, nên tất nhiên là chỉ những thứ ta copy mới tính vào kích cỡ image của ta, những thứ đời thừa khác không liên quan. Từ đó, ta đã giảm được một lượng kích thước đáng kể.

  + Khi Dockerfile của ta có nhiều stage, nếu chỉ muốn build 1 stage nào đó, hãy dùng thêm tham số `--target` khi build:

    ```
    docker build --target builder -t alexellis2/href-counter:latest .
    ```

  + Khi chạy câu lệnh trên, Docker sẽ chỉ build builder stage mà thôi, còn mặc định nó sẽ chạy hết tất cả stage. (Note: khi chạy những stage dưới thì nó cũng sẽ chạy những stage trên)

-> Những ví dụ trên đều có ở trang chủ Docker, có thể xem thêm tại [đây](<https://docs.docker.com/develop/develop-images/multistage-build/>)

###  Tăng tính bảo mật

+ Chỉ nên viết những lệnh cần thiết để khởi tạo môi trường, một số lệnh không liên quan hoặc có thể bị replace bởi volumes thì không nên viết trong Dockerfile

+ Nếu chạy `yarn install` trong quá trình build, mà sau đó ta lại mount thư mục hiện tại vào trong container, thì node_modules sau khi chạy yarn install sẽ bị replace ngay và luôn => mất => vô dụng

+ Nhớ sử dụng EXPOSE, mặc dù không viết thì nó vẫn chạy như thường.

###  CMD vs ENTRYPOINT

#### Mở đầu

+ Mỗi khi container được bật, nó đều cần có 1 lệnh nào đó để thực thi, làm nhiệm vụ của mình. `CMD` và `ENTRYPOINT` chính là những gì ta cần.

+ Cơ mà khi chúng ta viết Dockerfile hay docker-compose, ta chỉ thường chú ý đến command hay CMD, vậy ENTRYPOINT là gì vậy?

#### Magic

+ Lấy 1 ví dụ đơn giản với image `ubuntu`

  `Dockerfile`

  ```Dockerfile
  FROM ubuntu
  CMD ["echo", "Hello world"]
  ```

+ Khi ta chạy image, nó sẽ in ra màn hình "Hello world". Đơn giản nó là chạy `echo "Hello world"`?

+ Thực ra là nó đã chạy `/bin/sh -c echo "Hello world"`. WTF Magic gì vậy?

+ Vì Docker mặc định cho ta `ENTRYPOINT ["/bin/sh", "-c"]`. Và câu lệnh cuối cùng mà container chạy là `command = ENTRYPOINT + CMD`, chứ không đơn thuần chỉ là `CMD`.

#### ENTRYPOINT VS CMD

* `ENTRYPOINT`:
  * Là command chạy mỗi khi container start
  * Required, mặc định là `/bin/sh -c`
* `CMD`:
  * Mảnh ghép còn lại của cuộc đời `ENTRYPOINT`, tuy nhiên có hay không cũng chả quan trọng, vì nếu chỉ 1 mình `ENTRYPOINT` là chạy được rồi thì không cần thêm `CMD` nữa.
  * Không có giá trị mặc định

Khi ta dùng `docker start` hay `docker run`, Docker cũng đều dùng `CMD` của ta để append vào `ENTRYPOINT`, sau đó mới chạy.

VD

```Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/cat"]
```

Khi ta chạy `docker run x /etc/passwd`, thực chất là ta đang chạy `/bin/cat /etc/passwd`, như vậy file `/etc/passwd` sẽ được in ra màn hình.

#### Style khi viết ENTRYPOINT và CMD

Có 2 phong cách viết `ENTRYPOINT`/`CMD`

* Shell form: `CMD rails s -b 0.0.0.0"`

* Exec form: `CMD ["rails", "s", "-b", "0.0.0.0"]`

Ta có thể dùng kiểu nào cũng được, tuy nhiên Docker khuyên ta nên dùng Exec form hơn.

Shell form invoke 1 câu lệnh shell, với đầy đủ tính năng của shell, ta có thể truy cập biến ENV, dùng &&, ...

Còn với exec form, Docker parse chuỗi của ta theo format JSON, vì vậy ta phải dùng mảng của các string (nhớ double quote), sau đó làm gì đó ma giáo, nhưng không chạy như 1 câu lệnh shell, vì vậy những tính năng của shell đều cuốn theo chiều gió hết. Muốn sử dụng, ta phải thêm lệnh shell vào `CMD ["sh", "-c", "echo $HOME"]` (tuy nhiên ta đã có default ENTRYPOINT là `/bin/sh -c` rồi nên chỉ khi bạn override nó mới cần lưu ý.)

Lưu ý nữa là khi dùng `ENTRYPOINT` theo shell form, `CMD` và command của `docker run` sẽ bị ignore hết.

Ngoài ra, nó còn có liên quan đến shell processing, signal processing, khá là hại não, mọi người có thể dựa vào những keyword này để tìm hiểu thêm.

### So sánh LINK và DEPENDS_ON

Hiện tại `LINK` đã bị deprecated, không nên dùng nó nữa.

Trước đây, `LINK` dùng để liên kết các container, từ đó, container này có thể gọi đến container kia thông qua alias của nó (`NETWORK`). Ngoài ra nó còn 1 side-effect nữa là quy định trình tự start các container (`DEPENTS_ON`).

Dễ dàng thấy nó là `NETWORK + DEPENTS_ON`

Một note nhỏ là `depends_on` không chờ cho container khác *ready to work* mà nó chỉ chờ bật lên mà thôi, `depends_on` bị ignore khi dùng swarm mode.

### Khi nào cần build lại image?
+ Khi hệ thống - nội tại image cần có sự thay đổi
  + cài gem mới (khi chạy bundle install)
  + cài thêm system dependencies
    + apt-get install

+ Khi mà hệ thống không phụ thuộc vào những thay đổi thì không cần rebuild
  + cài gem bằng bundle install --path vendor/bundle
  + yarn install

+ Ở môi trường dev:
  + Ta thường có config volumes: .:/path/to/app, vì vậy khi chạy những lệnh trên, nó sẽ trực tiếp mount luôn ra ngoài host, những lần chạy service sau đó, nó cứ lấy thư mục gem kia mount vào container và chạy ầm ầm thôi.

##### Note
  + Chúng ta có thể bỏ option `networks` trong file `docker-compose.yml` nếu chúng ta không cần config gì khác như `ip`, `subnet`,... (dùng luôn default network khi start container).
  + Nếu không có nhu cầu access vào database từ host machine thì cũng không cần bind `port` ra :smile: Cụ thể là ở trong service `mysql` chúng ta có thể bỏ options `ports`.
  + Một số file `docker-compose.yml` dùng `links` thay vì `depends_on` thì thay vì dùng short-hand syntax như:
    ```yaml
    web:
      links:
      - db
    ```
  + Chúng ta nên dùng `<tên service>:<service alias>` để dễ dàng cho việc maintain
    ```yaml
    web:
      links:
      - mysql:db
    ```
  + Khi làm việc với `cron jobs` chúng ta cần chú ý tới việc setting timezone cho container. Chúng ta có thể setting cho chúng thông qua `Dockerfile` hoặc biến env khi chạy container.