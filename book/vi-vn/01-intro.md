---
title: "Chapter 01: Towards Modern C++"
type: book-en-us
order: 1
---

# Chương 01: Hướng tới C++ hiện đại

[TOC]

**Môi trường biên dịch**: Cuốn sách này sẽ sử dụng `clang++` làm trình biên dịch duy nhất được sử dụng,
và luôn sử dụng cờ biên dịch `-std=c++2a` trong mã của bạn.

```bash
> clang++ -v
Apple LLVM version 10.0.1 (clang-1001.0.46.4)
Target: x86_64-apple-darwin18.6.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```
## 1.1 Các Tính Năng Đã Bị Loại Bỏ

Trước khi học C++ hiện đại, hãy cùng xem qua các tính năng chính đã bị loại bỏ kể từ C++11:

> **Lưu ý**: Việc loại bỏ không có nghĩa là không thể sử dụng, mà chỉ nhằm ám chỉ rằng các tính năng này sẽ biến mất trong các tiêu chuẩn tương lai và nên tránh sử dụng. Tuy nhiên, các tính năng bị loại bỏ vẫn là một phần của thư viện chuẩn, và hầu hết các tính năng này thực sự được "dành riêng" vĩnh viễn vì lý do tương thích.

- **Hằng chuỗi ký tự không còn được phép gán cho `char *`. Nếu bạn cần gán và khởi tạo một `char *` với hằng chuỗi ký tự, bạn nên sử dụng `const char *` hoặc `auto`.**

  ```cpp
  char *str = "hello world!"; // Sẽ xuất hiện cảnh báo loại bỏ
  ```

- **Mô tả ngoại lệ của C++98, `unexpected_handler`, `set_unexpected()` và các tính năng liên quan khác đã bị loại bỏ và nên sử dụng `noexcept`.**

- **`auto_ptr` đã bị loại bỏ và nên sử dụng `unique_ptr`.**

- **Từ khóa `register` đã bị loại bỏ và có thể sử dụng nhưng không còn ý nghĩa thực tế.**

- **Phép toán `++` của kiểu `bool` đã bị loại bỏ.**

- **Nếu một lớp có hàm hủy, các thuộc tính mà nó tạo ra hàm tạo sao chép và toán tử gán sao chép đã bị loại bỏ.**

- **Kiểu chuyển đổi kiểu C ngôn ngữ đã bị loại bỏ (tức là sử dụng `(convert_type)`) trước các biến, và nên sử dụng `static_cast`, `reinterpret_cast`, `const_cast` để chuyển đổi kiểu.**

- **Đặc biệt, một số thư viện chuẩn của C có thể sử dụng đã bị loại bỏ trong tiêu chuẩn C++17 mới nhất, như `<ccomplex>`, `<cstdalign>`, `<cstdbool>` và `<ctgmath>` v.v.**

- ... và nhiều hơn nữa
Cũng có những tính năng khác như ràng buộc tham số (C++11 cung cấp `std::bind` và `std::function`), `export` v.v. cũng đã bị loại bỏ. Những tính năng được đề cập ở trên **Nếu bạn chưa từng sử dụng hoặc nghe về chúng, xin đừng cố gắng hiểu chúng. Bạn nên tiến gần hơn đến tiêu chuẩn mới và học các tính năng mới trực tiếp**. Sau tất cả, công nghệ đang tiến lên phía trước.

## 1.2 Tương Thích Với C

Vì một số lý do bất khả kháng và lịch sử, chúng ta phải sử dụng một số mã C (thậm chí là mã C cũ) trong C++, ví dụ như các lệnh gọi hệ thống của Linux. Trước khi C++ hiện đại ra đời, hầu hết mọi người đều nói về "sự khác biệt giữa C và C++ là gì". Nói chung, ngoài việc trả lời các tính năng lớp hướng đối tượng và các tính năng mẫu của lập trình tổng quát, không có ý kiến nào khác hoặc thậm chí là một câu trả lời trực tiếp. "Gần như" cũng là câu trả lời của nhiều người. Biểu đồ Venn trong Hình 1.2 trả lời một cách khái quát về sự tương thích giữa C và C++.

![Hình 1.2: Sự tương thích giữa ISO C và ISO C++](../../assets/figures/comparison.png)

Từ bây giờ, bạn nên có ý tưởng rằng "C++ **không phải** là một siêu tập của C" trong đầu (và không phải từ đầu, sau này [Tham khảo để đọc thêm](#further-readings) Sự khác biệt giữa C++98 và C99 được đưa ra). Khi viết C++, bạn cũng nên tránh sử dụng các kiểu lập trình như `void*` bất cứ khi nào có thể. Khi bạn phải sử dụng C, bạn nên chú ý đến việc sử dụng `extern "C"`, tách biệt mã C khỏi mã C++, và sau đó liên kết chúng lại với nhau, ví dụ:
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

Bạn nên biên dịch mã C trước bằng `gcc`:

```bash
gcc -c foo.c
```

Biên dịch và xuất tệp `foo.o`, sau đó liên kết mã C++ với tệp `.o` bằng `clang++` (hoặc cả hai biên dịch thành `.o` và sau đó liên kết chúng lại):

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

> **Lưu ý**: Thụt lề trong [`Makefile`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2FMakefile%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding/modern-cpp-tutorial/Makefile") là một ký tự tab thay vì ký tự khoảng trắng. Nếu bạn sao chép mã này trực tiếp vào trình soạn thảo của mình, tab có thể bị thay thế tự động. Hãy đảm bảo thụt lề trong [`Makefile`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2FMakefile%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding/modern-cpp-tutorial/Makefile") được thực hiện bằng tab.
>
> Nếu bạn không biết cách sử dụng [`Makefile`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2FMakefile%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding/modern-cpp-tutorial/Makefile"), không sao cả. Trong hướng dẫn này, bạn sẽ không xây dựng mã quá phức tạp. Bạn cũng có thể đọc cuốn sách này bằng cách đơn giản sử dụng `clang++ -std=c++2a` trên dòng lệnh.

Nếu bạn mới làm quen với C++ hiện đại, có lẽ bạn vẫn chưa hiểu đoạn mã nhỏ sau đây:

```cpp
[out = std::ref(std::cout << "Kết quả từ mã C: " << add(1, 2))](){
    out.get() << ".\n";
}();
```

Đừng lo lắng vào lúc này, chúng ta sẽ gặp lại chúng trong các chương sau.

[Mục lục](./toc.md) | [Chương trước](./00-preface.md) | [Chương tiếp theo: Cải tiến khả năng sử dụng ngôn ngữ](./02-usability.md)
## Đọc Thêm

- [A Tour of C++ (Ấn bản thứ 2) Bjarne Stroustrup](https://www.amazon.com/dp/0134997832/ref=cm_sw_em_r_mt_dp_U_GogjDbHE2H53B)
  [Lịch sử của C++](http://en.cppreference.com/w/cpp/language/history)
- [Hỗ trợ trình biên dịch C++](https://en.cppreference.com/w/cpp/compiler_support)
- [Sự không tương thích giữa ISO C và ISO C++](http://david.tribble.com/text/cdiffs.htm#C99-vs-CPP98)

## Giấy Phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy Phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy Phép Creative Commons Ghi Công-Phi Thương Mại-Không Phát Sinh 4.0 Quốc Tế</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).