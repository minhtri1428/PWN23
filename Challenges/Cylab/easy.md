# PIE TIME
- Khi checksec file vuln mà challenge cung cấp ta thấy được Pie đang bật có nghĩa là địa chỉ base của hàm main sẽ được thay đổi sau mỗi lần chạy
  <img width="376" height="182" alt="image" src="https://github.com/user-attachments/assets/32af3a41-a4ad-4cd6-bc8d-7d38edc4a33d" />
  
- Nhìn vào source C ta thấy đc khi chạy chương trình sẽ in ra địa chỉ hiện tại của main 
 <img width="641" height="342" alt="image" src="https://github.com/user-attachments/assets/84ccf709-1f19-4392-8b66-e9035920f068" />

- Hàm ta cần gọi là hàm win vì in nó ra flag
<img width="446" height="581" alt="image" src="https://github.com/user-attachments/assets/04d1ba9d-4e7f-450f-a7ca-6c243ffee6eb" />

- Khi PIE bật thì địa chỉ base của hàm main sẽ đc thay đổi sau mỗi lần chạy , nhưng offset giữa hàm main và win thì không đổi. Vậy nên ta chỉ cần tìm offset giữa 2 hàm sau đó lấy địa chỉ leak của hàm main - offset = địa chỉ hàm win
 <img width="1110" height="141" alt="image" src="https://github.com/user-attachments/assets/e857f6fe-fd4c-4baf-be80-4ad60c0c8a09" />

- Sau khi kiểm tra , ta thấy đc địa chỉ của hàm main là 0x134c và  hàm win là 0x12aa. Như vậy, offset sẽ bằng 162 byte
  <img width="246" height="47" alt="image" src="https://github.com/user-attachments/assets/0bf05219-2953-450f-b218-a741d0b8acd8" />

- Khi chạy chương trình, màn hình in ra địa chỉ của hàm main là 0x5e51b65d534c , sau đó ta lấy địa chỉ main - offset = 0x5e51b65d52aa
  <img width="942" height="141" alt="image" src="https://github.com/user-attachments/assets/9715f435-ce6a-4235-9785-a2f20b2c4db7" />

  và ta có đc flag
