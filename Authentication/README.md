# **Authentication**

## **Lab: 2FA simple bypass**

- Ở phòng thí nghiệm này có xác thực 2 yếu tố nhưng có thể bỏ qua. Ở phòng thí nghiệm này chúng ta nhận được tên người dùng và mật khẩu hợp lệ nhưng không có quyền vào mã xác minh 2FA của người dùng.

- Khi tôi đăng nhập tài khoản nó sẽ bắt nhập thêm một mã xác minh gửi qua email.

![](./img_auth/3.png)

![](./img_auth/2.png)

![](./img_auth/1.png)

- Tôi đã thử việc tấn công qua 2 mã xác thực bằng cách khi bắt nhập mã code xác thực thì tôi chuyển hướng đến trang web `my-account`

## **Lab: Password reset broken logic**

- Chức năng đặt lại mật khẩu của phòng thí nghiệm này dễ bị tấn công. Để giải quyết phòng thí nghiệm, hãy đặt lại mật khẩu của Carlos, sau đó đăng nhập và truy cập trang "Tài khoản của tôi" của anh ấy.

- Khi tôi thử chức năng `Forgot your password` thì nó đã gửi cho tôi url kèm token để đổi một mật khẩu mới nên tôi đã thử sử dụng điều ấy để thay đổi mật khẩu của tài khoản người khác.

![](./img_auth/5.png)

- Nhưng khi tôi sửa hoặc xóa token đi mã vẫn hoạt động điều chứng tỏ tôi có thể tấn công và thay đổi mật khẩu người dùng khác.

## **Lab: Username enumeration via response timing**

- Vì một số trang web giới hạn số lần đăng nhập khi mình đăng nhập sai vậy làm cách nào để mình brute force để tìm mật khẩu người dùng ?

- Tôi đã sử dụng tiêu đề header `X-Forwarded-For`, nó sẽ cho phép tôi giả mạo địa chỉ IP.

- Thực hiện nhận biết qua thời gian phản hồi.

![](./img_auth/6.png)

## **Lab: 2FA broken logic**

- Ở phòng thí nghiệm này, chúng ta có mã xác thực 2 lần gửi qua email.

![](./img_auth/7.png)

- Nhưng ở trang web `/login2`, tôi đã thấy có xác thực ở cookies nên tôi đã nghĩ nếu tôi đã thử đổi cookie thành carlos để bypass được mã xác thực rồi đăng nhập vào carlos.

![](./img_auth/9.png)

## **Lab: Brute-forcing a stay-logged-in cookie**

- Khi tôi đăng nhập vào tài khoản của mình tôi thấy cookie `stay-logged-in` nó được mã hóa base64 và md5 như sau:

![](./img_auth/10.png)

- Từ đó, tôi đã brute force để tìm mật khẩu như sau:

![](./img_auth/11.png)

![](./img_auth/12.png)

## **Lab: Password reset poisoning via middleware**

- Tôi đã thực hiện bypass mã bằng cách để mã gửi request đến proxy của tôi bằng header `X-Forwarded-Host: exploit-0a3b00bc0465e68280453994012e0065.exploit-server.net/exploit`

![](./img_auth/13.png)

## **Lab: Broken brute-force protection, multiple credentials per request**

- Khi tôi thực hiện đăng nhập thì nó gửi request đăng nhập theo json nên tôi đã thử brute force mật khẩu như sau:

![](./img_auth/14.png)

![](./img_auth/15.png)
