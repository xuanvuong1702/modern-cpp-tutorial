---
title: "Chương 01: Hướng Tới C++ Hiện Đại"
type: book-vi-vn
order: 1
---

# Chương 01: Hướng tới C++ hiện đại

[TOC]

**Môi trường biên dịch**: Cuốn sách này sẽ sử dụng `clang++` làm trình biên dịch duy nhất,
và luôn sử dụng cờ biên dịch `-std=c++2a` trong mã của bạn.

```bash
> clang++ -v
Apple LLVM version 10.0.1 (clang-1001.0.46.4)
Target: x86_64-apple-darwin18.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```

## 1.1 Các tính năng không còn được khuyến khích sử dụng

Trước khi tìm hiểu về C++ hiện đại, chúng ta hãy xem qua các tính năng chính đã không còn được khuyến khích sử dụng kể từ C++11:

> **Lưu ý**: Không còn được khuyến khích sử dụng không có nghĩa là hoàn toàn không thể sử dụng, nó chỉ nhằm ám chỉ rằng các tính năng này sẽ biến mất khỏi các tiêu chuẩn trong tương lai và nên tránh. Tuy nhiên, các tính năng không còn được khuyến khích sử dụng vẫn là một phần của thư viện chuẩn, và trên thực tế, hầu hết các tính năng này được "bảo lưu" vĩnh viễn vì lý do tương thích.

- **Không còn được phép gán hằng chuỗi ký tự cho `char *`. Nếu bạn cần gán và khởi tạo `char *` bằng một hằng chuỗi ký tự, bạn nên sử dụng `const char *` hoặc `auto`.**

  ```cpp
  char *str = "hello world!"; // Sẽ xuất hiện cảnh báo không còn được khuyến khích sử dụng
  ```

- **Mô tả ngoại lệ của C++98, `unexpected_handler`, `set_unexpected()` và các tính năng liên quan khác đã không còn được khuyến khích sử dụng và nên sử dụng `noexcept`.**

- **`auto_ptr` đã không còn được khuyến khích sử dụng và nên sử dụng `unique_ptr`.**

- **Từ khóa `register` đã không còn được khuyến khích sử dụng và có thể được sử dụng nhưng không còn ý nghĩa thực tế.**

- **Phép toán `++` trên kiểu `bool` đã không còn được khuyến khích sử dụng.**

- **Nếu một lớp có hàm hủy, các thuộc tính mà nó tạo ra hàm tạo sao chép và toán tử gán sao chép đã không còn được khuyến khích sử dụng.**

- **Ép kiểu theo phong cách ngôn ngữ C đã không còn được khuyến khích sử dụng (ví dụ: sử dụng `(convert_type)`) trước các biến, và nên sử dụng `static_cast`, `reinterpret_cast`, `const_cast` để ép kiểu.**

- **Đặc biệt, một số thư viện chuẩn C có thể sử dụng đã không còn được khuyến khích sử dụng trong tiêu chuẩn C++17 mới nhất, chẳng hạn như `<ccomplex>`, `<cstdalign>`, `<cstdbool>` và `<ctgmath>`, v.v.**

- ... và nhiều hơn nữa

Ngoài ra còn có các tính năng khác như ràng buộc tham số (C++11 cung cấp `std::bind` và `std::function`), `export`, v.v. cũng đã không còn được khuyến khích sử dụng. Các tính năng được đề cập ở trên **nếu bạn chưa bao giờ sử dụng hoặc nghe nói về chúng, vui lòng đừng cố gắng tìm hiểu chúng. Bạn nên tiếp cận với tiêu chuẩn mới và học trực tiếp các tính năng mới**. Rốt cuộc, công nghệ luôn phát triển.

## 1.2 Tương thích với C

Vì một số lý do bất khả kháng và lịch sử, chúng ta phải sử dụng một số mã C (thậm chí là mã C cũ) trong C++, ví dụ: các lệnh gọi hệ thống Linux. Trước khi C++ hiện đại ra đời, hầu hết mọi người đều thảo luận về "sự khác biệt giữa C và C++ là gì". Nói chung, ngoài việc trả lời các tính năng lớp hướng đối tượng và các tính năng template của lập trình generic, không có ý kiến nào khác hoặc thậm chí là một câu trả lời trực tiếp. "Gần như" cũng là câu trả lời của rất nhiều người. Biểu đồ Venn trong Hình 1.2 đại khái trả lời câu hỏi về khả năng tương thích giữa C và C++.

![Hình 1.2: Khả năng tương thích giữa ISO C và ISO C++](../../assets/figures/comparison.png)

Từ giờ trở đi, bạn nên ghi nhớ rằng "C++ **không phải** là một siêu tập của C" (và ngay từ đầu đã không phải, sau này [Tài liệu tham khảo để đọc thêm](#further-readings) sẽ đưa ra sự khác biệt giữa C++98 và C99). Khi viết C++, bạn cũng nên tránh sử dụng các phong cách lập trình như `void*` bất cứ khi nào có thể. Khi bạn phải sử dụng C, bạn nên chú ý đến việc sử dụng `extern "C"`, tách biệt mã C khỏi mã C++, sau đó liên kết chúng lại với nhau, ví dụ:

```cpp
// foo.h
#ifdef __cplusplus
extern "C" {
#endif

int add(int x, int y);

#ifdef __cplusplus
}
#endif

// foo.c
int add(int x, int y) {
    return x+y;
}

// 1.1.cpp
#include "foo.h"
#include <iostream>
#include <functional>

int main() {
    [out = std::ref(std::cout << "Kết quả từ mã C: " << add(1, 2))](){
        out.get() << ".\n";
    }();
    return 0;
}
```

Trước tiên, bạn nên biên dịch mã C bằng `gcc`:

```bash
gcc -c foo.c
```

Biên dịch và xuất ra tệp `foo.o`, sau đó liên kết mã C++ với tệp `.o` bằng `clang++` (hoặc biên dịch cả hai thành `.o` rồi liên kết chúng lại với nhau):

```bash
clang++ 1.1.cpp foo.o -std=c++2a -o 1.1
```

Tất nhiên, bạn có thể sử dụng `Makefile` để biên dịch mã trên:

```makefile
C = gcc
CXX = clang++

SOURCE_C = foo.c
OBJECTS_C = foo.o

SOURCE_CXX = 1.1.cpp

TARGET = 1.1
LDFLAGS_COMMON = -std=c++2a

all:
	$(C) -c $(SOURCE_C)
	$(CXX) $(SOURCE_CXX) $(OBJECTS_C) $(LDFLAGS_COMMON) -o $(TARGET)

clean:
	rm -rf *.o $(TARGET)
```

> **Lưu ý**: Thụt đầu dòng trong `Makefile` là một ký tự tab chứ không phải dấu cách. Nếu bạn sao chép trực tiếp đoạn mã này vào trình soạn thảo của mình, tab có thể được tự động thay thế bằng dấu cách. Vui lòng đảm bảo thụt đầu dòng trong `Makefile` được thực hiện bằng tab.
>
> Nếu bạn không biết cách sử dụng `Makefile` cũng không sao. Trong hướng dẫn này, bạn sẽ không xây dựng mã quá phức tạp. Bạn cũng có thể đọc cuốn sách này bằng cách sử dụng `clang++ -std=c++2a` trên dòng lệnh.

Nếu bạn mới làm quen với C++ hiện đại, có lẽ bạn vẫn chưa hiểu đoạn mã nhỏ sau:

```cpp
[out = std::ref(std::cout << "Kết quả từ mã C: " << add(1, 2))](){
    out.get() << ".\n";
}();
```

Đừng lo lắng lúc này, chúng ta sẽ gặp lại chúng trong các chương sau.

[Mục lục](./toc.md) | [Chương trước](./00-preface.md) | [Chương tiếp theo: Cải thiện khả năng sử dụng ngôn ngữ](./02-usability.md)

## Đọc thêm

- [A Tour of C++ (Ấn bản thứ 2) Bjarne Stroustrup](https://www.amazon.com/dp/0134997832/ref=cm_sw_em_r_mt_dp_U_GogjDbHE2H53B)
  [Lịch sử của C++](http://en.cppreference.com/w/cpp/language/history)
- [Hỗ trợ trình biên dịch C++](https://en.cppreference.com/w/cpp/compiler_support)
- [Sự không tương thích giữa ISO C và ISO C++](http://david.tribble.com/text/cdiffs.htm#C99-vs-CPP98)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Ghi Công-Phi Thương mại-Không Phát sinh 4.0</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).
