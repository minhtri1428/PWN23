# 1. WSL/Ubuntu
- Là tính năng cho phép người dùng chạy Linux ngay bên trong Windows mà k cần chạy máy ảo
- Vì binary thường là các file elf(định dạng thực thi của Linux) và các tool (gdb, pwndbg, pwntools, gcc...) đc xây dựng cho Linux nên WSL/Ubuntu để chạy được các tool này trên máy Windows.

Các lệnh cơ bản 
- cd <đường đãn thư mục>: di chuyển tới thư mục mong muốn , cd ~ về home
- ls liệt kê nội dung của thư mục, ls -la hiển thị chi tiết tất cả các file, kể cả file ẩn.
- cat <file> xem nội dung file text.
- file <tên file> kiểm tra định dạng.
- chmod +x <tên file> cấp quyền thực thi .
- strings <tên_file> trích xuất các chuỗi ký tự có trong file binary.
- grep "chuỗi" file : tìm chuỗi trong file cụ thể , grep -r "chuỗi" /home/user/.. tìm chuỗi trong mọi file thuộc /home/user/..
# 2. GCC
Lệnh cơ bản:
- gcc -g -o <tên_file_chạy> <tên_file.c>: Biên dịch và giữ thông tin debug (-g).

Tắt/bật bảo vệ 
- fno-stack-protector: Tắt Canary.
- z execstack: Bật quyền thực thi trên stack 
- no-pie: Tắt PIE 
# 3. Pwndbg 
- Là công cụ xem chương trình chạy thực tế dựa vào đó ta có thể DEBUG & phân tích động.

Lệnh cơ bản 
- start chạy 
- break (b) đặt breakpoint
- run (r) chạy chương trình
- continue (c) chạy tiếp
- next (n) / step (s) chạy qua 1 dòng code 
- ni/si chạy qua 1 lệnh assembly 
- disassemble (disas) xem assembly của hàm
- info registers (i r) xem thanh ghi
- checksec kiểm tra cơ chế bảo vệ (canary, NX, PIE, RELRO)
- vmmap xem bản đồ bộ nhớ
- cyclic/cyclic -l tạo pattern và tìm offset overflow
- telescope xem nội dung stack, tự resolve con trỏ
- search tìm chuỗi/byte trong bộ nhớ
- got/plt xem bảng GOT/PLT
# Pwntools 
- Là một framemwork Python chuyên dụng cho viết script exploit và tương tác với binary. Nó giúp đơn giản hóa các thao tác lặp đi lặp lại như kết nối, pack địa chỉ, tạo payload và tương tác với tiến trình.
- Cấu trúc cơ bản của một script exploit
<img width="1215" height="680" alt="image" src="https://github.com/user-attachments/assets/887d0df3-e0d8-41c8-a79e-78b5c45b0fd5" />

### Đóng gói dữ liệu (Packing)

Chuyển đổi số nguyên (địa chỉ) thành dạng byte để gửi vào chương trình:
- p64(value): Đóng gói thành 8 byte (dùng cho kiến trúc 64-bit).
- p32(value): Đóng gói thành 4 byte (dùng cho kiến trúc 32-bit).
- u64(data) / u32(data): Giải nén ngược từ byte ra số .

### Giao tiếp với Tiến trình (I/O)
Gửi dữ liệu:
- p.send(data): Gửi dữ liệu thô.
- p.sendline(data): Gửi dữ liệu kèm ký tự xuống dòng \n.
- p.sendafter(delim, data): Đợi đến khi nhận được delim rồi mới gửi data.

Nhận dữ liệu:
- p.recv(n): Nhận n byte.
- p.recvline(): Nhận đến hết dòng (gặp \n).
- p.recvuntil(delim): Nhận đến khi gặp chuỗi delim (ví dụ: p.recvuntil(b'name: ')).

Tương tác thủ công:
- p.interactive(): tương tác với chương trình.

Phân tích File ELF & Tìm Offset
- Đọc thông tin file: elf = ELF('./chall')
- Lấy địa chỉ hàm: elf.symbols['system'] hoặc elf.plt['puts'] .
- Lấy địa chỉ GOT: elf.got['puts'].

Tìm offset tràn bộ đệm:
- cyclic(200): Tạo một chuỗi pattern tuần hoàn (De Bruijn) để gửi vào chương trình.
- cyclic_find(0x6161616c): Tìm vị trí offset dựa trên giá trị bị crash trong thanh ghi RIP .

Các hàm khác
- asm('nop'): Chuyển mã assembly thành mã máy (shellcode).
- shellcraft.sh(): Tự động tạo shellcode để gọi /bin/sh .
- ROP(elf): Tự động tìm các gadget để xây dựng ROP chain .



