---
title: "Chương 09: Các tính năng nhỏ"
type: book-vi-vn
order: 9
---

# Chương 09: Các tính năng nhỏ

[TOC]

## 9.1 Kiểu dữ liệu mới

### `long long int`

`long long int` không phải là một kiểu dữ liệu mới được giới thiệu trong C++11.
Kiểu dữ liệu này đã được đưa vào tiêu chuẩn C99,
và được hỗ trợ bởi hầu hết các trình biên dịch.
C++11 chính thức đưa `long long int` vào thư viện chuẩn,
quy định rằng kiểu dữ liệu này phải có ít nhất 64 bit.

## 9.2 `noexcept` và các toán tử liên quan

Một trong những ưu điểm lớn của C++ so với C là C++ cung cấp
một cơ chế xử lý ngoại lệ hoàn chỉnh.
Tuy nhiên, trước C++11, hiếm khi lập trình viên sử dụng
câu lệnh khai báo ngoại lệ sau tên hàm.
Kể từ C++11, cơ chế khai báo ngoại lệ cũ đã bị thay thế,
vì vậy chúng ta sẽ không thảo luận về nó trong chương này.

C++11 đơn giản hóa việc khai báo ngoại lệ thành hai trường hợp:

1. Hàm có thể ném ra bất kỳ ngoại lệ nào.
2. Hàm không thể ném ra bất kỳ ngoại lệ nào.

Từ khóa `noexcept` được sử dụng để chỉ định rằng một hàm không thể ném
ra ngoại lệ. Ví dụ:

```cpp
void may_throw();           // Hàm có thể ném ra ngoại lệ
void no_throw() noexcept;   // Hàm không thể ném ra ngoại lệ
```

Nếu một hàm được khai báo với `noexcept` mà lại ném ra ngoại lệ,
chương trình sẽ bị kết thúc bởi hàm `std::terminate()`.

`noexcept` cũng có thể được sử dụng như một toán tử để kiểm tra xem
một biểu thức có thể ném ra ngoại lệ hay không.
Nếu biểu thức không thể ném ra ngoại lệ, toán tử này trả về `true`,
ngược lại trả về `false`.

```cpp
#include <iostream>
void may_throw() {
    throw true;
}
auto non_block_throw = []{
    may_throw();
};
void no_throw() noexcept {
    return;
}

auto block_throw = []() noexcept {
    no_throw();
};
int main()
{
    std::cout << std::boolalpha
        << "may_throw() noexcept? " << noexcept(may_throw()) << std::endl
        << "no_throw() noexcept? " << noexcept(no_throw()) << std::endl
        << "non_block_throw() noexcept? " << noexcept(non_block_throw()) << std::endl
        << "block_throw() noexcept? " << noexcept(block_throw()) << std::endl;
    return 0;
}
```

Khi được sử dụng để khai báo hàm, `noexcept` sẽ chặn mọi ngoại lệ
bị ném ra từ bên trong hàm.
Ví dụ:

```cpp
try {
    may_throw();
} catch (...) {
    std::cout << "Bắt được ngoại lệ từ may_throw()" << std::endl;
}
try {
    non_block_throw();
} catch (...) {
    std::cout << "Bắt được ngoại lệ từ non_block_throw()" << std::endl;
}
try {
    block_throw();
} catch (...) {
    std::cout << "Bắt được ngoại lệ từ block_throw()" << std::endl;
}
```

Kết quả đầu ra:

```
Bắt được ngoại lệ từ may_throw()
Bắt được ngoại lệ từ non_block_throw()
```

## 9.3 Literal

### Raw string literal

Trong C++ truyền thống, việc viết các chuỗi chứa nhiều ký tự đặc biệt
rất bất tiện.
Ví dụ, để viết một chuỗi chứa mã HTML, bạn cần phải sử dụng
rất nhiều ký tự escape.
Tương tự, để biểu diễn đường dẫn tệp tin trong Windows,
bạn cần phải viết `C:\\Path\\To\\File`.

C++11 giới thiệu raw string literal, cho phép bạn viết các chuỗi
chứa ký tự đặc biệt mà không cần phải escape.
Để khai báo raw string literal, bạn sử dụng tiền tố `R`
và đặt chuỗi trong cặp dấu ngoặc đơn `()`.
Ví dụ:

```cpp
#include <iostream>
#include <string>

int main() {
  std::string str = R"(C:\Path\To\File)";
  std::cout << str << std::endl;
  return 0;
}
```

### User-defined literal

C++11 cho phép bạn định nghĩa các literal tùy chỉnh bằng cách nạp chồng
toán tử hậu tố cho dấu ngoặc kép `""`.
Ví dụ:

```cpp
// Định nghĩa literal tùy chỉnh cho chuỗi
std::string operator"" _wow1(const char *wow1, size_t len) {
  return std::string(wow1) + "woooooooooow, amazing";
}

// Định nghĩa literal tùy chỉnh cho số nguyên
std::string operator"" _wow2(unsigned long long i) {
  return std::to_string(i) + "woooooooooow, amazing";
}

int main() {
  auto str = "abc"_wow1;
  auto num = 1_wow2;
  std::cout << str << std::endl;
  std::cout << num << std::endl;
  return 0;
}
```

C++11 hỗ trợ bốn loại literal tùy chỉnh:

1. **Số nguyên:**
   hàm nạp chồng toán tử hậu tố phải có một trong các kiểu sau:
   `unsigned long long`, `const char *` hoặc là một hàm template.
   Ví dụ trên sử dụng kiểu `unsigned long long`.

2. **Số thực:**
   hàm nạp chồng toán tử hậu tố phải có một trong các kiểu sau:
   `long double`, `const char *` hoặc là một hàm template.

3. **Chuỗi:**
   hàm nạp chồng toán tử hậu tố phải có danh sách tham số là
   `(const char *, size_t)`.

4. **Ký tự:**
   hàm nạp chồng toán tử hậu tố phải có kiểu dữ liệu là một trong các kiểu sau:
   `char`, `wchar_t`, `char16_t` hoặc `char32_t`.

## 9.4 Căn chỉnh bộ nhớ

C++11 giới thiệu hai từ khóa mới là `alignof` và `alignas`
để kiểm soát việc căn chỉnh bộ nhớ.
Toán tử `alignof` trả về kích thước căn chỉnh của một kiểu dữ liệu,
là một giá trị có kiểu `std::size_t` và phụ thuộc vào nền tảng.
C++11 cũng giới thiệu specifier `alignas` để chỉ định kích thước căn chỉnh
cho một biến hoặc kiểu dữ liệu.

Ví dụ:

```cpp
#include <iostream>

struct Storage {
  char a;
  int b;
  double c;
  long long d;
};

struct alignas(std::max_align_t) AlignasStorage {
  char a;
  int b;
  double c;
  long long d;
};

int main() {
  std::cout << alignof(Storage) << std::endl;
  std::cout << alignof(AlignasStorage) << std::endl;
  return 0;
}
```

`std::max_align_t` là một kiểu dữ liệu đặc biệt,
nó yêu cầu kích thước căn chỉnh bằng với kích thước căn chỉnh lớn nhất
của tất cả các kiểu dữ liệu cơ bản.
Trên hầu hết các nền tảng, kích thước căn chỉnh lớn nhất là của kiểu `long double`,
do đó `AlignasStorage` sẽ có kích thước căn chỉnh là 8 hoặc 16.

## Kết luận

Chương này đã giới thiệu một số tính năng nhỏ nhưng hữu ích của C++11.
Trong đó, `noexcept` là một tính năng quan trọng,
nó cho phép bạn ngăn chặn việc lan truyền ngoại lệ và giúp trình biên dịch
tối ưu hóa mã nguồn hiệu quả hơn.

[Mục lục](./toc.md) | [Chương trước](./08-filesystem.md) | [Chương tiếp theo: C++20](./10-cpp20.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).

