# writeup-task0
# >  #                         COMPUTER STRUCTURE

---

## * CPU(Central Processing Unit)
   - CPU được coi là bộ phận quan trọng nhất của máy tính , điều khiển mọi hoạt động của hệ thống 
   - Các thành phần bên trong CPU bao gồm:ALU ,CU , và Register 
   - ALU (Arithmetic Logic Unit) thực hiện các phép tính toán học (+, -, ×, /) và logic (AND, OR, so sánh...)
   - CU (Control Unit) điều phối, "chỉ huy" các bộ phận khác hoạt động đúng trình tự
- Registers(Thanh ghi) vùng nhớ siêu nhỏ, siêu nhanh để lưu tạm dữ liệu đang xử lý
## * Bộ nhớ
- Register — Là phần load và xử lý data với tốc độ gần như tức thời, nhỏ nhất, nằm trong CPU, dung lượng cực nhỏ chỉ vài chục đến vài trăm byte 
- Cache (L1, L2, L3) — Bộ nhớ đệm tốc độ cao, nằm giữa CPU và RAM, giảm thời gian chờ dữ liệu. (Cache miss làm cho cpu phải lấy dữ liệu từ ram => làm chương trình chậm đi rất nhiều )
-   RAM (Random Access Memory) — Bộ nhớ chính của hệ thống, lưu trữ dữ liệu và chương trình đang chạy, độ trễ chậm hơn cache vài chục lần 
- Storage (SSD/HDD) — là nơi cho phép lưu trữ dữ liệu nhiều nhất, lâu nhất nhưng cũng là chậm nhất 
![image](https://hackmd.io/_uploads/HJ-MhNSYGg.png)
=>> Dung lượng bộ nhớ sẽ tỉ lệ nghịch với tốc độ xử lý, tức là sức chứa bộ nhớ càng lớn thì tốc độ xử lý càng chậm










#  Assembly (ASM)
---
- Assembly language (hợp ngữ) là ngôn ngữ lập trình cấp thấp, gần với ngôn ngữ máy nhất mà con người còn đọc hiểu được. 
- Mỗi dòng lệnh assembly gần như tương ứng trực tiếp với 1 lệnh mà CPU thực thi.
- Mỗi kiến trúc CPU có tập lệnh assembly riêng (x86-64 khác ARM, khác RISC-V...) không portable như C
- Làm việc trực tiếp với thanh ghi (registers) và bộ nhớ
- Cần assembler (VD: NASM, MASM, GAS) để dịch sang mã máy (không phải compiler như C/C++)
- Một dòng code C/C++ có thể viết thành nhiều dòng dưới dạng assembly
