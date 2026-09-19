# 1. Kiến trúc máy tính tổng quát
## - Mô hình von Neumann
<img width="782" height="567" alt="image" src="https://github.com/user-attachments/assets/a8d70ea6-2476-42b8-b0d5-911ae0cc7042" />

### - Ý tưởng cốt lõi của von Neumann: code (chương trình) và data (dữ liệu) được lưu chung trong một vùng nhớ, CPU đọc lệnh từ bộ nhớ, giải mã, rồi thực thi— đây là lý do vì sao các lỗi như buffer overflow có thể nguy hiểm: dữ liệu và lệnh nằm cùng một không gian địa chỉ, nên ghi đè dữ liệu có thể ảnh hưởng tới luồng thực thi.
## - Các thành phần chính của CPU
- Control Unit (CU): điều phối, giải mã lệnh, quyết định lệnh tiếp theo cần lấy từ đâu (dựa vào rip).
- ALU (Arithmetic Logic Unit): thực hiện các phép toán số học (cộng, trừ...) và logic (AND, OR, XOR, so sánh...).
- Registers: bộ nhớ siêu nhanh ngay trong CPU, dùng để chứa toán hạng, kết quả tạm, con trỏ (xem chi tiết ở phần 2).
- Cache (L1/L2/L3): bộ nhớ đệm giữa CPU và RAM, giúp tăng tốc truy cập dữ liệu hay dùng lại.
## - Chu trình Fetch–Decode–Execute
### - Mọi lệnh máy đều được CPU xử lý qua 3 bước lặp lại liên tục:
+ Fetch: lấy lệnh tại địa chỉ rip đang trỏ tới, từ bộ nhớ.
+ Decode: giải mã xem đây là lệnh gì (mov, add, jmp...) và toán hạng là gì.
+ Execute: thực thi lệnh đó (qua ALU nếu là phép toán, qua CU nếu là điều khiển luồng...), rồi cập nhật rip trỏ tới lệnh tiếp theo (hoặc nhảy tới địa chỉ khác nếu là lệnh jmp/call/ret).
## - Memory Hierarchy (phân cấp bộ nhớ)
#### - Thứ tự từ Nhanh / Nhỏ / Đắt đến Chậm / Lớn / Rẻ
  <img width="917" height="85" alt="image" src="https://github.com/user-attachments/assets/2f0eaf9d-9b9d-46d7-bef9-124e406e1546" />
  
=> Hiểu phân cấp này giúp giải thích lý do truy cập Register/Stack (đã nằm gần CPU, thường trong Cache) lại nhanh hơn nhiều so với việc truy cập Heap lớn hay đọc file trên đĩa.
## - Kiến Trúc 32-bit vs 64-bit
 - Khái niệm: Con số "32-bit" hay "64-bit" thường chỉ độ rộng thanh ghi và độ rộng bus địa chỉ của CPU.
 - Không gian địa chỉ: CPU 64-bit có thể địa chỉ hóa không gian nhớ lớn hơn rất nhiều ($2^{64}$ so với $2^{32}$ của 32-bit), các thanh ghi đa dụng rộng 64-bit thay vì 32-bit.
 - Phân tích Binary trên Linux: Các binary 32-bit (x86) và 64-bit (x86-64) có ABI và bộ thanh ghi khác nhau — cần chú ý khi phân tích bằng lệnh file hoặc readelf -h.
# 2. Kiến trúc x86-64
 - x86-64 (hay AMD64) là kiến trúc CPU 64-bit, mở rộng từ x86 32-bit. Một số điểm cốt lõi:
 - CPU thực thi lệnh máy (machine code) tuần tự, lệnh được nạp từ vùng nhớ text (code segment).
 - Dữ liệu được thao tác qua các thanh ghi (registers) — bộ nhớ siêu nhanh nằm ngay trong CPU.
 - Có 16 thanh ghi đa dụng 64-bit: rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8, r9, r10, r11, r12, r13, r14, r15.
 - Ngoài ra còn rip (instruction pointer — trỏ tới lệnh tiếp theo sẽ thực thi) và rflags (cờ trạng thái: zero flag, carry flag, sign flag...).
## - Cách chia nhỏ một thanh ghi (ví dụ với rax)
<img width="727" height="162" alt="image" src="https://github.com/user-attachments/assets/195f3f9e-7c14-400f-b99f-4f7d21abf411" />
<img width="1028" height="485" alt="image" src="https://github.com/user-attachments/assets/4df82a67-6b31-4e52-a9d5-5983aefb7e7a" />

### ** Lưu ý : khi ghi vào eax (32-bit), CPU tự động zero-extend — xóa toàn bộ 32 bit cao của rax. Nhưng khi ghi vào ax hoặc al, phần cao hơn giữ nguyên (không bị xóa).
## - Vai trò thường dùng của một số thanh ghi
- rax: thường chứa giá trị trả về của hàm, và trong syscall là số hiệu syscall.
 - rdi, rsi, rdx, rcx, r8, r9: chứa 6 tham số đầu tiên khi gọi hàm (System V ABI).
 - rbp: base pointer — trỏ tới đáy của stack frame hiện tại.
 - rsp: stack pointer — luôn trỏ tới đỉnh (top) của stack.
 - rip: instruction pointer — đây là mục tiêu chính khi khai thác lỗi (control rip = kiểm soát luồng thực thi chương trình).
# 3. Calling Convention (System V AMD64 ABI — dùng trên Linux)
## - Khi một hàm được gọi thì :
- Tham số truyền theo thứ tự: rdi, rsi, rdx, rcx, r8, r9. Nếu nhiều hơn 6 tham số, các tham số dư được đẩy lên stack.
- Giá trị trả về nằm ở rax (nếu là số nguyên/con trỏ).
- Khi gọi hàm bằng lệnh call, địa chỉ trở về (return address) — tức địa chỉ lệnh ngay sau call — được tự động đẩy vào stack.
- Hàm kết thúc bằng ret, lệnh này pop giá trị trên đỉnh stack ra và nhảy (jmp) tới đó — đây chính là cơ chế mà buffer overflow lợi dụng để chiếm quyền điều khiển rip.
##  Phân loại thanh ghi
<img width="1020" height="317" alt="image" src="https://github.com/user-attachments/assets/d02db622-387f-48a5-a7fa-75ae2d9ede6f" />

## * Ví dụ prologue/epilogue của một hàm điển hình
<img width="795" height="337" alt="image" src="https://github.com/user-attachments/assets/25d1938d-e387-48ec-9e1e-a8aafee1c078" />

# 4. Memory Layout của một chương trình
### - Khi chương trình chạy, không gian địa chỉ ảo của nó được chia thành các vùng, thường sắp theo thứ tự địa chỉ thấp → cao như sau:
<img width="577" height="340" alt="image" src="https://github.com/user-attachments/assets/2cd22790-259f-4341-862f-7ae4c54f1a8f" />   

- Text: chứa mã máy, thường chỉ có quyền đọc + thực thi (r-x), không ghi được.
- Data/BSS: biến toàn cục và static.
- Heap: cấp phát động (malloc, new), mọc từ địa chỉ thấp lên cao.
- Stack: nơi lưu local variable, tham số, return address; mọc từ địa chỉ cao xuống thấp.
- Giữa heap và stack có một vùng trống lớn, và còn có thêm vùng cho shared libraries (libc.so...) thường nằm gần đỉnh không gian địa chỉ hoặc ở vị trí ngẫu nhiên nếu bật ASLR (Address Space Layout Randomization).
### =>> Memory layout là nền tảng bắt buộc để hiểu các lỗi như buffer overflow (ghi đè dữ liệu trên stack), heap overflow, use-after-free...
# 5. Bit, Byte và Endianness
 - Bit: đơn vị nhỏ nhất, giá trị 0 hoặc 1.
 - Byte: 8 bit, biểu diễn được giá trị từ 0–255 (0x00–0xFF).
 - MSB (Most Significant Bit/Byte): bit/byte có trọng số lớn nhất.
 - LSB (Least Significant Bit/Byte): bit/byte có trọng số nhỏ nhất.
 ## - Endianness — cách sắp xếp byte trong bộ nhớ
- Giả sử ta có giá trị 4-byte: 0x12345678 (MBS=12 , LBS=78).
### - Little-endian (x86/x86-64): byte có trọng số thấp nhất được lưu ở địa chỉ thấp nhất.
<img width="537" height="86" alt="image" src="https://github.com/user-attachments/assets/f7ee6eb8-c6a7-4138-a770-07f299bc7e30" />

### - Big-endian (dùng trong một số kiến trúc mạng, network byte order): byte có trọng số cao nhất lưu ở địa chỉ thấp nhất.
<img width="513" height="76" alt="image" src="https://github.com/user-attachments/assets/05f256fc-c5ac-4545-8081-6fee2218312c" />

#### * x86 = little-endian → khi dump memory bằng gdb/xxd, nếu thấy chuỗi byte trông "ngược", đó là bình thường — cần đảo ngược lại để đọc ra giá trị số thật.
VD: chuỗi byte 41 41 41 41 42 42 42 42 trên stack (little-endian), nếu đọc thành 2 giá trị 4-byte, ta được 0x41414141 và 0x42424242 — đây là kiểu dữ liệu rất hay gặp khi debug buffer overflow (do 'A' = 0x41, 'B' = 0x42).
# 6. Stack
### - Stack là vùng nhớ hoạt động theo cơ chế LIFO (Last In, First Out).
### - Push và Pop
`push <giá trị>:`
- Giảm rsp đi 8 (vì mỗi lần push/pop trên x86-64 làm việc với 8 byte).
- Ghi giá trị vào địa chỉ [rsp] mới.
`pop <thanh ghi>:`
- Đọc giá trị tại [rsp].
- Tăng rsp lên 8.
<img width="916" height="147" alt="image" src="https://github.com/user-attachments/assets/59cea2e6-ee08-4b1b-86b1-b5a6452946dd" />

## - Stack frame của một hàm
### - Mỗi khi một hàm được gọi, một "khung" (frame) mới được tạo trên stack, thường chứa:
- Return address (do call tự động push)
- Saved rbp cũa caller (do push rbp trong prologue)
- Local variables (cấp phát bằng sub rsp, N)
- Đôi khi có thêm buffer canary (stack protector) để chống overflow
<img width="1011" height="317" alt="image" src="https://github.com/user-attachments/assets/a6f6f519-8669-4bf7-b240-2fd1fa216f92" />

## - Mối liên hệ giữa Stack, Memory và Register
`rsp` và `rbp` là thanh ghi nhưng giá trị của chúng là địa chỉ bộ nhớ trỏ vào vùng stack.
- Khi một buffer local (ví dụ char `buf[16]`) bị ghi tràn (overflow) mà không kiểm tra độ dài, dữ liệu ghi thừa sẽ đè lên saved `rbp`, rồi đè lên return address. Nếu kẻ tấn công kiểm soát được return address, họ có thể điều khiển `rip` sau khi hàm `ret` — đây chính là ý tưởng cốt lõi của stack buffer overflow.
# 7. Hex và Binary
### - Bảng quy đổi nhanh
<img width="1007" height="536" alt="image" src="https://github.com/user-attachments/assets/d32f47f3-92aa-4e2e-bca5-9c512043a7d0" />

### - Cách đổi Hex ↔ Binary nhanh
- Mỗi ký tự hex tương ứng chính xác 4 bit → ghép trực tiếp, không cần qua decimal.
`Ví dụ: 0xA3 → A = 1010, 3 = 0011 → 10100011.`
### - Công cụ thực hành trên Linux
<img width="945" height="193" alt="image" src="https://github.com/user-attachments/assets/aa8c1dee-e072-4c60-8258-2f6b5b04ef94" />

### - Thao tác dữ liệu dạng bytes trong Python (rất hay dùng khi viết exploit)
<img width="627" height="297" alt="image" src="https://github.com/user-attachments/assets/ec738481-1085-4fd6-af5c-2e42d552837a" />


