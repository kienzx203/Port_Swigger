# **JWT**

- JWT được sử dụng phổ biến nhất trong cơ chế xác thực, quản lý phiên và kiểm soát truy cập, những lỗ hổng này có khả năng gây tổn hại cho toàn bộ trang web và người dùng của nó.

![](./img_jwt/Screenshot%202023-06-28%20133013.png)

## **`JWTs là gì?`**

- JSON web tokens (JWTs)là một định dạng được tiêu chuẩn hóa để gửi dữ liệu JSON được ký bằng mật mã giữa các hệ thống. Về mặt lý thuyết, chúng có thể chứa bất kỳ loại dữ liệu nào, nhưng thường được sử dụng nhất để gửi thông tin ("yêu cầu") về người dùng như một phần của cơ chế xác thực, xử lý phiên và kiểm soát truy cập.

- Không giống như classic session tokens, tất cả dữ liệu mà máy chủ cần, được lưu trữ phía máy khách chính là JWT. Điều này làm cho JWT trở thành lựa chọn phổ biến cho các trang web có tính phân tán cao, nơi người dùng cần tương tác liền mạch với nhiều máy chủ back-end.

- `JWT có 3 phần:`
  - a header, a payload, and a signature

```
eyJraWQiOiI5MTM2ZGRiMy1jYjBhLTRhMTktYTA3ZS1lYWRmNWE0NGM4YjUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTY0ODAzNzE2NCwibmFtZSI6IkNhcmxvcyBNb250b3lhIiwic3ViIjoiY2FybG9zIiwicm9sZSI6ImJsb2dfYXV0aG9yIiwiZW1haWwiOiJjYXJsb3NAY2FybG9zLW1vbnRveWEubmV0IiwiaWF0IjoxNTE2MjM5MDIyfQ.SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP-Hw6GG1zfl4JyngsZReIfqRvIAEi5L4HV0q7_9qGhQZvy9ZdxEJbwTxRs_6Lb-fZTDpW6lKYNdMyjw45_alSCZ1fypsMWz_2mTpQzil0lOtps5Ei_z7mM7M8gCwe_AGpI53JxduQOaB5HkT5gVrv9cKu9CsW5MS6ZbqYXpGyOG5ehoxqm8DL5tFYaW3lB50ELxi0KsuTKEbD0t5BCl0aCR2MBJWAbN-xeLwEenaqBiwPVvKixYleeDQiBEIylFdNNIMviKRgXiYuAvMziVPbwSgkZVHeEdF5MQP1Oe2Spac-6IfA
```

![](./img_jwt/Screenshot%202023-06-28%20133659.png)

```
data = base64urlEncode( header ) + "." + base64urlEncode( payload )
signature = Hash( data, secret );
```

## **`JWT vs JWS vs JWE`**

- Trên thực tế, JWT không thực sự được sử dụng như một thực thể độc lập. Thông số JWT được mở rộng bởi cả thông số kỹ thuật Chữ ký Web JSON (JWS) và Mã hóa Web JSON (JWE), xác định các cách cụ thể để thực sự triển khai JWT.

![](./img_jwt/Screenshot%202023-06-28%20134406.png)

## **`Lỗi phát sinh JWT attack`**

- Những lỗi triển khai thường do chữ ký của JWT không được xác minh chính xác. Điều này cho phép kẻ tấn công giả mạo các giá trị được chuyển đến ứng dụng thông qua tải trọng của mã thông báo. Ngay cả khi chữ ký được xác minh mạnh mẽ, thì việc chữ ký đó có thực sự đáng tin cậy hay không phụ thuộc rất nhiều vào khóa bí mật của máy chủ vẫn là bí mật. Nếu khóa này bị rò rỉ theo một cách nào đó, hoặc có thể đoán được hoặc bruteforce, kẻ tấn công có thể tạo chữ ký hợp lệ cho bất kỳ mã thông báo tùy ý nào, làm ảnh hưởng đến toàn bộ cơ chế.

## **`Exploiting flawed JWT signature verification`**

>### `Accepting arbitrary signatures`

- Các thư viện JWT thường cung cấp một phương thức để xác minh mã thông báo và một phương thức khác chỉ giải mã chúng. Ví dụ: thư viện Node.js jsonwebtoken có verify() và decode().

- Đôi khi, các nhà phát triển nhầm lẫn hai phương thức này và chỉ chuyển các mã thông báo đến cho phương thức decode(). Điều này thực sự có nghĩa là ứng dụng hoàn toàn không xác minh chữ ký.

> `Lab: JWT authentication bypass via unverified signature`

- Ở trang web này do lỗi không xác minh signature của JWT. Chính vì vậy, kẻ tấn công có thể sửa payload và thực hiện dưới quyền admin.

- Sau khi tôi đăng nhập vào tài khoản chúng ta nhận được một mã cookie là JWT.

![](./img_jwt/Screenshot%202023-06-28%20140803.png)

- Sử dụng [JWT.io](https://jwt.io/)

![](./img_jwt/Screenshot%202023-06-28%20140946.png)

- Tôi đã thực hiện sửa payload và thực hiện dưới quyền admin.

![](./img_jwt/Screenshot%202023-06-28%20141414.png)

> ### `Accepting tokens with no signature`

- Trong số những thứ khác, tiêu đề JWT chứa tham số alg. Điều này cho máy chủ biết thuật toán nào đã được sử dụng để ký mã thông báo và do đó, thuật toán nào cần sử dụng khi xác minh chữ ký.

```
{
    "alg": "HS256",
    "typ": "JWT"
}
```

- JWT có thể được ký bằng nhiều thuật toán khác nhau, nhưng cũng có thể không được ký. Trong trường hợp này, tham số alg được đặt thành none, biểu thị cái gọi là "JWT không bảo mật". Do sự nguy hiểm rõ ràng của điều này, các máy chủ thường từ chối các mã thông báo không có chữ ký. Tuy nhiên, vì kiểu lọc này dựa vào phân tích cú pháp chuỗi, đôi khi bạn có thể bỏ qua các bộ lọc này bằng các kỹ thuật che giấu cổ điển, chẳng hạn như viết hoa hỗn hợp và mã hóa không mong muốn.

> `Lab: JWT authentication bypass via flawed signature verification`

- Ở trang web này sau khi đăng nhập tài khoản người dùng chúng ta có được session cookie là mã JWT. Tang web này có lỗi do no signature của JWT.

- Từ đấy, ta có thể sửa `alg : none` để chiếm quyền admin.

![](./img_jwt/Screenshot%202023-06-28%20143453.png)

![](./img_jwt/Screenshot%202023-06-28%20143605.png)

![](./img_jwt/Screenshot%202023-06-28%20143705.png)

> ### `Brute-forcing secret keys`

> `Lab: JWT authentication bypass via weak signing key`

- Sau khi nhận được JWT tôi thực hiện brute force bằng hashcat.

![](./img_jwt/Screenshot%202023-06-28%20144810.png)

- Tôi nhận được secretkey như sau:
![](./img_jwt/Screenshot%202023-06-28%20144922.png)

- Tôi thực hiện giả mạo một JWT mới dưới quyền admin từ secretkey mình tìm được.

![](./img_jwt/Screenshot%202023-06-28%20145054.png)

> ### `JWT header parameter injections`

- Theo đặc tả JWS, chỉ có tham số tiêu đề alg là bắt buộc. Tuy nhiên, trên thực tế, các tiêu đề JWT (còn được gọi là tiêu đề JOSE) thường chứa một số tham số khác. Những cái sau đây được những kẻ tấn công đặc biệt quan tâm.

  - `jwk (JSON Web Key)` - là một định dạng được tiêu chuẩn hóa để biểu diễn các khóa dưới dạng đối tượng JSON, các máy chủ có thể sử dụng để nhúng khóa công khai của chúng trực tiếp vào trong chính mã thông báo ở định dạng JWK.

  - `jku (JSON Web Key Set URL)` - Cung cấp một URL từ đó các máy chủ có thể tìm nạp một bộ khóa chứa khóa chính xác.

  - `kid (Key ID)` - Cung cấp ID mà máy chủ có thể sử dụng để xác định khóa chính xác trong trường hợp có nhiều khóa để chọn. Tùy thuộc vào định dạng của khóa, khóa này có thể có tham số con phù hợp

- `jwt header:`

```
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```

- Lý tưởng nhất là các máy chủ chỉ nên sử dụng một danh sách trắng giới hạn các khóa công khai để xác minh chữ ký JWT. Tuy nhiên, các máy chủ được định cấu hình sai đôi khi sử dụng bất kỳ khóa nào được nhúng trong tham số jwk.

- Bạn có thể khai thác hành vi này bằng cách ký JWT đã sửa đổi bằng khóa riêng RSA của riêng bạn, sau đó nhúng khóa chung phù hợp vào tiêu đề jwk.

> `Lab: JWT authentication bypass via jwk header injection`

- Ở trang web này tôi thực hiện tấn công bằng cách thêm `jkw` vào header. Để thực hiện tấn công.

![](./img_jwt/Screenshot%202023-06-28%20154938.png)

- Chúng ta sẽ tạo khóa RSA key

![](./img_jwt/Screenshot%202023-06-28%20155124.png)

- Thực hiện add payload "sub": "administrator" để thực hiện dưới quyền admin.

![](./img_jwt/Screenshot%202023-06-28%20155708.png)

> `Injecting self-signed JWTs via the jku parameter`

- Thay vì nhúng khóa công khai trực tiếp bằng tham số tiêu đề jwk, một số máy chủ cho phép bạn sử dụng tham số tiêu đề jku (JWK Set URL) để tham chiếu Bộ JWK chứa khóa. Khi xác minh chữ ký, máy chủ sẽ tìm nạp khóa có liên quan từ URL này.

```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

> `Lab: JWT authentication bypass via jku header injection`

- Thay vì chèn jwk vào header một cách trực tiếp chúng ta có thể tấn công thông qua jku header để nó sẽ có thể truy vấn đến url để lấy khóa.

- Chúng ta tạo url với key như sau:

![](./img_jwt/Screenshot%202023-06-28%20162517.png)

- Sau đó chúng ta set kid, jku, sub để chạy dưới quyền admin

![](./img_jwt/Screenshot%202023-06-28%20163104.png)

> `Injecting self-signed JWTs via the kid parameter`

- Máy chủ có thể sử dụng một số khóa mật mã để ký các loại dữ liệu khác nhau, không chỉ JWT. Vì lý do này, tiêu đề của JWT có thể chứa tham số kid (ID khóa), giúp máy chủ xác định khóa nào sẽ sử dụng khi xác minh chữ ký.

- Trong trường hợp này, máy chủ có thể chỉ cần tìm JWK có cùng KID.Tuy nhiên, đặc tả JWS không xác định cấu trúc cụ thể cho ID này - nó chỉ là một chuỗi tùy ý do nhà phát triển lựa chọn. Ví dụ: họ có thể sử dụng tham số kid để trỏ đến một mục nhập cụ thể trong cơ sở dữ liệu hoặc thậm chí là tên của tệp

```
{
    "kid": "../../path/to/file",
    "typ": "JWT",
    "alg": "HS256",
    "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
```

- Về mặt lý thuyết, bạn có thể làm điều này với bất kỳ tệp nào, nhưng một trong những phương pháp đơn giản nhất là sử dụng /dev/null, có trên hầu hết các hệ thống Linux. Vì đây là một tệp trống nên việc đọc nó sẽ trả về một chuỗi rỗng. Do đó, việc ký mã thông báo bằng một chuỗi trống sẽ dẫn đến chữ ký hợp lệ.

## **`Cách phòng chống tấn công JWT`**

- Sử dụng thư viện cập nhật để xử lý JWT và đảm bảo rằng nhà phát triển của bạn hiểu đầy đủ cách thức hoạt động của nó, cùng với bất kỳ ý nghĩa bảo mật nào. Các thư viện hiện đại khiến bạn gặp khó khăn hơn trong việc vô tình triển khai chúng một cách không an toàn, nhưng điều này không thể đánh lừa được do tính linh hoạt vốn có của các thông số kỹ thuật liên quan.

- Đảm bảo rằng bạn thực hiện xác minh chữ ký mạnh mẽ trên mọi JWT mà bạn nhận được và tính đến các trường hợp cạnh như JWT được ký bằng thuật toán không mong muốn.

- Thực thi một danh sách trắng nghiêm ngặt các máy chủ được phép cho tiêu đề jku.

- Đảm bảo rằng bạn không dễ bị xâm nhập đường dẫn hoặc chèn SQL thông qua tham số tiêu đề con.

## **`Algorithm JWT`**

> `Symmetric vs asymmetric algorithms`

- JWT có thể được ký bằng nhiều thuật toán khác nhau. Một số trong số này, chẳng hạn như HS256 (HMAC + SHA-256) sử dụng khóa "đối xứng". Điều này có nghĩa là máy chủ sử dụng một khóa duy nhất để ký và xác minh mã thông báo. Rõ ràng, điều này cần được giữ bí mật, giống như mật khẩu.

![](./img_jwt/Screenshot%202023-06-30%20151327.png)

- Các thuật toán khác, chẳng hạn như RS256 (RSA + SHA-256) sử dụng cặp khóa "bất đối xứng". Điều này bao gồm một khóa riêng mà máy chủ sử dụng để ký mã thông báo và một khóa chung liên quan đến toán học có thể được sử dụng để xác minh chữ ký.

![](./img_jwt/Screenshot%202023-06-30%20151636.png)

> `Cách thuật toán hoạt động`

![](./img_jwt/Screenshot%202023-06-30%20170437.png)

- [**`BÀI BLOG JWT`**](https://viblo.asia/p/jwt-tu-co-ban-den-chi-tiet-LzD5dXwe5jY#_3-luong-xu-ly-cua-1-he-thong-su-dung-bao-mat-jwt-6)
