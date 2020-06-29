---
layout: post
title:  "Handle Exception"
author: tuananhhodien
categories: [ Lập trình ]
image: assets/images/2019-10-26-handle-exception/home.png
tags: [featured]
---

# Phân loại lỗi:
![alt text]({{ site.baseurl }}/assets/images/2019-10-26-handle-exception/home.png)
## Exceptions: Tất cả các exceptions hệ thống quăng ra.
### Checked: Các Exceptions được hệ thống quăng ra giúp chúng ta (quá trình compile time)
Ví dụ: IOException, SQL Exception, …
### Unchecked: Các Exceptions xuất hiện trong quá trình runtime.
Ví dụ: NullPointerException, ClasscastException
#### Handled: Theo business, những Exceptions nào sẽ được Handled để xử lý
Ví dụ: Khi đọc từ db không thấy record (EntityNotFoundException) nhưng business là insert mới vào nếu không có, khi này sẽ được handled.
#### Unhanded: Khi không biết phải xử lý sao với exceptions, khi này sẽ throws Exceptions cho function cao hơn.
##### Client: Lỗi thuộc về Client, lỗi này do input của người nhập sai format hoặc thông tin không phù hợp với Business
Ví dụ: Input của người dùng nhập cmnd nhưng không tìm thấy cmnd trong db.
##### Server: Lỗi này thuộc về Server, lỗi này do xử lý của server không bị về mặt kỹ thuật.
Ví dụ: Server từng có một file lưu thông tin về config (config), nhưng không tồn tại sau quá trình dev, khi đó file config không đọc được, lỗi IOException sẽ quăng ra.
# Xử lý lỗi: Lỗi không handled được sẽ được throws ra

### Rule 1: Quy ước về header status
* Client side: header status = 400 (Bad Request)
* Server side: header status = 500 (Internal Server Error)
Ref: https://www.restapitutorial.com/httpstatuscodes.html
### Rule 2: Response body lỗi luôn ở format:
```json
{
    "statusCode": 400, // Client or Server side 
    "reasonPhrase": "Bad Request",
    "errors": [ // List of Errors 
        {
            "code": "last-name-required", // Code defined in exception.properties 
            "message": "last_name is required" // Message defined by code in exception.properties
        },
        {
            "code": "geek-id-exist",
            "message": "geek with id = 1 is already exists in database"
        },
        {
            "code": "first-name-required",
            "message": "first_name is required"
        },
        {
            "code": "phone-min-value",
            "message": "phone should not lesser than 1"
        }
    ]
}
```
### Rule 3: Luôn luôn thows exceptions và catch exceptions throws ra với log.error(described message).
•	Lý do log.error – debug issue của backend dễ dàng hơn.
```java
try {
    Desktop desktop = Desktop.getDesktop();
    
    if (validUrl(url)) {// throw business exception
        throw new UserInputIncorrectUrlExeption();
    } 

    desktop.browse(new URI(url));
} catch (IOException | URISyntaxException | UserInputIncorrectUrlExeption e) {
    log.error("openBrowserWithUrl = " + url + " fail");
    throw e;
}
```
### Rule 4: Luôn luôn thows exceptions và catch exceptions throws ra với log.error(described message, exeption) ở cấp cao nhất (thường là tầng controller)
Lý do: Lúc này vừa log được exceptions, vừa print stack trace để biết code lỗi chỗ nào
```java
try {
    String token = getToken(TOKEN_URL);
    String message = getWelcomeMessageWithAuth(token, MESSAGE_URL);
    openBrowserWithUrl(GOOGLE_SEARCH_QUERY + message);
} catch (Exception e) {
    log.error("searchOnBrowserWithMessage fail ", e); 
}
```
### Rule 5: Tất cả những lỗi thuộc về validation syntax sẽ được xử lý trước, sau đó mới tới các lỗi xử lý ở tầng business (database, business rule).