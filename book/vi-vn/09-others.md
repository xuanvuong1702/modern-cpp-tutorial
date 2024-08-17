---
title: Chương 09 Các Tính Năng Nhỏ
type: book-vi-vn
order: 9
---

# Chương 09 Các Tính Năng Nhỏ

[TOC]

## 9.1 Kiểu Dữ Liệu Mới

### `long long int`

`long long int` không phải lần đầu tiên được giới thiệu trong C++11.
Ngay từ C99, `long long int` đã được bao gồm trong tiêu chuẩn C,
vì vậy hầu hết các trình biên dịch đã hỗ trợ nó.
C++11 hiện nay chính thức tích hợp nó vào thư viện chuẩn,
quy định một kiểu `long long int` với ít nhất 64 bit.

## 9.2 `noexcept` và Các Hoạt Động Của Nó

Một trong những lợi thế lớn của C++ so với C là
C++ tự định nghĩa một bộ cơ chế xử lý ngoại lệ hoàn chỉnh.
Tuy nhiên, trước C++11, hầu như không ai sử dụng biểu thức khai báo ngoại lệ sau tên hàm.
Bắt đầu từ C++11, cơ chế này đã bị loại bỏ,
vì vậy chúng ta sẽ không thảo luận hoặc giới thiệu cơ chế trước đó.
Cách hoạt động và cách sử dụng nó, bạn không nên chủ động tìm hiểu.

C++11 đơn giản hóa các khai báo ngoại lệ thành hai trường hợp:

1. Hàm có thể ném bất kỳ ngoại lệ nào
2. Hàm không thể ném bất kỳ ngoại lệ nào

Và sử dụng `noexcept` để giới hạn hai hành vi này, ví dụ:

```cpp
void may_throw();           // Có thể ném bất kỳ ngoại lệ nào
void no_throw() noexcept;   // Không thể ném bất kỳ ngoại lệ nào
```

Nếu một hàm được sửa đổi với `noexcept` mà ném ngoại lệ,
trình biên dịch sẽ sử dụng `std::terminate()`
để ngay lập tức kết thúc chương trình.

`noexcept` cũng có thể được sử dụng như một toán tử để thao tác một biểu thức.
Khi biểu thức không có ngoại lệ, nó trả về `true`,
ngược lại, nó trả về `false`.

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
        << "lmay_throw() noexcept? " << noexcept(non_block_throw()) << std::endl
        << "lno_throw() noexcept? " << noexcept(block_throw()) << std::endl;
    return 0;
}
```

`noexcept` có thể thay đổi chức năng của việc chặn ngoại lệ
sau khi sửa đổi một hàm. Nếu một ngoại lệ được tạo ra bên trong,
bên ngoài sẽ không kích hoạt. Ví dụ:

```cpp
try {
    may_throw();
} catch (...) {
    std::cout << "bắt được ngoại lệ từ may_throw()" << std::endl;
}
try {
    non_block_throw();
} catch (...) {
    std::cout << "bắt được ngoại lệ từ non_block_throw()" << std::endl;
}
try {
    block_throw();
} catch (...) {
    std::cout << "bắt được ngoại lệ từ block_throw()" << std::endl;
}
```

Kết quả đầu ra cuối cùng là:

```
bắt được ngoại lệ từ may_throw()
bắt được ngoại lệ từ non_block_throw()
```

## 9.3 Literal

### Raw String Literal

Trong C++ truyền thống, việc viết một chuỗi đầy đủ các ký tự đặc biệt
rất khó khăn. Ví dụ, một chuỗi chứa HTML ontology
cần phải thêm một số lượng lớn các ký tự thoát.
Ví dụ, một đường dẫn tệp trên Windows thường như sau: `C:\\Path\\To\\File`.

C++11 cung cấp các literal chuỗi gốc,
có thể được trang trí với `R` ở phía trước một chuỗi,
và chuỗi gốc được bao bọc trong dấu ngoặc đơn, ví dụ:

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = R"(C:\Path\To\File)";
    std::cout << str << std::endl;
    return 0;
}
```

### Literal Tùy Chỉnh

C++11 giới thiệu khả năng tùy chỉnh literals bằng cách
nạp chồng toán tử hậu tố dấu ngoặc kép:

```cpp
// Tùy chỉnh literal chuỗi phải được đặt theo danh sách tham số sau
std::string operator"" _wow1(const char *wow1, size_t len) {
    return std::string(wow1)+"woooooooooow, amazing";
}

std::string operator"" _wow2 (unsigned long long i) {
    return std::to_string(i)+"woooooooooow, amazing";
}

int main() {
    auto str = "abc"_wow1;
    auto num = 1_wow2;
    std::cout << str << std::endl;
    std::cout << num << std::endl;
    return 0;
}
```

Literal tùy chỉnh hỗ trợ bốn loại literals:

1. Literal số nguyên: Khi nạp chồng, bạn phải sử dụng `unsigned long long`, `const char *`, và các tham số toán tử literal mẫu. Loại đầu tiên được sử dụng trong mã trên;
2. Literal số thực: Bạn phải sử dụng `long double`, `const char *`, và các literal mẫu khi nạp chồng;
3. Literal chuỗi: Một bảng tham số dạng `(const char *, size_t)` phải được sử dụng;
4. Literal ký tự: Tham số chỉ có thể là `char`, `wchar_t`, `char16_t`, `char32_t`.

## 9.4 Căn Chỉnh Bộ Nhớ

C++ 11 giới thiệu hai từ khóa mới, `alignof` và `alignas`, để hỗ trợ kiểm soát căn chỉnh bộ nhớ.
Từ khóa `alignof` có thể lấy giá trị phụ thuộc vào nền tảng của kiểu `std::size_t` để truy vấn căn chỉnh của nền tảng.
Tất nhiên, đôi khi chúng ta không hài lòng với điều này và thậm chí muốn tùy chỉnh căn chỉnh của cấu trúc. Tương tự, C++ 11 giới thiệu `alignas`.
Để định hình lại căn chỉnh của một cấu trúc. Hãy xem hai ví dụ sau:

```cpp
#include <iostream>

struct Storage {
    char      a;
    int       b;
    double    c;
    long long d;
};

struct alignas(std::max_align_t) AlignasStorage {
    char      a;
    int       b;
    double    c;
    long long d;
};

int main() {
    std::cout << alignof(Storage) << std::endl;
    std::cout << alignof(AlignasStorage) << std::endl;
    return 0;
}
```

trong đó `std::max_align_t` yêu cầu cùng một căn chỉnh cho mỗi kiểu số học, vì vậy nó hầu như không có sự khác biệt trong các kiểu số học lớn nhất.
Ngược lại, kết quả trên hầu hết các nền tảng là `long double`, vì vậy yêu cầu căn chỉnh cho `AlignasStorage` mà chúng ta nhận được ở đây là 8 hoặc 16.

## Kết Luận

Một số tính năng được giới thiệu trong phần này là những tính năng
sử dụng các tính năng hiện đại của C++ thường xuyên hơn mà
chưa được giới thiệu. `noexcept` là tính năng quan trọng nhất.
Một trong những tính năng của nó là ngăn chặn sự lan truyền của các ngoại lệ,
hiệu quả cho phép trình biên dịch tối ưu hóa mã của chúng ta đến mức tối đa.

[Table of Content](./toc.md) | [Previous Chapter](./08-filesystem.md) | [Next Chapter: Outlook: Introduction of C++20](./10-cpp20.md)

## Giấy Phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy Phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy Phép Creative Commons Ghi Công-Phi Thương Mại-Không Phái Sinh 4.0 Quốc Tế</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).