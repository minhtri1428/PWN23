# Buffer Overflow
- Buffer Overflow xảy ra khi chương trình ghi dữ liệu vào một vùng nhớ vượt quá kích thước đã cấp phát cho nó, khiến dữ liệu bị tràn sang vùng nhớ liền kề.

   <img width="627" height="143" alt="image" src="https://github.com/user-attachments/assets/3ee08fd9-c6cb-438d-b5bf-1db601475b8a" />
- Buffer được ghi từ địa chỉ thấp lên cao. Nếu ghi vượt quá kích thước buffer, dữ liệu tràn sẽ lần lượt đè lên: biến cục bộ khác → canary (nếu có) → saved RBP → return address → con trỏ hàm/GOT, mức độ nguy hiểm  cũng tăng dần theo thứ tự.

Nguyên nhân :
- Ngôn ngữ C/C++ không tự dộng kiểm tra biên khi ghi vào mảng hoặc buffer
- Dữ liệu người dùng và dữ liệu điều khiển chương trình như (biến cục bộ khác, saved RBP, return address) nằm cạnh nhau trên cùng một stack frame, nên ghi tràn dữ liệu có thể làm hỏng luôn phần điều khiển.
- Các hàm C gây lỗi : gets(), strcpy(), strcat(), sprintf(), scanf("%s"). Trong đó, gets() nguy hiểm nhất vì k có cách nào để giới hạn độ dài đọc vào của hàm.
# Ví dụ (video của anh Trí) - bof1
<img width="607" height="486" alt="image" src="https://github.com/user-attachments/assets/d7bc5506-ac74-4b7b-bc85-63f751d2a1f4" />

- Khi đọc pseudo-code của file bof1 ta thấy đc chương trình sẽ cho ta quyền điều khiển shell nếu thỏa mãn điều kiện các biến v5 v6 v7 !=0, nhưng chương trình chỉ cho phép nhập dữ liệu vào biến buf. Ở đây ta thấy đc cả biến buf và v5 v6 v7 chỉ chứa đc 40 byte , nhưng hàm read lại có thể đọc đc tối đa 48 byte => lỗ hổng có thể khai thác được
  <img width="887" height="246" alt="image" src="https://github.com/user-attachments/assets/b7cabf3c-9736-480d-937b-de7e82b1ab42" />

- Sau khi nhập vào 40 byte, dữ liệu bị tràn và ghi đè lên biến a b c và ta có được quyền điều khiển shell
# bof2 - Pwntools
<img width="717" height="481" alt="image" src="https://github.com/user-attachments/assets/c1b5eeb2-af52-4f31-983c-324fb8fdf1c1" />


- Tương tự như bof1, mục tiêu của bài này cũng là ghi đè lên các biến để chiếm shell. Nhưng để chiếm đc shell từ chương trình, giá trị ghi đè phải thỏa mãn điều kiện của hàm .
 <img width="968" height="467" alt="image" src="https://github.com/user-attachments/assets/8f299704-c7ff-4193-ac2a-82475c54e314" />


- Vì điều kiện của hàm là các giá trị k thể nhập đc từ bàn phím, nên ta sẽ sử dụng công cụ pwntools để gửi dữ liệu trực tiếp vào chương trình thông qua script.
  <img width="898" height="240" alt="image" src="https://github.com/user-attachments/assets/d0f62a47-cf30-4abf-93c1-3ba0a475bb16" />
   
- Sau khi chạy script thì ta đã có được shell
- 
