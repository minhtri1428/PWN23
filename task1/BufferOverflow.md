# Buffer Overflow
- Buffer Overflow xảy ra khi chương trình ghi dữ liệu vào một vùng nhớ vượt quá kích thước đã cấp phát cho nó, khiến dữ liệu bị tràn sang vùng nhớ liền kề.

Nguyên nhân :
- Ngôn ngữ C/C++ không tự dộng kiểm tra biên khi ghi vào mảng hoặc buffer
- Dữ liệu người dùng và dữ liệu điều khiển chương trình như (biến cục bộ khác, saved RBP, return address) nằm cạnh nhau trên cùng một stack frame, nên ghi tràn dữ liệu có thể làm hỏng luôn phần điều khiển.
- Các hàm C gây lỗi : gets(), strcpy(), strcat(), sprintf(), scanf("%s"). Trong đó, gets() nguy hiểm nhất vì k có cách nào để giới hạn độ dài đọc vào của hàm.
- Buffer được ghi từ địa chỉ thấp lên cao. Nếu ghi vượt quá kích thước buffer, dữ liệu tràn sẽ lần lượt đè lên: biến cục bộ khác → canary (nếu có) → saved RBP → return address → con trỏ hàm/GOT, mức độ nguy hiểm  cũng tăng dần theo thứ tự.
