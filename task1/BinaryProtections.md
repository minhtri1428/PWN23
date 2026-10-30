# PIE (Position Independent Executable
- PIE là thuộc tính của file binary đc biên dịch dưới dạng mã ko phụ thuộc vị trí, nên có thể đc nạp một cách ngẫu nhiên vào bộ nhớ qua bất kì address nào
- ASLR (Address Space Layout Randomization) là cơ chế bảo mật của hệ điều hành, xếp ngẫu nhiên vị trí các vùng nhớ của chương trình mỗi lần chạy. Giúp chống lại việc khai thác Stack, Heap, Libraries (libc) ở các vị trí cố định.
- Nếu binary k bật PIE thì code/data segment vẫn đc lưu tại địa chỉ cố định, tạo điều kiện cho khai thác lỗ hổng. PIE khắc phục điều đó bằng cách cho địa chỉ của code/data segment cũng trở nên ngẫu nhiên. Và PIE cũng chỉ hoạt dộng khi ASLR đang bật
- Điều này khiến cho việc sử dụng gadget và tìm địa chỉ hàm trong chương trình trở nên khó khăn hơn.
- Vì chỉ có base address bị ngẫu nhiên, nên offset giữa các hàm vẫn dc giữ nguyên =>> chỉ cần leak ra đc 1 địa chỉ trong binary là có thể tính ra đc mội địa chỉ khác.
# NX (No-execute)
- Là cơ chế bảo vệ ở mức phần cứng và hệ điều hành, gắn cờ các vùng nhớ chứa dữ liệu như Stack, Heap, .data, .bss là read/write only và k thể thực thi.
- Nếu người khai thác chèn shellcode vào stack rồi ghi đè return address để nhảy tới đó, CPU sẽ phát  hiện thực thi tại vùng không có quyền và hệ điệu hành sẽ gửi tín hiệu dừng chương trình.
# RELRO (RELocation Read-Only)
- Là cơ chế chuyển bảng GOT sang chế độ read only, để kẻ tấn công k ghi đè đc các địa chỉ trong đó.
## Các mức RELRO 
### 1. No RELRO
- Không có lớp bảo vệ nào, toàn bộ GOT ghi được, thứ tự các section không được sắp xếp lại.
### 2. Partial RELRO 
- Linker sắp xếp lại các section để `.got`, `.dynamic`, `.fini_array`, `.init_array` nằm trước các biến toàn cục (.data/.bss), và đánh dấu chúng là read only sau khi nạp. Nhờ vậy lỗi tràn bộ đệm từ .data/.bss không thể ghi đè lên các vùng này.
- Tuy nhiên, `.got.plt` (chứa địa chỉ các hàm liên kết động) vẫn dùng Lazy Binding và vẫn có quyền `rw-`.
### 3. Full RELRO
- Tắt hoàn toàn Lazy Binding: ngay khi chương trình khởi chạy, dynamic linker phân giải địa chỉ của toàn bộ hàm thư viện và điền sẵn vào bảng GOT.
- Sau khi điền xong, linker gọi `mprotect()` để đổi quyền của toàn bộ vùng GOT từ `rw-` thành `r--`. Lúc này .got.plt không còn ghi được, `.init_array` và `.fini_array` cũng chỉ đọc.
- Đổi lại, thời gian khởi động chương trình lâu hơn một chút vì phải phân giải hết symbol ngay từ đầu.
# Canary
- Là cơ chế giúp phát hiện và ngăn chặn các cuộc tấn công buffer overflow nhắm vào return address trên stack.
- Vì buffer được ghi từ địa chỉ thấp lên cao, muốn tràn tới saved RBP hoặc return address thì bắt buộc phải ghi đè qua Canary trước.
### Cách thức hoạt động
- Chương trình lưu lại giá trị gốc của canary và đặt lên stack.

Trước khi `ret`, so sánh giá trị canary hiện tại trren stack với giá trị gốc đã lưu trước đó. Nếu:
- Giống : hàm return bình thường.
- Khác : hàm sẽ bị dừng ngay lập tức, ngăn không cho return address bị kiểm soát.
- Canary sẽ được tạo lại từ đầu mỗi lần process khởi động, nên kẻ tấn công không thể đoán trước giá trị của nó.
