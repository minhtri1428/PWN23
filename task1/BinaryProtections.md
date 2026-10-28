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

