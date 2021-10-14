JavaScript là một ngôn ngữ xử lý đơn luồng (**single thread**), tức là trong một thời điểm chỉ có một task được thực thi.
### How JavaScript perform async tasks
Nếu bạn muốn xử lý một task bất đồng bộ hoặc một tác vụ chiếm khoảng 30s mới hoàn thành (asynchronous), ví dụ sau 5s thì hiển thị ra console dòng chữ "hello world!".

Thật may mắn là trình duyệt (browser) cung cấp cho chúng ta một vài tính năng giúp JavaScript làm việc đó: **Web API**.

Web API bao gồm nhiều thứ, một trong số đó bạn thường xuyên sử dụng như:
- DOM APIs
- setTimeout()
- fetch()
- localStorage
- console
- location

Tất cả những thứ đó sẽ giúp bạn tạo async và non-blocking tasks🚀.

Khi invoke một function, nó sẽ được add vào trong một stack gọi là **Call stack**. Call stack là một phần của JavaScript engine. Khi function đó trả về kết quả (return value) thì nó sẽ được lấy ra khỏi call stack (pop).

![](https://res.cloudinary.com/practicaldev/image/fetch/s--44yasyNX--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gid1.6.gif)

Trong ví dụ trên, `respond` function trả về một `setTimeout` function. Callback trong setTimeout function là một arrow function sẽ trả về giá trị là `'Hey!'`, và nó sẽ được add vào Web API. Trong lúc đó thì `setTimeout` và `respond` function sẽ được pop ra khỏi call stack.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--d_n4m4HH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif2.1.gif)

Trong Web API, timer sẽ chạy trong vòng 1000ms. Sau đó thì callback không được add vào call stack ngay lập tức mà nó sẽ được chuyển sang 1 queue. Đó là **Callback queue** hay còn gọi là Task queue.

![](https://res.cloudinary.com/practicaldev/image/fetch/s--MewGMdte--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif3.1.gif)

Và giờ thì là lúc event loop thực hiện nhiệm vụ của nó. Event loop sẽ check call stack, **nếu call stack rỗng (empty)** thì nó sẽ add item đầu tiên của callback queue vào call stack. Trong trường hợp này thì không có function nào khác được invoke. 

![](https://res.cloudinary.com/practicaldev/image/fetch/s--b2BtLfdz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif4.gif)

và sau đó thì callback sẽ được invoke và trả về giá trị rồi pop ra khỏi callstack. 

![](https://res.cloudinary.com/practicaldev/image/fetch/s--NYOknEYi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif5.gif)

Hãy thử trên console một ví dụ và đoán xem kết quả là gì nhé:
``` JavaScript
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```

![](https://res.cloudinary.com/practicaldev/image/fetch/s--BLtCLQcd--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://devtolydiahallie.s3-us-west-1.amazonaws.com/gif14.1.gif)

### MicroTask Queue
Cùng xem một ví dụ sau với `fetch()` api:
``` JavaScript
setTimeout(function cb() {
  console.log('Hi there');
}, 5000);

fetch('https://api.github.com')
.then(function res() {
  console.log('done');
});
```

Trong đoạn code trên, JavaScript engine sẽ thực hiện tuần tự code từ trên xuống dưới.
- `cb` function sẽ được register trên Web API và sau 5s sẽ được đẩy vào callback queue.
- Giả sử fetch api trả về response trong khoảng 2s, bạn nghĩ rằng `res` function cũng sẽ được đưa vào callback queue?

Không phải vậy, `res` function sẽ được push vào 1 queue khác có tên là **MicroTasks Queue**.

**MicroTasks Queue** là một queue chứa các task có độ ưu tiên cao (high priority). Những callback function từ `Promise` hoặc `MutationObserver` sẽ được push vào trong queue này. 

Event loop sẽ check các task và ưu tiên item trong microtask queue trước, callback queue sẽ xử lý sau. 

Vậy trong trường hợp các tasks trong micro task queue lại tạo ra một task khác cũng là promise và được đẩy vào microtask queue thì chẳng phải là các task ở trong callback queue sẽ phải chờ rất lâu hoặc ko có cơ hội thực hiện hay sao? đúng vậy, trường hợp này được biết đến với cái tên là **Starvation**
### References
https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif