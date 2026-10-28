# PIE (Position Independent Executable
- PIE là thuộc tính của file binary đc biên dịch dưới dạng mã ko phụ thuộc vị trí, nên có thể đc nạp một cách ngẫu nhiên vào bộ nhớ qua bất kì address nào
- ASLR (Address Space Layout Randomization) là cơ chế bảo mật của hệ điều hành, xếp ngẫu nhiên vị trí các vùng nhớ của chương trình mỗi lần chạy. Giúp chống lại việc khai thác Stack, Heap, Libraries (libc) ở các vị trí cố định.

