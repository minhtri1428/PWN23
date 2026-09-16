💻 1. Kiến Trúc Máy Tính Tổng Quát🏗️ Mô Hình Von NeumannPlaintext               ┌─────────────────────────────────────┐
               │                 CPU                 │
               │  ┌───────────────────────────────┐  │
               │  │ Control Unit (CU)             │  │
               │  ├───────────────────────────────┤  │
               │  │ Arithmetic Logic Unit (ALU)   │  │
               │  ├───────────────────────────────┤  │
               │  │ Registers                     │  │
               │  └───────────────────────────────┘  │
               └──────────────────┬──────────────────┘
                                  │ Bus (Địa chỉ / Dữ liệu / Điều khiển)
       ┌──────────────────────────┼──────────────────────────┐
       │                          │                          │
┌──────▼──────┐            ┌──────▼──────┐            ┌──────▼──────┐
│   Memory    │            │     I/O     │            │   Storage   │
│   (RAM)     │            │  (Bàn phím, │            │   (Ổ đĩa)   │
│             │            │  màn hình)  │            │             │
└─────────────┘            └─────────────┘            └─────────────┘
Ý tưởng cốt lõi của von Neumann: code (chương trình) và data (dữ liệu) được lưu chung trong một vùng nhớ, CPU đọc lệnh từ bộ nhớ, giải mã, rồi thực thi — đây là lý do vì sao các lỗi như buffer overflow có thể nguy hiểm: dữ liệu và lệnh nằm cùng một không gian địa chỉ, nên ghi đè dữ liệu có thể ảnh hưởng tới luồng thực thi.Các thành phần chính của CPUControl Unit (CU): điều phối, giải mã lệnh, quyết định lệnh tiếp theo cần lấy từ đâu (dựa vào rip).ALU (Arithmetic Logic Unit): thực hiện các phép toán số học (cộng, trừ...) và logic (AND, OR, XOR, so sánh...).Registers: bộ nhớ siêu nhanh ngay trong CPU, dùng để chứa toán hạng, kết quả tạm, con trỏ.Cache (L1/L2/L3): bộ nhớ đệm giữa CPU và RAM, giúp tăng tốc truy cập dữ liệu hay dùng lại.Chu trình Fetch–Decode–ExecuteMọi lệnh máy đều được CPU xử lý qua 3 bước lặp lại liên tục:Fetch: lấy lệnh tại địa chỉ rip đang trỏ tới, từ bộ nhớ.Decode: giải mã xem đây là lệnh gì (mov, add, jmp...) và toán hạng là gì.Execute: thực thi lệnh đó (qua ALU nếu là phép toán, qua CU nếu là điều khiển luồng...), rồi cập nhật rip trỏ tới lệnh tiếp theo (hoặc nhảy tới địa chỉ khác nếu là lệnh jmp/call/ret).Memory Hierarchy (phân cấp bộ nhớ)Từ nhanh/nhỏ/đắt nhất đến chậm/lớn/rẻ nhất:PlaintextRegisters  →  Cache (L1/L2/L3)  →  RAM  →  Disk (SSD/HDD)
(nhanh nhất,                               (chậm nhất,
 dung lượng nhỏ nhất)                      dung lượng lớn nhất)
Hiểu phân cấp này giúp hiểu vì sao truy cập register/stack (đã nằm gần CPU, thường trong cache) lại nhanh hơn nhiều so với truy cập heap lớn hay đọc file trên đĩa.32-bit vs 64-bitCon số "32-bit" hay "64-bit" thường chỉ độ rộng thanh ghi và độ rộng bus địa chỉ của CPU.CPU 64-bit có thể địa chỉ hóa không gian nhớ lớn hơn rất nhiều (2^64 so với 2^32), và các thanh ghi đa dụng rộng 64-bit thay vì 32-bit.Trên Linux, các binary 32-bit (x86) và 64-bit (x86-64) có ABI và bộ thanh ghi khác nhau — cần chú ý khi phân tích bằng file hoặc readelf -h để biết đang làm việc với kiến trúc nào.💻 2. Kiến trúc x86-64Tổng quan kiến trúc x86-64 (AMD64):CPU thực thi lệnh máy (machine code) tuần tự, lệnh được nạp từ vùng nhớ text (code segment).Dữ liệu được thao tác qua các thanh ghi (registers) — bộ nhớ siêu nhanh nằm ngay trong CPU.Có 16 thanh ghi đa dụng 64-bit: rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8, r9, r10, r11, r12, r13, r14, r15.Ngoài ra còn rip (instruction pointer — trỏ tới lệnh tiếp theo sẽ thực thi) và rflags (cờ trạng thái: zero flag, carry flag, sign flag...).Cách chia nhỏ một thanh ghi (ví dụ với rax)TênĐộ rộngÝ nghĩarax64-bitToàn bộ thanh ghieax32-bit32 bit thấp của raxax16-bit16 bit thấp của eaxah8-bitByte cao của axal8-bitByte thấp của axLưu ý quan trọng: khi ghi vào eax (32-bit), CPU tự động zero-extend — xóa toàn bộ 32 bit cao của rax. Nhưng khi ghi vào ax hoặc al, phần cao hơn giữ nguyên (không bị xóa).Vai trò thường dùng của một số thanh ghirax: thường chứa giá trị trả về của hàm, và trong syscall là số hiệu syscall.rdi, rsi, rdx, rcx, r8, r9: chứa 6 tham số đầu tiên khi gọi hàm (System V ABI).rbp: base pointer — trỏ tới đáy của stack frame hiện tại.rsp: stack pointer — luôn trỏ tới đỉnh (top) của stack.rip: instruction pointer — đây là mục tiêu chính khi khai thác lỗi (control rip = kiểm soát luồng thực thi chương trình).💻 3. Calling Convention (System V AMD64 ABI — dùng trên Linux)Quy ước gọi hàm (call):Tham số truyền theo thứ tự: rdi, rsi, rdx, rcx, r8, r9. Nếu nhiều hơn 6 tham số, các tham số dư được đẩy lên stack.Giá trị trả về nằm ở rax (nếu là số nguyên/con trỏ).Khi gọi hàm bằng lệnh call, địa chỉ trở về (return address) — tức địa chỉ lệnh ngay sau call — được tự động đẩy vào stack.Hàm kết thúc bằng ret, lệnh này pop giá trị trên đỉnh stack ra và nhảy (jmp) tới đó — đây chính là cơ chế mà buffer overflow lợi dụng để chiếm quyền điều khiển rip.Bảo toàn thanh ghi (Callee-saved vs Caller-saved):Callee-saved (hàm được gọi phải bảo toàn giá trị): rbx, rbp, r12, r13, r14, r15.Caller-saved (hàm gọi phải tự lưu nếu cần dùng lại sau khi gọi): rax, rcx, rdx, rsi, rdi, r8-r11.Ví dụ prologue/epilogue của một hàm điển hìnhĐoạn mãpush rbp         ; lưu rbp cũ của caller
mov  rbp, rsp    ; thiết lập rbp mới = đỉnh stack hiện tại
sub  rsp, 0x20   ; cấp phát local variables (32 byte)
; ... thân hàm ...
leave            ; tương đương: mov rsp, rbp; pop rbp
ret              ; pop return address vào rip
💻 4. Memory Layout của một chương trìnhTổng quanVirtual Address Space (thấp → cao):PlaintextĐịa chỉ cao  ┌─────────────────────┐
             │        Stack        │  ← local variables, return address
             │          ↓          │     (mọc xuống địa chỉ thấp)
             │                     │
             │  (khoảng trống)     │
             │                     │
             │          ↑          │
             │        Heap         │  ← malloc/new cấp phát
             ├─────────────────────┤
             │   BSS (biến chưa    │  ← global/static chưa khởi tạo
             │   khởi tạo = 0)     │
             ├─────────────────────┤
             │   Data (biến đã     │  ← global/static đã khởi tạo
             │   khởi tạo)         │
             ├─────────────────────┤
Địa chỉ thấp │   Text (code)       │  ← mã máy của chương trình (read-only)
             └─────────────────────┘
Chi tiết từng vùng nhớ:Text: chứa mã máy, thường chỉ có quyền đọc + thực thi (r-x), không ghi được.Data/BSS: biến toàn cục và static.Heap: cấp phát động (malloc, new), mọc từ địa chỉ thấp lên cao.Stack: nơi lưu local variable, tham số, return address; mọc từ địa chỉ cao xuống thấp.Giữa heap và stack có một vùng trống lớn, và còn có thêm vùng cho shared libraries (libc.so...) thường nằm gần đỉnh không gian địa chỉ hoặc ở vị trí ngẫu nhiên nếu bật ASLR (Address Space Layout Randomization).Hiểu memory layout là nền tảng bắt buộc để hiểu các lỗi như buffer overflow (ghi đè dữ liệu trên stack), heap overflow, use-after-free...💻 5. Bit, Byte và EndiannessCác khái niệm cơ bản:Bit: đơn vị nhỏ nhất, giá trị 0 hoặc 1.Byte: 8 bit, biểu diễn được giá trị từ 0–255 (0x00–0xFF).MSB (Most Significant Bit/Byte): bit/byte có trọng số lớn nhất.LSB (Least Significant Bit/Byte): bit/byte có trọng số nhỏ nhất.Endianness — cách sắp xếp byte trong bộ nhớ (Giả sử có giá trị 4-byte: 0x12345678)Little-endian (x86/x86-64 dùng cái này): byte có trọng số thấp nhất được lưu ở địa chỉ thấp nhất.PlaintextĐịa chỉ:   0x00  0x01  0x02  0x03
Giá trị:   0x78  0x56  0x34  0x12
Big-endian (dùng trong một số kiến trúc mạng, network byte order): byte có trọng số cao nhất lưu ở địa chỉ thấp nhất.PlaintextĐịa chỉ:   0x00  0x01  0x02  0x03
Giá trị:   0x12  0x34  0x56  0x78
Mẹo nhớ: x86 = little-endian → khi dump memory bằng gdb/xxd, nếu thấy chuỗi byte trông "ngược", đó là bình thường — cần đảo ngược lại để đọc ra giá trị số thật.Ví dụ thực hành: chuỗi byte 41 41 41 41 42 42 42 42 trên stack (little-endian), nếu đọc thành 2 giá trị 4-byte, ta được 0x41414141 và 0x42424242 — đây là kiểu dữ liệu rất hay gặp khi debug buffer overflow (do 'A' = 0x41, 'B' = 0x42).💻 6. StackTổng quan: Stack là vùng nhớ hoạt động theo cơ chế LIFO (Last In, First Out).Lệnh Push và Pop:push <giá trị>: Giảm rsp đi 8 (vì mỗi lần push/pop trên x86-64 làm việc với 8 byte), ghi giá trị vào địa chỉ [rsp] mới.pop <thanh ghi>: Đọc giá trị tại [rsp], tăng rsp lên 8.PlaintextTrước push:         Sau push rax (rax = 0x41):
rsp → [ ... ]        [ ... ]
                     rsp → [ 0x41 ]   ← rsp giảm, giá trị mới nằm đây
Stack frame của một hàm:Mỗi khi một hàm được gọi, một "khung" (frame) mới được tạo trên stack, thường chứa:Return address (do call tự động push)Saved rbp của caller (do push rbp trong prologue)Local variables (cấp phát bằng sub rsp, N)Đôi khi có thêm buffer canary (stack protector) để chống overflowPlaintextĐịa chỉ cao
┌───────────────────┐
│  Return address   │  ← do lệnh `call` push vào
├───────────────────┤
│  Saved rbp (old)  │  ← do prologue push rbp
├───────────────────┤ ← rbp trỏ vào đây
│  Local variable 1 │
├───────────────────┤
│  Local variable 2 │
├───────────────────┤ ← rsp trỏ vào đây (đỉnh stack hiện tại)
Địa chỉ thấp
Mối liên hệ giữa Stack, Memory và Register:rsp và rbp là thanh ghi nhưng giá trị của chúng là địa chỉ bộ nhớ trỏ vào vùng stack.Khi một buffer local (ví dụ char buf[16]) bị ghi tràn (overflow) mà không kiểm tra độ dài, dữ liệu ghi thừa sẽ đè lên saved rbp, rồi đè lên return address. Nếu kẻ tấn công kiểm soát được return address, họ có thể điều khiển rip sau khi hàm ret — đây chính là ý tưởng cốt lõi của stack buffer overflow.💻 7. Hex và BinaryBảng quy đổi nhanh:DecimalBinaryHex000000x0501010x51010100xA1511110xF255111111110xFFCách đổi Hex ↔ Binary nhanh:Mỗi ký tự hex tương ứng chính xác 4 bit → ghép trực tiếp, không cần qua decimal.Ví dụ: 0xA3 → A = 1010, 3 = 0011 → 10100011.Công cụ thực hành trên Linux:python3 -c "print(hex(1234))" # decimal -> hexpython3 -c "print(int('1F', 16))" # hex -> decimalpython3 -c "print(bin(200))" # decimal -> binaryecho -n "AAAA" | xxd # xem giá trị hex/ascii của chuỗiprintf '%x\n' 255 # in ra hexThao tác dữ liệu dạng bytes trong Python (viết exploit):Pythonimport struct

# Đóng gói số nguyên thành little-endian 4 byte
data = struct.pack("<I", 0x41424344)   # b'DCBA'

# Ngược lại: unpack
value = struct.unpack("<I", data)[0]

# pwntools cung cấp sẵn hàm tiện lợi hơn
from pwn import p32, p64, u32, u64
p64(0x4141414141414141)   # pack 8 byte little-endian
💻 8. Linux Cơ Bản Cần BiếtDanh sách lệnh thông dụng:LệnhCông dụngls -laLiệt kê file, gồm cả file ẩn và permissioncd, pwdDi chuyển và xem thư mục hiện tạicat, lessXem nội dung filechmod +x fileCấp quyền thực thi cho filefile <binary>Xác định loại file (ELF 64-bit, PIE, stripped...)strings <binary>In ra các chuỗi ký tự in được trong file — hữu ích để tìm hintobjdump -d -M intel binaryDisassemble file, xem mã Assembly (cú pháp Intel)readelf -h/-S binaryXem header, section của file ELFgdb ./binaryDebugger — công cụ trung tâm để phân tích/khai thác binarychecksec ./binary(cần cài pwntools) Kiểm tra các cơ chế bảo vệ: NX, PIE, Canary, RELROltrace, straceTheo dõi lời gọi hàm thư viện / syscall của chương trình khi chạyMột số lệnh GDB hay dùng khi mới bắt đầu:break main # đặt breakpoint tại hàm mainrun # chạy chương trìnhinfo registers # xem giá trị toàn bộ thanh ghix/20xg $rsp # xem 20 giá trị 8-byte (g=giant) tại $rsp dạng hexdisassemble main # xem assembly của hàm mainstepi / nexti # chạy từng lệnh assembly một💻 9. Assembly Cơ Bản (x86-64, Cú Pháp Intel)Cấu hình hiển thị cú pháp Intel:Trong gdb: set disassembly-flavor intelVới objdump: thêm cờ -M intelCác lệnh (instruction) cơ bản:LệnhÝ nghĩamov dst, srcCopy giá trị từ src vào dstpush src / pop dstĐẩy vào / lấy ra khỏi stackadd, subCộng, trừcmp a, bSo sánh a và b (thực chất là a - b, chỉ set flag)test a, bAND bit-wise giữa a và b, chỉ set flagje, jne, jg, jlNhảy có điều kiện (equal, not equal, greater, less) dựa theo flag sau cmp/testjmp addrNhảy không điều kiệncall addrGọi hàm: push return address, nhảy tới addrretPop giá trị từ stack vào rip, quay về callerlea dst, [addr]Load địa chỉ (không phải giá trị) vào dstsyscallGọi system call của kernel (Linux x86-64)Ví dụ đoạn Assembly đơn giản (Intel syntax):Đoạn mãmov eax, 5        ; eax = 5
add eax, 3        ; eax = eax + 3 = 8
cmp eax, 8        ; so sánh eax với 8 -> set ZF=1 vì bằng nhau
je  equal_label   ; nếu bằng thì nhảy tới equal_label
Syscall trên Linux x86-64 — nền tảng để viết shellcode:Quy ước gọi syscall (khác với calling convention của hàm bình thường!):rax = số hiệu syscallTham số theo thứ tự: rdi, rsi, rdx, r10, r8, r9 (chú ý: dùng r10 thay vì rcx so với calling convention hàm thường)Lệnh syscall để gọi vào kernelVí dụ: gọi write(1, "hi", 2) bằng Assembly thô (syscall number của write là 1):Đoạn mãmov rax, 1        ; syscall number: write
mov rdi, 1        ; fd = 1 (stdout)
lea rsi, [msg]    ; địa chỉ buffer chứa dữ liệu cần in
mov rdx, 2        ; số byte cần ghi
syscall
