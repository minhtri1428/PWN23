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
-fno-stack-protector: Tắt Canary.
-z execstack: Bật quyền thực thi trên stack 
-no-pie: Tắt PIE 
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
