# PIE (Position Independent Executable)
PIE là thuộc tính của file binary đc biên dịch dưới dạng mã ko phụ thuộc vị trí, nên có thể đc nạp một cách ngẫu nhiên vào bộ nhớ qua bất kì address nào.
ASLR (Address Space Layout Randomization) là cơ chế bảo mật của hệ điều hành, xếp ngẫu nhiên vị trí các vùng nhớ của chương trình mỗi lần chạy (Stack, Heap, libc), giúp chống lại việc khai thác ở các vị trí cố định.
Nếu binary k bật PIE thì code/data segment vẫn đc lưu tại địa chỉ cố định (như vùng 0x400000 trên 64-bit), tạo điều kiện cho khai thác lỗ hổng. PIE khắc phục điều đó bằng cách cho địa chỉ của code/data segment cũng trở nên ngẫu nhiên (dạng 0x55... hoặc 0x56...). Và PIE cũng chỉ hoạt động khi ASLR đang bật.

Điều này khiến cho việc sử dụng gadget và tìm địa chỉ hàm trong chương trình trở nên khó khăn hơn.

Vì chỉ có base address bị ngẫu nhiên (luôn chia hết cho 0x1000, tức 3 số cuối là 000), nên offset giữa các hàm vẫn đc giữ nguyên =>> chỉ cần leak ra đc 1 địa chỉ trong binary là có thể tính ra đc mọi địa chỉ khác (PIE_base = leaked_addr - offset).

NX (No-execute)

Là cơ chế bảo vệ ở mức phần cứng (dùng NX bit trong Page Table) và hệ điều hành, gắn cờ các vùng nhớ chứa dữ liệu như Stack, Heap, .data, .bss là read/write only (rw-) và k thể thực thi.

Nếu người khai thác chèn shellcode vào stack rồi ghi đè return address để nhảy tới đó, CPU sẽ phát hiện thực thi tại vùng không có quyền và hệ điều hành sẽ gửi tín hiệu SIGSEGV dừng chương trình.

Muốn vượt qua thì k chạy shellcode trực tiếp đc nữa mà phải chuyển sang tái sử dụng code có sẵn như ROP, ret2libc hoặc gọi mprotect để cấp lại quyền thực thi cho vùng nhớ.

RELRO (RELocation Read-Only)

Là cơ chế chuyển bảng GOT sang chế độ read only, để kẻ tấn công k ghi đè đc các địa chỉ trong đó.

Các mức RELRO:

No RELRO

Không có lớp bảo vệ nào, toàn bộ GOT ghi được (rw-), thứ tự các section không được sắp xếp lại.

Partial RELRO

Linker sắp xếp lại các section để .got, .dynamic, .fini_array, .init_array nằm trước các biến toàn cục (.data/.bss), và đánh dấu chúng là read only sau khi nạp. Nhờ vậy lỗi tràn bộ đệm từ .data/.bss không thể ghi đè lên các vùng này.

Tuy nhiên, .got.plt (chứa địa chỉ các hàm liên kết động) vẫn dùng Lazy Binding và vẫn có quyền rw- =>> vẫn thực hiện GOT overwrite ở đây được.

Full RELRO

Tắt hoàn toàn Lazy Binding: ngay khi chương trình khởi chạy, dynamic linker phân giải địa chỉ của toàn bộ hàm thư viện và điền sẵn vào bảng GOT.

Sau khi điền xong, linker gọi mprotect() để đổi quyền của toàn bộ vùng GOT từ rw- thành r--. Lúc này .got.plt không còn ghi được, .init_array và .fini_array cũng chỉ đọc =>> chặn đứng hoàn toàn GOT overwrite.

Đổi lại, thời gian khởi động chương trình lâu hơn một chút vì phải phân giải hết symbol ngay từ đầu.

Canary

Là cơ chế giúp phát hiện và ngăn chặn các cuộc tấn công buffer overflow nhắm vào return address trên stack.

Vì buffer được ghi từ địa chỉ thấp lên cao, muốn tràn tới saved RBP hoặc return address thì bắt buộc phải ghi đè qua Canary trước.

Cách thức hoạt động:

Khi chương trình chạy, hệ thống sinh ngẫu nhiên giá trị Canary gốc và lưu ở fs:0x28 (trên 64-bit), sau đó chép xuống đặt trên stack ngay trước saved RBP.

Trước khi ret, chương trình so sánh giá trị canary hiện tại trên stack với giá trị gốc ở fs:0x28. Nếu:

Giống: hàm return bình thường.

Khác: chương trình gọi __stack_chk_fail() để dừng ngay lập tức, ngăn không cho return address bị kiểm soát.

Canary sẽ được tạo lại từ đầu mỗi lần process khởi động nên không thể đoán trước, đồng thời byte thấp nhất của nó luôn là 0x00 (null-byte) để chặn các hàm chuỗi như puts hay printf("%s") vô tình làm leak mất. Muốn bypass thì phải leak đc canary trước rồi chèn đè lại đúng giá trị đó vào payload.
