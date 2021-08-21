Index là một cấu trúc dữ liệu riêng biệt trong DB, nó sở hữu một vùng nhớ riêng trên ổ cứng (disk). Nó sẽ lưu một bản copy của dữ liệu được đánh index. Việc tìm kiếm theo index rất nhanh vì dữ liệu đã được sắp xếp theo thứ tự

Khi các lệnh `INSERT, DELETE, UPDATE` được thực hiện, ngoài thay đổi dữ liệu trong bảng, DB còn phải thay đổi dữ liệu trong index sao cho dữ liệu vẫn phải được sắp xếp theo đúng thứ tự. Để thực hiện việc này, database sử dụng 2 kiểu cấu trúc dữ liệu: `Doubly Linked List` và `Tree`.

Mỗi khi `INSERT` một bản ghi, để đảm bảo index được sắp xếp đúng thứ tự thì DB sẽ phải dịch chuyển các bản ghi có giá trị lớn hơn bản ghi được insert vào, khiến cho việc INSERT mất nhiều thời gian. Để giải quyết vấn đề này, DB sẽ thiết lập một thứ tự logic mà độc lập với vị trí của dữ liệu trong bộ nhớ.

Với logical order thì DB sẽ sử dụng cấu trúc dữ liệu Doubly Linked List, mỗi một node sẽ có 2 con trỏ: 1 con trỏ hướng đến node liền trước, 1 con trỏ hướng đến node liền sau. Khi node mới được tạo ra, 2 node liền trước và liền sau node mới này chỉ cần cập nhật lại vị trí con trỏ.

Database dùng Doubly Linked List để kết nối các *index leaf nodes*. Mỗi một index leaf nodes sẽ được lưu trong một DB block hoặc page (đây là đơn vị lưu trữ nhỏ nhất trong DB). Mỗi index block sẽ có dung lượng như nhau - vài Kb. Trong mỗi block sẽ chứa nhiều nhất dữ liệu có thể của cột được đánh index (index entry). Điều này có nghĩa index duy trì trật tự của nó ở 2 mức: giữa các index entry trong cùng 1 block và giữa các block với nhau.

![[Pasted image 20210805222609.png]]

Hình ảnh mô tả mối liên hệ giữa index leaf node và dữ liệu trong bảng. Mỗi index entry bao gồm cột được đánh index (key, column 2) và tham chiếu tới hàng tương ứng trong bảng (thông qua ROWID hoặc RID). Không giống như index, dữ liệu trong bảng được lưu trữ theo cấu trúc heap và không được sắp xếp. Không có bất kì mối liên hệ nào giữa các bản ghi trong 1 table block hay giữa các table blocks với nhau

### B-tree structure
Các index leaf nodes được săp xếp theo một trật tự logic (arbitrary order), nghĩa là vị trí của một bản ghi trên disk rất có thể không khớp với vị trí của bản ghi đó nếu chiếu theo trật tự của index leaf nodes. Giống như khi bạn mở 1 cuốn danh bạ điện thoại để tìm số điện thoại của Smith nhưng khi bạn mở trang đầu tiên thì cái tên đầu tiên là Robinson, tức là chẳng có gì đảm bảo rằng Smith sẽ là cái tên tiếp theo. Vì thế, DB cần sử dụng một kiểu cấu trúc dữ liệu khác nữa để tìm kiếm nhanh: `balanced search tree (B-Tree)`
![[Pasted image 20210805223758.png]]
Ảnh này là ví dụ của index với 30 entries. Cấu trúc Doubly Linked List thiết lập một trật tự logic giữa các index leaf nodes. Còn các root và branch node hỗ trợ tìm kiếm nhanh giữa một rừng các leaf node

Trong cấu trúc này, mỗi một branch node sẽ trỏ đến một index entry có giá trị lớn nhất trong một leaf node. Trong ví dụ trên thì 46 sẽ là giá trị lớn nhất trong các leaf node 40, 43, 46 và các leaf node khác tương tự. Các branch node tiếp theo được xây dựng tương tự, lớp sau được tạo nên bởi các giá trị lớn nhất của lớp trước, cho đến khi có một node lớn nhất gọi là **root**. 
**Note**: A B-tree is a balanced tree—not a binary tree.
### B-tree traversal
![[Pasted image 20210805224523.png]]
Quá trình tìm kiếm key 57 như sau:
- Bắt đầu từ root node với 3 giá trị phía ngoài cùng bên tay trái: 39, 83, 98
- Mỗi entry sẽ được so sánh với key search (57) cho đến khi value lớn hơn hoặc bằng key search. ở đây nó sẽ dừng lại ở entry số 83 vì 39 nhỏ hơn 57 nên dữ liệu tìm kiếm sẽ không có ở branch node 39.
- Tiếp tục lặp lại bước trên cho đến khi tìm đến leaf node. khi đến leaf node 57 đầu tiên, nó vẫn sẽ tiếp tục tìm hết vì biết đâu sẽ có thêm giá trị nữa
- Cuối cùng kết quả thu được là 2 giá trị 57 ứng với 2 ROW ID trỏ tới 2 bản ghi trong table data

Quá trình tìm này rất hiệu quả vì:
- Mọi node được tìm ra với số lượng step là như nhau
- Độ tăng trưởng của cây là chậm (logarithmic growth of tree depth). Điều này có thể hiểu rằng độ sâu của cây tìm kiếm sẽ phát triển chậm so với số leaf node. Trên thực tế thì độ sâu của index với hàng triệu record là 4 hoặc 5.