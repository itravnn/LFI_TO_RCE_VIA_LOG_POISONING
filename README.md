# LFI_TO_RCE_VIA_LOG_POISONING
## What is a file inclusion vulnerability ?
Nguyên nhân: do xác thực đầu vào kém. Trong code php web, các lập trình viên thường sử dụng các lệnh include, require, include_once, require _ once , các lệnh này cho phép việc file hiện tại có thể gọi ra 1 file khác.

Dấu hiệu để nhận biết rằng trang web có thể tấn công file inclusion là đường link thường có dạng php?page=,hoặc php?file= .... 

Kết quả: cho phép attacker truy cập trái phép vào những tập tin nhạy cảm trên máy chủ web hoặc thực thi các tệp tin độc hại

File Inclusion có thể dẫn đến các cuộc tấn công sau :

- Code execution on the web server

- Cross Site Scripting Attacks (XSS)

- Denial of service (DOS)

- Data Manipulation Attacks

## What is LFI to RCE ?
Local file inclustion (LFI) là lỗi cơ bản của các trang web, thông thường là do lỗi cấu hình che dấu file và sai sót khi filter input form , cho phép bạn di chuyển vào thư mục và đọc tệp (path traversal). Nó có thể cung cấp các thông tin nhảy cảm như là passwd, php.ini, access_log, config.php…, đôi khi có thể được sử dụng dể Remote Code Execution (RCE)

Remote Code Execution (RCE) nghĩ là thực thi code từ xa, Tức là bạn có thể thông qua một kĩ thuật nào đó để có thể đạt được quyền điều khiển trên máy nạn nhân, thông qua đó có thể thực thi bash, shell...hoặc code của một vài ngôn ngữ kịch bản như python, perl, javascript...
