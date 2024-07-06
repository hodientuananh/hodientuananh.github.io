---
layout: post
title:  "Cách xử lý khi lệch múi giờ giữa front end và back end"
author: hodientuananh
categories: [ Lập trình ]
image: assets/images/2019-11-29-lech-mui-gio/home.jpg
---

# Một số khái niệm
![alt text]({{ site.baseurl }}/assets/images/2019-11-29-lech-mui-gio/home.jpg)
## Chuẩn ISO 8601
Chuẩn hiển thị ngày tháng và múi giờ máy và người có thể hiểu. [wikipedia](https://en.wikipedia.org/wiki/ISO_8601)
## Một số quy ước
### Ngày bắt đầu chương trình MI
* Oracle: "2019-10-15 17:00:00.0 +00:00"
* Javascript:
    * new Date('2019-10-15 17:00:00.0 +0:00')
    * Wed Oct 16 2019 00:00:00 GMT+0700 (Indochina Time)
    * "2019-10-15T17:00:00.000Z"
* Java: "2019-10-15 17:00:00.0 +0:00"
### Ngày kết thúc chương trình MI
* Oracle: "2020-03-15 16:59:59.0 +0:00"
* Javascript:
    * new Date("2020-03-15 16:59:59.0 +0:00")
    * Sun Mar 15 2020 23:59:59 GMT+0700 (Indochina Time)
    * "2020-03-15T16:59:59.000Z"
* Java: "2020-03-15 16:59:59.0 +00:00"
### Múi giờ
* UTC: múi giờ 00:00
* Việt Nam = Indochina Time = +07:00
## Tài liệu tham khảo
* Cách insert cột timestamp vào Oracle đảm bảo chuẩn ISO 8601. [Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/18/sqlrf/TO_UTC_TIMESTAMP_TZ.html#GUID-1728EE3E-EC0C-4FA8-B404-99C0A445CE82)
Ví dụ:
```
UPDATE MI_INFO_TEST
SET START_DATE = TO_UTC_TIMESTAMP_TZ('2019-10-15T17:00:00.000Z')
WHERE MI_CODE = 'GIFT_INFO_2020';

UPDATE MI_INFO_TEST
SET END_DATE = TO_UTC_TIMESTAMP_TZ('2020-03-15T16:59:59.000Z')
WHERE MI_CODE = 'GIFT_INFO_2020';
```

* Tại sao nên lưu ngày vào DATE/ Timestamp thay vì VARCHAR. [stackoverflow](https://stackoverflow.com/questions/4759012/when-to-use-varchar-and-date-datetime)
    * Dễ dàng thêm/ bớt ngày.
    * Dễ dàng trích xuất ngày/ tháng/ năm.
    * Dễ dàng sort ngày.
    * Dễ dàng chuyển quan format VARCHAR, ngược lại thì rất khó.
    * Tiết kiệm memory space.
    * Hiệu năng cao hơn VARCHAR khi số row tăng thêm 2m – 3m.
    * Convention cho developer khác hiểu.
* 3 rules để có thể handle lệch ngày. [dev.to](https://dev.to/corykeane/3-simple-rules-for-effectively-handling-dates-and-timezones-1pe0)
    * Rule 1: Luôn giữ code ở backend và thông tin ở database với với chuẩn UTC. Vì đây là giờ chuẩn mà các timezone khác đều tính toán dựa vào nó, những timezone khác là offset time dựa trên UTC.
    * Rule 2: Convert datetime cho user ở từng địa phương thông qua frontend, tức là frontend sẽ chuyển từ giờ UTC sang múi giờ địa phương để hiển thị cho khách hàng.
    * Rule 3: Sử dụng thư viện chuẩn để handle (theo chuẩn ISO 8601)
    * Front end: moment.js
    * Backend: java.time.*
* Chú ý: Theo rule c.i nhưng không assump hệ thống Back end phải chạy theo chuẩn UTC (vì sysadmin có thể thay đổi muí giờ hệ thống thành bất cứ khi nào, hay JVM có thể change timezone bằng cách gọi Timezone.setDefault). [stackoverflow](https://stackoverflow.com/questions/33343893/handling-time-zone-in-web-application)
* Một số best practices để handle lệch ngày (cũng theo 3 rules trên). [stackoverflow](https://stackoverflow.com/questions/2532729/daylight-saving-time-and-time-zone-best-practices)
* Cách get offset time (lệch thời gian so với UTC +00:00) trên Front end. [stackoverflow](https://stackoverflow.com/questions/1091372/getting-the-clients-timezone-offset-in-javascript)
* Cách Front end bỏ qua offset. [stackoverflow](https://stackoverflow.com/questions/1486476/json-stringify-changes-time-of-date-because-of-utc)
# Vấn đề đặt ra
## Back end giữa server Việt Nam và server Azure đưa ra kết quả khác nhau cho cùng 1 input và logic và database:
![alt text]({{ site.baseurl }}/assets/images/2019-11-29-lech-mui-gio/van-de.png)
### Bài toán đặt ra
Chương trình MI diễn ra từ 00:00:00 ngày 16-10-2019 đến 23:59:59 ngày 15-03-2019.
### Tổ chức database MI
![alt text]({{ site.baseurl }}/assets/images/2019-11-29-lech-mui-gio/database.png)
## Hiện trạng
* User (Việt Nam): Người dùng từ Việt Nam đang hiểu chương trình sẽ được áp dụng từ 00:00:00 ngày 16-10-2019, múi giờ +07:00
* Server (Việt Nam): Chấp nhận request vì request trong giờ bắt đầu chương trình: 00:00:00 ngày 16-10-2019, múi giờ +07:00
* Server (Azure): Không chấp nhận request, vì server hiểu request vào thời điểm 17:00:00.0 15-10-2019  +0:00' < 00:00:00 ngày 16-10-2019, múi giờ +0:00, tức là server chỉ chấp nhận request từ Việt Nam vào lúc 07:00:00 ngày 16-10-2019, múi giờ +07:00
## Giải pháp và lợi ích/ đánh đổi
### Front end sẽ theo Back end
Front end sẽ + 07:00 (offset) để đáp ứng yêu cầu của server Azure.
Lợi ích:
* Back end sẽ không cần xử lý việc lệch giờ vì Front end sẽ theo chuẩn giờ UTC.
* Đáp ứng được các best practices.
* Đáp ứng được 2 rules handle.
* Database không cần thay đổi.
Đánh đổi:
* Bất kỳ Front end từ một Việt Nam đều mặc định phải biết rule + 07:00 để theo thời gian chuẩn UTC.
* Việc hiển thị ngày request khi nhận thông tin từ server Azure về phải biết – 07:00 để hiển thị cho đúng.
* Sau này nếu 1 timezone khác tham gia vào chương trình cũng phải hiểu việc thêm bớt offset để theo chuẩn UTC.
* Rule Back End sẽ bị vi phạm, tức là nếu hệ thống bị thay đổi thành múi giờ khác thì coi như mọi thông tin của database đều vô giá trị. Vì backend giờ sẽ hiểu database ở một múi giờ khác (không còn +00:00).
### Back end sẽ theo Front end
Back end sẽ +07:00 những request từ Việt Nam để hiểu là theo múi giờ Việt Nam thì đây là request hợp lệ. Điều này đồng nghĩa với request từ Việt Nam phải gửi kèm offset trong request của mình.
Lợi ích:
* Front end sẽ không còn quan tâm đến việc thêm bớt múi giờ vì Back end đã làm.
* Đáp ứng các best practices.
* Đáp ứng được 2 rules handle.
* Sau này nếu 1 timezone khác tham gia vào chương trình sẽ chỉ quan tâm đến múi giờ địa phương => dễ mở rộng.
* Việc hiển thị ngày request khi nhận thông tin từ server Azure/ server Việt Nam về sẽ giống với lúc gửi đi, không cần thêm bớt offset nữa => dễ sử dụng.
Đánh đổi:
* Back end sẽ phải thêm logic xử lý request từ timezone khác để xem với timezone đó thì chương trình có hợp lệ.
* Database phải thêm cột offset.
* Rule Back End sẽ bị vi phạm, tức là nếu hệ thống bị thay đổi thành múi giờ khác thì coi như mọi thông tin của database đều vô giá trị. Vì backend giờ sẽ hiểu database ở một múi giờ khác (không còn +00:00).
### Chuyển tất cả thông tin ngày thành String
Không còn quan tâm tới vấn đề chênh lệch múi giờ, mọi thông tin đều tường minh.
Lợi ích:
* Front end và Back end không còn quan tâm đến múi giờ, gửi thông tin nào thì sẽ nhận lại ngày đó, database cũng sẽ lưu thông tin dưới dạng String, không còn quan tâm đến giờ.
* Sau này nếu 1 timezone khác tham gia vào chương trình sẽ không còn quan tâm đến múi giờ chênh lệch => dễ mở rộng múi giờ.
* Việc hiển thị ngày request khi nhận thông tin từ server Azure/ server Việt Nam về sẽ giống với lúc gửi đi, không cần thêm bớt offset nữa => dễ sử dụng.
* Đáp ứng được 2 rules handle.
* Đáp ứng được rule back end.
* Database không cần thay đổi.
Đánh đổi:
* Front end và Back end phải thống nhất một format để xử lý Date sau này.
* Vi phạm các best practices. => sẽ khó sửa chữa sau này.
* Database sẽ vi phạm rule => sẽ đánh đổi sau này.
### Cố định database ở mốc thời gian Việt Nam
Chuyển start date + end date thành dạng timestamp (dạng duy nhất cố định múi giờ UTC) và start date + end date theo khung giờ Việt Nam.
Cụ thể: 
![alt text]({{ site.baseurl }}/assets/images/2019-11-29-lech-mui-gio/co-dinh-database.png)
Lợi ích:
* Đáp ứng được 3 rule handle.
* Đáp ứng được rule back end.
* Database sẽ đáp ứng được rule.
* Đáp ứng các best practices.
* Việc hiển thị ngày request khi nhận thông tin từ server Azure/ server Việt Nam về sẽ giống với lúc gửi đi, không cần thêm bớt offset nữa => dễ sử dụng.
Đánh đổi:
* Front end và Back end sẽ phải thay đổi lại theo chuẩn ISO8601.
* Database phải thay đổi (Tất cả cột liên quan đến Date).
* Sau này nếu 1 timezone khác tham gia vào chương trình sẽ phải quan tâm đến timezone của chương trình => Bài toán hiện xử lý cho thị trường Việt Nam.
