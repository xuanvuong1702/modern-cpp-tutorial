---
title: "Chương 04: Container"
type: book-vi-vn
order: 4
---

# Chương 04: Container

[TOC]

## 4.1 Container tuyến tính

### `std::array`

Khi bạn thấy container này, bạn sẽ có những câu hỏi sau:

1. Tại sao lại giới thiệu `std::array` thay vì sử dụng trực tiếp `std::vector`?
2. Đã có mảng truyền thống, tại sao lại sử dụng `std::array`?

Trước tiên, trả lời câu hỏi thứ nhất. Không giống như `std::vector`, kích thước của đối tượng `std::array` là cố định. Nếu kích thước của container là cố định, bạn nên ưu tiên sử dụng `std::array`.
Ngoài ra, do `std::vector` tự động mở rộng, khi lưu trữ một lượng lớn dữ liệu và sau đó xóa container,
thì container không tự động trả lại bộ nhớ tương ứng của các phần tử đã xóa. Trong trường hợp này, bạn cần phải gọi hàm `shrink_to_fit()` để giải phóng phần bộ nhớ này.

```cpp
std::vector<int> v;
std::cout << "size:" << v.size() << std::endl;         // in ra 0
std::cout << "capacity:" << v.capacity() << std::endl; // in ra 0

// Như bạn thấy, bộ nhớ của std::vector được quản lý tự động và
// tự động mở rộng khi cần thiết.
// Nhưng nếu không đủ dung lượng, nó cần phải cấp phát thêm bộ nhớ,
// và việc cấp phát lại bộ nhớ thường là một thao tác tốn kém hiệu năng.
v.push_back(1);
v.push_back(2);
v.push_back(3);
std::cout << "size:" << v.size() << std::endl;         // in ra 3
std::cout << "capacity:" << v.capacity() << std::endl; // in ra 4

// Logic tự động mở rộng ở đây rất giống với slice trong Golang.
v.push_back(4);
v.push_back(5);
std::cout << "size:" << v.size() << std::endl;         // in ra 5
std::cout << "capacity:" << v.capacity() << std::endl; // in ra 8

// Như bạn thấy, mặc dù container đã xóa các phần tử,
// nhưng bộ nhớ của các phần tử đã xóa không được trả lại.
v.clear();
std::cout << "size:" << v.size() << std::endl;         // in ra 0
std::cout << "capacity:" << v.capacity() << std::endl; // in ra 8

// Bộ nhớ bổ sung có thể được trả lại cho hệ thống bằng cách gọi hàm shrink_to_fit()
v.shrink_to_fit();
std::cout << "size:" << v.size() << std::endl;         // in ra 0
std::cout << "capacity:" << v.capacity() << std::endl; // in ra 0
```

Câu hỏi thứ hai đơn giản hơn. Sử dụng `std::array` khiến mã trông "hiện đại" hơn, đồng thời nó cung cấp một số hàm tiện ích để thao tác với mảng, chẳng hạn như lấy kích thước và kiểm tra xem mảng có rỗng hay không, `std::array` cũng tương thích với các thuật toán container trong thư viện chuẩn, ví dụ như `std::sort`.

Sử dụng `std::array` đơn giản như việc chỉ định kiểu và kích thước của nó:

```cpp
std::array<int, 4> arr = {1, 2, 3, 4};

arr.empty(); // kiểm tra xem container có rỗng hay không
arr.size();  // trả về kích thước của container

// hỗ trợ iterator
for (auto &i : arr)
{
    // ...
}

// sử dụng biểu thức lambda để sắp xếp
std::sort(arr.begin(), arr.end(), [](int a, int b) {
    return b < a;
});

// kích thước mảng phải là constexpr
constexpr int len = 4;
std::array<int, len> arr = {1, 2, 3, 4};

// không hợp lệ, khác với mảng kiểu C, std::array sẽ không được suy luận thành T*
// int *arr_p = arr;
```

Khi sử dụng `std::array`, không thể tránh khỏi việc phải tương tác với các giao diện kiểu C.
Có ba cách để làm điều này:

```cpp
void foo(int *p, int len) {
    return;
}

std::array<int, 4> arr = {1, 2, 3, 4};

// truyền tham số theo kiểu C
// foo(arr, arr.size()); // không hợp lệ, không thể chuyển đổi ngầm định
foo(&arr[0], arr.size());
foo(arr.data(), arr.size());

// sử dụng `std::sort`
std::sort(arr.begin(), arr.end());
```

### `std::forward_list`

`std::forward_list` là một container danh sách liên kết đơn, và cách sử dụng tương tự `std::list`,
vì vậy chúng ta sẽ không dành nhiều thời gian để giới thiệu nó.

Bạn cần biết rằng, không giống như `std::list` được triển khai bằng danh sách liên kết đôi,
`std::forward_list` được triển khai bằng danh sách liên kết đơn.
Nó cung cấp khả năng chèn phần tử với độ phức tạp `O(1)`, nhưng không hỗ trợ truy cập ngẫu nhiên nhanh
(đây là một đặc điểm chung của danh sách liên kết).
`std::forward_list` cũng là container duy nhất trong thư viện chuẩn không cung cấp phương thức `size()`.
Nó có hiệu quả sử dụng bộ nhớ tốt hơn `std::list` khi không yêu cầu khả năng duyệt hai chiều.

## 4.2 Container không có thứ tự

Chúng ta đã quen thuộc với các container có thứ tự `std::map`/`std::set` trong C++ truyền thống.
Bên trong, các container này được triển khai bằng cây đỏ-đen.
Độ phức tạp trung bình của các thao tác chèn và tìm kiếm là `O(log(size))`.
Khi chèn một phần tử, giá trị của phần tử được so sánh với các phần tử khác theo toán tử `<`,
và vị trí thích hợp để chèn phần tử vào container được xác định.
Khi duyệt qua các phần tử trong container này, các phần tử sẽ được in ra theo thứ tự của toán tử `<`.

Các phần tử trong container không có thứ tự không được sắp xếp, và bên trong,
chúng được triển khai bằng bảng băm.
Độ phức tạp trung bình của các thao tác chèn và tìm kiếm là `O(1)`,
giúp đạt được hiệu suất vượt trội khi thứ tự của các phần tử trong container không quan trọng.

C++11 giới thiệu bốn container không có thứ tự: `std::unordered_map`, `std::unordered_multimap`,
`std::unordered_set` và `std::unordered_multiset`.

Cách sử dụng của chúng về cơ bản tương tự như các container `std::map`, `std::multimap`,
`std::set` và `std::multiset` trong C++ truyền thống.
Do chúng ta đã quen thuộc với những container này, nên chúng ta sẽ không so sánh chúng chi tiết.
Hãy cùng so sánh trực tiếp `std::map` và `std::unordered_map`:

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <map>

int main() {
    // khởi tạo theo cùng thứ tự
    std::unordered_map<int, std::string> u = {
        {1, "1"},
        {3, "3"},
        {2, "2"}
    };
    std::map<int, std::string> v = {
        {1, "1"},
        {3, "3"},
        {2, "2"}
    };

    // duyệt qua các phần tử
    std::cout << "std::unordered_map" << std::endl;
    for (const auto & n : u)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";

    std::cout << std::endl;
    std::cout << "std::map" << std::endl;
    for (const auto & n : v)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";
}
```

Kết quả:

```
std::unordered_map
Key:[2] Value:[2]
Key:[3] Value:[3]
Key:[1] Value:[1]

std::map
Key:[1] Value:[1]
Key:[2] Value:[2]
Key:[3] Value:[3]
```

## 4.3 Tuple

Lập trình viên quen thuộc với Python chắc chắn biết đến khái niệm tuple.
Trong C++ truyền thống, ngoại trừ `std::pair` dường như không có cấu trúc sẵn có
nào để lưu trữ các kiểu dữ liệu khác nhau (thường chúng ta sẽ tự định nghĩa struct).
Nhưng điểm yếu của `std::pair` là nó chỉ có thể lưu trữ hai phần tử.

### Thao tác cơ bản

Có ba hàm cốt lõi để sử dụng tuple:

1. `std::make_tuple`: tạo tuple
2. `std::get`: lấy giá trị của một phần tử trong tuple
3. `std::tie`: giải nén tuple

```cpp
#include <tuple>
#include <iostream>

auto get_student(int id) {
    if (id == 0)
        return std::make_tuple(3.8, 'A', "John");
    if (id == 1)
        return std::make_tuple(2.9, 'C', "Jack");
    if (id == 2)
        return std::make_tuple(1.7, 'D', "Ive");

    // không được phép trả về 0 trực tiếp
    // kiểu trả về là std::tuple<double, char, std::string>
    return std::make_tuple(0.0, 'D', "null");
}

int main() {
    auto student = get_student(0);
    std::cout << "ID: 0, "
              << "GPA: "   << std::get<0>(student) << ", "
              << "Grade: " << std::get<1>(student) << ", "
              << "Name: "  << std::get<2>(student) << '\n';

    double gpa;
    char grade;
    std::string name;

    // giải nén tuple
    std::tie(gpa, grade, name) = get_student(1);
    std::cout << "ID: 1, "
              << "GPA: "   << gpa << ", "
              << "Grade: " << grade << ", "
              << "Name: "  << name << '\n';
}
```

Ngoài việc sử dụng hằng số để lấy phần tử từ tuple, C++14 bổ sung thêm khả năng
sử dụng kiểu để lấy phần tử:

```cpp
std::tuple<std::string, double, double, int> t("123", 4.5, 6.7, 8);
std::cout << std::get<std::string>(t) << std::endl;
std::cout << std::get<double>(t) << std::endl; // không hợp lệ, lỗi runtime
std::cout << std::get<3>(t) << std::endl;
```

### Truy cập phần tử bằng chỉ mục thời gian chạy

Nếu bạn suy nghĩ kỹ, bạn có thể nhận ra vấn đề trong đoạn mã trên.
`std::get<>` yêu cầu chỉ mục là một hằng số thời gian biên dịch,
vì vậy đoạn mã sau không hợp lệ:

```cpp
int index = 1;
std::get<index>(t);
```

Vậy phải làm thế nào? Câu trả lời là sử dụng `std::variant<>` (được giới thiệu
trong C++17) để cung cấp danh sách các kiểu có thể cho `variant<>`.
Bạn có thể sử dụng `variant<>` để chứa nhiều kiểu dữ liệu khác nhau
(tương tự như kiểu động trong các ngôn ngữ như Python hay JavaScript):

```cpp
#include <variant>
template <size_t n, typename... T>
constexpr std::variant<T...> _tuple_index(const std::tuple<T...>& tpl, size_t i) {
    if constexpr (n >= sizeof...(T))
        throw std::out_of_range("Chỉ mục vượt quá giới hạn.");
    if (i == n)
        return std::variant<T...>{ std::in_place_index<n>, std::get<n>(tpl) };
    return _tuple_index<(n < sizeof...(T)-1 ? n+1 : 0)>(tpl, i);
}
template <typename... T>
constexpr std::variant<T...> tuple_index(const std::tuple<T...>& tpl, size_t i) {
    return _tuple_index<0>(tpl, i);
}
template <typename T0, typename ... Ts>
std::ostream & operator<< (std::ostream & s, std::variant<T0, Ts...> const & v) {
    std::visit([&](auto && x){ s << x;}, v);
    return s;
}
```

Bây giờ chúng ta có thể:

```cpp
int i = 1;
std::cout << tuple_index(t, i) << std::endl;
```

### Nối và duyệt tuple

Một yêu cầu phổ biến khác là nối hai tuple, có thể thực hiện với `std::tuple_cat`:

```cpp
auto new_tuple = std::tuple_cat(get_student(1), std::move(t));
```

Bạn có thể hình dung ra cách duyệt qua một tuple một cách nhanh chóng?
Chúng ta vừa mới tìm hiểu cách truy cập phần tử của tuple bằng chỉ mục thời gian chạy,
việc duyệt qua tuple trở nên đơn giản hơn.
Trước tiên, chúng ta cần biết kích thước của tuple:

```cpp
template <typename T>
auto tuple_len(T &tpl) {
    return std::tuple_size<T>::value;
}
```

Bây giờ, chúng ta có thể duyệt qua tuple:

```cpp
for (int i = 0; i != tuple_len(new_tuple); ++i)
    // truy cập phần tử bằng chỉ mục thời gian chạy
    std::cout << tuple_index(new_tuple, i) << std::endl;
```

## Kết luận

Chương này giới thiệu sơ lược về các container mới trong C++ hiện đại.
Cách sử dụng của chúng tương tự như các container hiện có trong C++.
Chúng khá đơn giản, và bạn có thể chọn container phù hợp với từng trường hợp cụ thể
để có hiệu suất tốt hơn.

Mặc dù `std::tuple` rất hữu ích, nhưng thư viện chuẩn cung cấp chức năng hạn chế
và không có cách nào để đáp ứng yêu cầu truy cập và duyệt phần tử bằng chỉ mục
thời gian chạy. May mắn thay, chúng ta có thể tự triển khai các phương thức để làm điều đó.

[Mục lục](./toc.md) | [Chương trước](./03-runtime.md) | [Chương tiếp theo: Con trỏ thông minh và quản lý bộ nhớ](./05-pointers.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).
