# **Web cache poisoning**

## **Web cache poisoning là gì?**

- Web cache poisoning là một kĩ thuật nâng cao, kẻ tấn công khái thác hành vi của máy chủ web và bộ đệm để gửi phản hồi HTTP có hại cho người dùng khác.

- Về cơ bản, Web cache poisoning bao gồm 2 giai đoạn. Đầu tiên, kẻ tấn công phải tìm ra phản hồi từ máy chủ. Cần đảm bảo rằng phản hồi của họ được lưu trong bộ nhớ cache và sau đó được gửi tới nạn nhân dự kiến.

## **Cách hoạt động của Web cache**

- Nếu một máy chủ phải gửi phản hồi mới cho từng yêu cầu HTTP riêng lẻ, điều này có thể sẽ làm máy chủ quá tải, dẫn đến các vấn đề về độ trễ và trải nghiệm người dùng kém, đặc biệt là trong thời gian bận rộn. Bộ nhớ đệm chủ yếu là một phương tiện để giảm các vấn đề như vậy.

![](./img_wcp/Screenshot%202023-06-17%20162116.png)

- Bộ đệm nằm giữa máy chủ và người dùng. Nơi nó lưu các phản hồi yêu cầu cụ thể, thường được lưu trữ một thời gian cố định. Nếu một người dùng khác gửi yêu cầu tương đương, thì bộ nhớ đệm chỉ cung cấp 1 phản hồi đã lưu trước đó.

- Bộ đệm kiểm tra yêu cầu có tương đương với yêu cầu đã lưu trong cache hay không bằng cách so sánh các `Cache keys`.

## **Xây dựng một cuộc tấn công  Web cache poisoning**

> **Identify and evaluate unkeyed inputs**

- Bất kì 1 cuộc tấn công bộ đệm nào đều dựa vào `unkeyed inputs`, chẳng hạn như `headers`. Bộ đệm đều bỏ qua các `unkeyed inputs` khi quyết định có cung cấp phản hồi được lưu trong bộ nhớ cache hay không. Hành vi này có nghĩa là bạn có thể sử dụng chúng để đưa vào tải trọng của mình và tạo ra phản hồi "cache poisoning", nếu được lưu vào bộ đệm ẩn sẽ được phân phát cho tất cả người dùng có yêu cầu có khóa bộ đệm phù hợp. Do đó, bước đầu tiên khi xây dựng một cuộc tấn công đầu độc bộ đệm web là xác định các `unkeyed inputs` được máy chủ hỗ trợ.

> **Elicit a harmful response from the back-end server**

- Khi bạn đã xác định được `unkeyed inputs`, bước tiếp theo là đánh giá chính xác cách trang web xử lý đầu vào đó. Hiểu điều này là điều cần thiết để khơi gợi thành công một phản hồi có hại. Nếu một đầu vào được phản ánh trong phản hồi từ máy chủ mà không được làm sạch đúng cách hoặc được sử dụng để tạo động dữ liệu khác, thì đây là một điểm vào tiềm ẩn cho việc đầu độc bộ nhớ đệm web.

- Việc phản hồi có được lưu vào bộ đệm ẩn hay không có thể phụ thuộc vào tất cả các loại yếu tố, chẳng hạn như phần mở rộng tệp, loại nội dung, tuyến đường, mã trạng thái và tiêu đề phản hồi. Có thể bạn sẽ cần dành một chút thời gian để đơn giản là giải quyết các yêu cầu trên các trang khác nhau và nghiên cứu cách thức hoạt động của bộ nhớ cache. Sau khi tìm ra cách nhận được phản hồi được lưu trong bộ nhớ đệm có chứa đầu vào độc hại của bạn, bạn đã sẵn sàng cung cấp khai thác cho các nạn nhân tiềm năng.

## **Cách ngăn chặn lỗ hổng Web cache poisoning**

- Cách dứt khoát để ngăn chặn ngộ độc bộ đệm web rõ ràng là vô hiệu hóa hoàn toàn bộ nhớ đệm.

- Vô hiệu hóa các header không cần thiết để trong trang web.

## **Khai thác lỗ hổng Web cache poisoning**

### **Exploiting cache implementation flaws(Triển khai hệ thống)**

- **Cache probing methodology :**
  - Xác định một oracle cache phù hợp:
    - Một `cache oracle` là một trang hoặc một endpoint cung cấp phản hồi về hành vi của bộ đệm. Điều này cần phải lưu được trong bộ nhớ cache và phải cho biết cách nào bạn đã nhận được phản hồi lưu trong bộ nhớ cache hay phản hồi trực tiếp từ máy chủ .
      - `An HTTP header that explicitly tells you whether you got a cache hit`
      - `Observable changes to dynamic content`
      - `Distinct response times`
  - Probe key handling:
    - Đối với mỗi trường hợp mà bạn muốn kiểm tra, bạn gửi hai yêu cầu tương tự và so sánh các phản hồi. Để xác định xem yếu tố nào được loại khỏi cache key
  - Identify an exploitable gadget:
    - Các tiện ích này thường là các lỗ hổng cổ điện phía client side.

- **Exploiting cache key flaws :**
  - `Unkeyed port`
  - `Unkeyed query string`:

> `Lab: Web cache poisoning via an unkeyed query string`

- Trong một số trường hợp, có thể chèn tải trọng độc hại vào tham số truy vấn và lưu vào cache response từ máy chủ nếu tham số truy vấn là `unkeyed` và được phản ánh trong phản hồi. Sau đó, người dùng sẽ nhận được phản hồi bị nhiễm độc nếu họ gửi yêu cầu phù hợp mà không có chuỗi truy vấn.

- Phương pháp này chuyển đổi XSS được phản ánh thành XSS được lưu trữ vì cuộc tấn công là một kiểu tiêm tập lệnh điển hình và tập lệnh được lưu trong bộ nhớ cache của web. Mặc dù phương pháp này có thể dễ dàng bị phát hiện khi được thực hiện trực tiếp, nhưng nó có thể tránh bị phát hiện trong các tình huống phức tạp hơn nhiều.

- Các bước để thực thi tấn công:
  - Chèn các tham số truy vấn tùy ý vào URL yêu cầu. Lưu ý rằng nếu bạn sửa đổi các tham số truy vấn, bạn vẫn có thể nhận được lần truy cập vào bộ đệm. Điều này cho thấy chúng không phải là một phần của khóa bộ đệm.
  - Quan sát xem các tham số được đưa vào của bạn được phản ánh lại trong phản hồi khi yêu cầu bị lỗi bộ đệm.
  - Chèn một truy vấn tùy ý để thoát khỏi chuỗi được phản ánh và thêm tải trọng XSS.

![](./img_wcp/Screenshot%202023-06-17%20231315.png)

![](./img_wcp/Screenshot%202023-06-17%20231401.png)

![](./img_wcp/Screenshot%202023-06-17%20231511.png)

- `Unkeyed query parameters`

> `Lab: Web cache poisoning via an unkeyed query parameter`

- Một số trang web loại trừ query string hoàn chỉnh khỏi cache key. Tuy nhiên, một số trang web loại trừ các tham số truy vấn không liên quan đến chương trình phụ trợ, chẳng hạn như các tham số để theo dõi hoặc quảng cáo được nhắm mục tiêu. Các tham số UTM như utm_content là mục tiêu tuyệt vời để kiểm tra lỗ hổng này.
- [**DOCS utm_tracking**](https://bkhost.vn/blog/utm-tracking/)
- Các bước để thực thi tấn công:
  - Chèn các tham số truy vấn tùy ý `utm_content=abc123` vào URL yêu cầu. Lưu ý rằng nếu bạn sửa đổi các tham số truy vấn, bạn vẫn có thể nhận được lần truy cập vào bộ đệm. Điều này cho thấy chúng không phải là một phần của khóa bộ đệm.
  - Quan sát xem các tham số được đưa vào của bạn được phản ánh lại trong phản hồi khi yêu cầu bị lỗi bộ đệm.
  - Chèn một truy vấn tùy ý để thoát khỏi chuỗi được phản ánh và thêm tải trọng XSS.

![](./img_wcp/Screenshot%202023-06-17%20234836.png)

- `Cache parameter cloaking (Che giấu tham số bộ đệm)`

  - Nếu bạn có thể tìm ra cách bộ đệm phân tích cú pháp URL để xác định và xóa các tham số không mong muốn, thì bạn có thể tìm thấy một số điểm thú vị.

> `Lab: Parameter cloaking`

- Các bước thực hiện tấn công:
  - Chèn tham số utm_content và gửi yêu cầu.

![](./img_wcp/Screenshot%202023-06-18%20003420.png)

    - Như ta đã thấy việc thực hiện cache poisoning ở đây dường như không thể
    - Phát hiện ra script /js/geolocate.js có sử dụng web cache 
    - Sử dụng công cụ Param miner

![](./img_wcp/Screenshot%202023-06-18%20011133.png)

    - Thực hiện tấn công với payload xss
    - Cả dấu và (&) và dấu chấm phẩy (;) đều được coi là dấu phân cách
![](./img_wcp/Screenshot%202023-06-18%20011421.png)

- `Exploiting fat GET support`
  - Trong một số trường hợp, method HTTP sẽ là `unkeyed`.
  - `ví dụ:`

    ```
    GET /?param=innocent HTTP/1.1
    Host: innocent-website.com
    X-HTTP-Method-Override: POST
    …
    param=bad-stuff-here
    ```

  - Kẻ tấn công có thể đưa một tải trọng độc hại vào yêu cầu GET và vì nội dung yêu cầu không được bao gồm trong khóa bộ đệm nên phản hồi sẽ được lưu vào bộ đệm. Sau đó, nếu một người dùng hợp pháp thực hiện một yêu cầu GET thông thường khớp với cùng một khóa bộ đệm, thì họ sẽ nhận được phản hồi bị nhiễm độc từ bộ đệm.

> `Lab: Web cache poisoning via a fat GET request`

- Các bước thực hiện tấn công:
  - Trong Burpsuite, điều hướng đến lịch sử HTTP, quan sát GET /js/geolocate.js?callback=setCountryCookie và gửi nó đến tab bộ lặp.
  - Để tự động hóa quy trình này với công cụ khai thác thông số, hãy nhấp chuột phải vào < công cụ khai thác thông số < select fat GET scan.

  ![](./img_wcp/Screenshot%202023-06-18%20132354.png)
  ![](./img_wcp/Screenshot%202023-06-18%20132635.png)

- `Normalized cache keys`
  - Hành vi này có thể cho phép bạn khai thác các lỗ hổng XSS. Nếu bạn gửi một yêu cầu độc hại bằng Burp Repeater, bạn có thể đầu độc bộ đệm bằng tải trọng XSS chưa được mã hóa. Khi nạn nhân truy cập URL độc hại, tải trọng sẽ vẫn được trình duyệt của họ mã hóa URL; tuy nhiên, sau khi URL được chuẩn hóa bởi bộ nhớ cache, nó sẽ có cùng khóa bộ nhớ cache với phản hồi chứa tải trọng chưa được mã hóa của bạn.

- `Cache key injection`
  - Các thành phần có khóa thường được nhóm lại với nhau thành một chuỗi để tạo khóa bộ đệm. Nếu bộ nhớ đệm không thực hiện thoát đúng cách các dấu phân cách giữa các thành phần, bạn có khả năng có thể khai thác hành vi này để tạo hai yêu cầu khác nhau có cùng khóa bộ đệm.

    ![](./img_wcp/Screenshot%202023-06-18%20141056.png)

> `Lab: Cache key injection`

- Các bước thực hiện tấn công:
  - Chèn utm_content vào param ta nhìn kĩ phản hồi.
    ![](./img_wcp/Screenshot%202023-06-18%20142248.png)
  - Đi đến `/js/localize.js.` Ta biết được trang web có sử dụng web cache.
  - Thực hiện cache injection
    ![](./img_wcp/Screenshot%202023-06-18%20142539.png)

- `Using web cache poisoning to exploit unsafe handling of resource imports`
  - Một số trang web sử dụng `unkeyed headers` để tự động tạo URL để nhập tài nguyên, chẳng hạn như tệp JavaScript được lưu trữ bên ngoài. Trong trường hợp này, nếu kẻ tấn công thay đổi giá trị của `unkeyed headers` phù hợp thành miền mà chúng kiểm soát, thì chúng có khả năng thao túng URL để trỏ đến tệp JavaScript độc hại của chính chúng.

> `Lab: Web cache poisoning with an unkeyed header`

- Các bước tấn công:
  - Gửi yêu cầu và quan sát thấy rằng tiêu đề X-Forwarded-Host đã được sử dụng để tạo động một URL tuyệt đối để nhập tệp JavaScript được lưu trữ tại /resources/js/tracking.js.

    ![](./img_wcp/Screenshot%202023-06-18%20165305.png)

  - Thực hiện khai thác qua url kẻ tấn công

    ![](./img_wcp/Screenshot%202023-06-18%20165903.png)
    ![](./img_wcp/Screenshot%202023-06-18%20165926.png)

- `Using web cache poisoning to exploit cookie-handling vulnerabilities`
  - Lỗi xử lý cookie bằng bộ đệm này cũng có thể bị khai thác bằng cách sử dụng các kỹ thuật đầu độc bộ đệm web. Tuy nhiên, trên thực tế, vectơ này tương đối hiếm khi so với ngộ độc bộ đệm dựa trên tiêu đề. Khi tồn tại các lỗ hổng đầu độc bộ đệm dựa trên cookie, chúng có xu hướng được xác định và giải quyết nhanh chóng do người dùng hợp pháp đã vô tình đầu độc bộ đệm.

> `Lab: Web cache poisoning with an unkeyed cookie`

- Chúng ta sẽ thấy rằng cookie được phản ánh trong phản hồi từ đó chúng ta sẽ thực hiện XSS vào và để `X-cache:hit`

![](./img_wcp/Screenshot%202023-06-18%20204957.png)

![](./img_wcp/Screenshot%202023-06-18%20205211.png)

> `Lab: Web cache poisoning with multiple headers`

- Khi tôi thực hiện `X-Forwarded-Host` quan sát thì tôi không thấy bất kì điều gì xảy ra.

![](./img_wcp/Screenshot%202023-06-18%20211836.png)

- Khi tôi thực hiện `X-Forwarded-Schema` quan sát thì tôi thấy trang web đã bắt tôi chuyển hướng tới https. Điều đó có nghĩa trang web chỉ chấn nhận `https://`

![](./img_wcp/Screenshot%202023-06-18%20213543.png)

- Từ đấy, tôi đã thực hiện tấn công để trang web có thể lấy mã độc từ web của tôi.

![](./img_wcp/Screenshot%202023-06-18%20214907.png)
