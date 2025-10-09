Mô tả ngắn

Vinicius v3 là một obfuscator/encoder Python mạnh mẽ, thiết kế để che giấu mã nguồn bằng nhiều tầng nén, mã hóa tùy chỉnh và các cơ chế bảo vệ thời chạy (anti-debug, anti-tamper, hide builtins, junk code). Công cụ hướng tới bảo vệ mã chạy trên môi trường Python và gây khó khăn cho việc reverse-engineering.

Tính năng chính

🔐 Multi-layer protection: nhiều tầng mã hóa/nén (marshal, base85/b85, bz2, zlib, lzma).

🛡️ Anti-Debug & Anti-Crack: kiểm tra debugger, kiểm tra kích thước / nội dung file, so sánh hằng cấu hình.

🎭 Hide builtins: thay thế trực tiếp các gọi tới builtins bằng getattr(builtins, "...").

🗑️ Junk code injection: chèn các đoạn AST “vô nghĩa” để làm rối code.

✨ F-string → join conversion: chuyển f-string thành join/calls để khó đọc.

🎨 CLI đẹp: sử dụng pystyle để có giao diện CLI màu sắc / banner.

🚀 Xuất file: tạo file obfuscated với tiền tố viniciusv3-<yourfile>.

Yêu cầu

Python 3.6+ (đoạn mã kiểm tra version khi khởi tạo).

pystyle (script auto-cài nếu thiếu).

file nguồn mã cần obfuscate phải ở mã hoá UTF-8.

Cách cài & chạy nhanh
# clone hoặc copy file viniciusv3.py vào thư mục chứa mã nguồn cần obfuscate
python viniciusv3.py


Ví dụ tương tác CLI (theo luồng trong code):

Nhập tên file cần encode: File Name: myscript.py

Tùy chọn bật/tắt:

AN TI - CRACK [y/n] (mặc định Y)

AN TI - DEBUG [y/n] (mặc định Y)

Do you want to hide the original content entirely? (y/n) (hide builtins)

Do you want to add some junk code mixing? (y/n)

Sau khi chạy xong, file đã mã hóa sẽ được lưu dưới dạng:

viniciusv3-myscript.py

Kiến trúc & cấu trúc chính (giải thích từ mã nguồn)

Trích yếu cấu trúc tên lớp/hàm chính dựa trên mã viniciusv3.py. 

viniciusv3

Biến/Module cấu hình

__VINICIUSV3__ — hằng cấu hình (tên, liên hệ, thông điệp). Nếu bị sửa đổi, mã có cơ chế từ chối chạy (anti-tamper).

BANNER — banner ASCII hiển thị khi chạy.

Lớp & chức năng chính

Evisulove — khởi tạo môi trường runtime, kiểm tra phiên bản Python, và chuẩn bị các global helper (ví dụ evin_add, meo, vetages, ...). Đây là lớp “bootstrap”.

Evisu — chịu trách nhiệm giải nén / decode chuỗi đã mã hóa (phương thức scam() trong mã). Lần lượt chạy qua các bước giải mã nén/encode đã áp dụng.

__Ngnhatnamanh_ — loader cuối cùng: dùng marshal.loads để nạp bytecode và exec chúng trong globals.

Biến/func trợ giúp: enc(), ngon_ba_zo() (tạo tên ngẫu nhiên), obfstr(), obfint() (chuyển hằng string/int sang biểu thức AST nhằm che dấu).

Trình biến đổi AST

cv — chuyển đổi f-string (JoinedStr) sang chuỗi join/calls.

hide — thay thế các gọi trực tiếp tới builtins bằng getattr(builtins, "...").

obf — thay thế hằng (strings/ints) bằng các biểu thức đánh lừa.

junk — chèn các cấu trúc rác (gen_jcode) vào module / function / class bodies.

Quy trình obfuscation (tóm tắt bước)

Đọc file nguồn và prefix bằng anti (đoạn code kiểm tra runtime ban đầu).

AST transform:

chuyển f-strings,

che builtins (nếu chọn),

obfuscate hằng,

chèn junk code (nếu chọn).

compile(ast.unparse(code), '<Viniciusv3>', 'exec') → marshal.dumps(...).

Lặp nhiều lần:

marshal.dumps → base64.b85encode(bz2.compress(zlib.compress(...))) (vòng lặp mã hóa lồng).

Kết thúc bằng: base64.a85encode(bz2.compress(zlib.compress(lzma.compress(code)))).

Ghi ra file viniciusv3-<original> chứa template loader (SANH) + byte string đã encode.

Điều này làm cho quá trình reverse rất khó: mã thực thi cuối cùng cần lược ngược toàn bộ các bước giải nén và marshal.loads nhiều tầng.

Tầng lớp bảo mật & cơ chế chống đảo ngược

Multi-compression & multiple layers marshal

Dữ liệu bytecode bị marshal.dumps nhiều lần, sau đó nén bằng zlib/bz2/lzma và mã hóa với base85/ASCII85 → tốn công để lột từng lớp.

Anti-Tamper (file integrity)

Kiểm tra số dòng file (open(__file__, 'rb').read().splitlines()) so với giá trị kỳ vọng; nếu khác, script sẽ từ chối chạy.

So sánh __VINICIUSV3__ với giá trị gốc; nếu thay đổi, thoát.

Anti-Debug

Kiểm tra sys.gettrace(); nếu có trace (debugger) thì dừng chương trình.

Kiểm tra tính nguyên thủy của một số builtin (str(print) …) để phát hiện hook.

Hide builtins

Khi bật, mọi truy cập tới builtins phổ biến (print, len, exec, …) được chuyển thành getattr(ev in_add('builtins'), "print") — khiến static analysis khó khôi phục.

Junk Code Injection

Chèn AST rối rắm (lambdas, vòng while vô hạn, phép chia 0 trong try/except, biểu thức khó đọc) để tốn công khi phân tích tĩnh.

Tên biến/func động & mapping tùy chỉnh

Tên biến, hàm, hằng được đổi thành tên Hàn/Unicode/chuỗi ngẫu nhiên để giảm khả năng đọc tên gốc.

Custom encoding map

Mã hóa các chuỗi hex vào một mapping unicode/emoji riêng (hàm enc() và bộ string/cust), làm cho các chuỗi nhạy cảm khó đọc.

Các tuỳ chọn khuyến nghị

Đối với production protection: bật anti_crack, anti_debug, hide_builtins, junk_code.

Lưu ý: quá nhiều lớp bảo vệ có thể làm khó debug khi bản thân bạn cần sửa lỗi — giữ một bản gốc không obfuscated để phát triển.

Cách hoạt động khi chạy file đã obfuscate

File viniciusv3-<file> chứa một loader (chuỗi SANH) mô tả:

Khởi tạo bootstrap (Evisulove) để đặt các helper.

Dùng Evisu(...).scam() để giải mã bytecode.

Dùng marshal.loads rồi exec để chạy bytecode gốc.

Nếu muốn debug hoặc hiểu luồng giải mã, bạn có thể:

Dời bước anti/chặn check file để bỏ một lớp anti-debug (chỉ dùng trên bản local).

Thêm print() trong scam() để in kích thước/kiểu dữ liệu qua mỗi bước giải nén.

Cẩn trọng: sửa loader có thể khiến script từ chối chạy nếu anti-tamper vẫn còn.

Lỗi thường gặp & khắc phục

Module pystyle thiếu: script cố gắng tự cài (pip install pystyle<ver>). Nếu không thể, cài thủ công: pip install pystyle.

Phiên bản Python không khớp: thông báo và thoát — chạy đúng Python 3.6+ (mã có kiểm tra sys.version_info).

Vấn đề encoding file: đảm bảo file nguồn là UTF-8.

Kết quả obfuscated không chạy: khả năng do anti-tamper yêu cầu số dòng cố định — kiểm tra các tùy chọn anti_crack/anti_debug.

Bảo mật & Trách nhiệm

Công cụ cung cấp lớp bảo mật để bảo vệ bản quyền, tuy nhiên:

Không dùng để che giấu mã độc hoặc hành vi trái pháp luật.

Việc obfuscation không thay thế bảo vệ pháp lý (copyright, license).

Người dùng chịu trách nhiệm về việc dùng công cụ phù hợp với pháp luật địa phương.

Liên hệ & thông tin tác giả

Thông tin trong file:

__ADMIN_: Cte Vcl

Liên hệ: https://t.me/ctevclwar / https://www.facebook.com/ng.xau.k25
(Chi tiết lấy từ hằng __VINICIUSV3__ trong mã). 

viniciusv3

Mẫu README ngắn (copy / paste)
# VINICIUS v3 — High Speed Obfuscator
Một trình obfuscator Python multi-layer (marshal, base85, bz2, zlib, lzma) kèm anti-debug/anti-tamper, hide builtins, và junk code injection.

## Cài đặt
python viniciusv3.py

## Sử dụng
Nhập tên file, chọn tuỳ chọn anti/junk/hide. File output: viniciusv3-<tên file>.

## Lưu ý
Giữ bản gốc chưa obfuscate để sửa lỗi. Sử dụng có trách nhiệm.
