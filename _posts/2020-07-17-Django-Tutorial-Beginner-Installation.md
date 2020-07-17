---
layout: post
title:  "Django Tutorial Beginner Part 1"
author: tuananhhodien
categories: [ Django ]
image: assets/images/2020-07-17-django-tutorial/installation.png
---

# Viết một chương trình Django đầu tiên, phần 1, cài đặt
Trước khi bắt đầu một chương trình Django, chúng ta cần phải cài đặt trước. Sau đây là một số bước bắt buộc.
## Cài đặt Python
Bởi vì Django là một framework python, nên ta phải cài đặt Python trước. Python có sẵn một database để test là SQLite, nên chúng ta không cần cài đặt thêm database mới gì hết.
Chúng ta lấy version cuối của Python tại [https://www.python.org/downloads/](https://www.python.org/downloads/) tuỳ theo từng hệ điều hành.
Khi cài đặt xong, chúng ta kiểm tra nó đã cài đặt thành công bằng câu lệnh:
```json
python3 
Python 3.8.2 (v3.8.2:7b3ab5921f, Feb 24 2020, 17:52:18) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
## Cài đặt database
Phần cài đặt này là cần thiết nếu bạn làm với một database lớn như MySQL, PostgreSQL, MariaDB,... Tạm thời chúng ta sẽ dùng SQLite.
## Cài đặt Django
Có nhiều cách để cài đặt Django, nhưng chúng ta sẽ cài đặt bằng cách chính thống nhất.
### Cài đặt pip
Cài đặt pip theo hướng dẫn [https://pip.pypa.io/en/stable/installing/](https://pip.pypa.io/en/stable/installing/).
Nếu chúng ta đã có pip cài đặt rồi, thì chú ý nếu nó bị quá hạn, một khi nó quá hạn, sẽ không thể cài đặt được.
Nâng cấp pip bằng câu lệnh 
```json
python -m pip install -U pip
```
### Cài đặt môi trường ảo venv
[https://docs.python.org/3/tutorial/venv.html](https://docs.python.org/3/tutorial/venv.html) hỗ trợ một môi trường tương tác độc lập, 
không ảnh hưởng đến thư viện dùng chung của python.
```json
python3 -m venv tutorial-env
```
Để active môi trường:
* Trên Linux/ Mac
 ```json
source tutorial-env/bin/activate
 ```
* Trên Windows
 ```json
tutorial-env\Scripts\activate.bat
 ```
### Cài đặt Django
```json
python -m pip install Django
```
### Kiểm tra lại
Để kiểm tra lại Django đã được cài đặt hoàn tất, chúng ta làm theo lệnh sau
 ```json
python3
Python 3.8.2 (v3.8.2:7b3ab5921f, Feb 24 2020, 17:52:18) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import django
>>> print(django.get_version())
2.2
>>> 
 ```
### Hoàn tất
Vậy là chúng ta đã xong phần cài đặt Django.