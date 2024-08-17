---
title: "Chương 04: Containers"
type: book-vi-vn
order: 4
---

# Chương 04: Containers

[TOC]

## 4.1 Container Tuyến Tính

### `std::array`

Khi bạn thấy container này, bạn sẽ có những câu hỏi sau:

1. Tại sao lại giới thiệu `std::array` thay vì sử dụng trực tiếp `std::vector`?
2. Đã có mảng truyền thống, tại sao lại sử dụng `std::array`?

Đầu tiên, trả lời câu hỏi thứ nhất. Không giống như `std::vector`, kích thước của đối tượng `std::array` là cố định. Nếu kích thước của container là cố định, thì bạn có thể sử dụng `std::array` trước.
Ngoài ra, vì `std::vector` tự động mở rộng, khi lưu trữ một lượng lớn dữ liệu và sau đó xóa container,
container không tự động trả lại bộ nhớ tương ứng của phần tử đã xóa. Trong trường hợp này, bạn cần chạy thủ công `shrink_to_fit()` để giải phóng phần bộ nhớ này.

```cpp
std::vector<int> v;
std::cout << "size:" << v.size() << std::endl;         // output 0
std::cout << "capacity:" << v.capacity() << std::endl; // output 0

// Như bạn thấy, bộ nhớ của std::vector được quản lý tự động và
// tự động mở rộng khi cần thiết.
// Nhưng nếu không đủ không gian, bạn cần phân bổ thêm bộ nhớ,
// và việc phân bổ lại bộ nhớ thường là một thao tác tốn kém về hiệu suất.
v.push_back(1);
v.push_back(2);
v.push_back(3);
std::cout << "size:" << v.size() << std::endl;         // output 3
std::cout << "capacity:" << v.capacity() << std::endl; // output 4

// Logic tự động mở rộng ở đây rất giống với slice của Golang.
v.push_back(4);
v.push_back(5);
std::cout << "size:" << v.size() << std::endl;         // output 5
std::cout << "capacity:" << v.capacity() << std::endl; // output 8

// Như có thể thấy dưới đây, mặc dù container đã xóa các phần tử,
// bộ nhớ của các phần tử đã xóa không được trả lại.
v.clear();
std::cout << "size:" << v.size() << std::endl;         // output 0
std::cout << "capacity:" << v.capacity() << std::endl; // output 8

// Bộ nhớ bổ sung có thể được trả lại hệ thống thông qua lệnh shrink_to_fit()
v.shrink_to_fit();
std::cout << "size:" << v.size() << std::endl;         // output 0
std::cout << "capacity:" << v.capacity() << std::endl; // output 0
```

Vấn đề thứ hai đơn giản hơn nhiều. Sử dụng `std::array` có thể làm cho mã trở nên "hiện đại" hơn và đóng gói một số hàm thao tác, chẳng hạn như lấy kích thước mảng và kiểm tra xem nó có rỗng không, và cũng sử dụng các thuật toán container thân thiện với tiêu chuẩn trong thư viện, chẳng hạn như `std::sort`.

Sử dụng `std::array` đơn giản như việc chỉ định kiểu và kích thước của nó:

```cpp
std::array<int, 4> arr = {1, 2, 3, 4};

arr.empty(); // kiểm tra xem container có rỗng không
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

// không hợp lệ, khác với mảng kiểu C, std::array sẽ không suy diễn thành T*
// int *arr_p = arr;
```

Khi chúng ta bắt đầu sử dụng `std::array`, không thể tránh khỏi việc chúng ta sẽ gặp phải giao diện tương thích kiểu C. Có ba cách để làm điều này:

```cpp
void foo(int *p, int len) {
    return;
}

std::array<int, 4> arr = {1,2,3,4};

// truyền tham số kiểu C
// foo(arr, arr.size()); // không hợp lệ, không thể chuyển đổi ngầm định
foo(&arr[0], arr.size());
foo(arr.data(), arr.size());

// sử dụng `std::sort`
std::sort(arr.begin(), arr.end());
```

### `std::forward_list`

`std::forward_list` là một container danh sách, và cách sử dụng tương tự như `std::list`, vì vậy chúng ta không dành nhiều thời gian để giới thiệu nó.

Điều cần biết là, không giống như việc triển khai danh sách liên kết kép của `std::list`, `std::forward_list` được triển khai bằng cách sử dụng danh sách liên kết đơn.
Cung cấp việc chèn phần tử với độ phức tạp `O(1)`, không hỗ trợ truy cập ngẫu nhiên nhanh (đây cũng là một đặc điểm của danh sách liên kết),
Nó cũng là container duy nhất trong thư viện chuẩn không cung cấp phương thức `size()`. Có hiệu suất sử dụng không gian cao hơn `std::list` khi không cần lặp lại hai chiều.

## 4.2 Container Không Có Thứ Tự

Chúng ta đã quen thuộc với container có thứ tự `std::map`/`std::set` trong C++ truyền thống. Các phần tử này được triển khai nội bộ bằng cây đỏ-đen.
Độ phức tạp trung bình của việc chèn và tìm kiếm là `O(log(size))`. Khi chèn một phần tử, kích thước phần tử được so sánh theo toán tử `<` và phần tử được xác định là giống nhau.
Và chọn vị trí thích hợp để chèn vào container. Khi duyệt qua các phần tử trong container này, đầu ra sẽ được duyệt từng cái một theo thứ tự của toán tử `<`.

Các phần tử trong container không có thứ tự không được sắp xếp, và nội bộ được triển khai bằng bảng băm. Độ phức tạp trung bình của việc chèn và tìm kiếm phần tử là `O(constant)`,
Có thể đạt được hiệu suất đáng kể mà không cần quan tâm đến thứ tự của các phần tử bên trong container.

C++11 giới thiệu hai container không có thứ tự: `std::unordered_map`/`std::unordered_multimap` và
`std::unordered_set`/`std::unordered_multiset`.

Cách sử dụng của chúng về cơ bản tương tự như `std::map`/`std::multimap`/`std::set`/`set::multiset` ban đầu.
Vì chúng ta đã quen thuộc với các container này, chúng ta sẽ không so sánh từng cái một. Hãy so sánh trực tiếp `std::map` và `std::unordered_map`:

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

    // duyệt qua các phần tử theo cùng cách
    std::cout << "std::unordered_map" << std::endl;
    for( const auto & n : u)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";

    std::cout << std::endl;
    std::cout << "std::map" << std::endl;
    for( const auto & n : v)
        std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";
}
```

Kết quả cuối cùng là:

```txt
std::unordered_map
Key:[2] Value:[2]
Key:[3] Value:[3]
Key:[1] Value:[1]

std::map
Key:[1] Value:[1]
Key:[2] Value:[2]
Key:[3] Value:[3]
```

## 4.3 Tuples

Các lập trình viên đã quen thuộc với Python chắc chắn sẽ biết đến khái niệm tuples. Nhìn vào các container trong C++ truyền thống, ngoại trừ `std::pair`
dường như không có cấu trúc sẵn có nào để lưu trữ các loại dữ liệu khác nhau (thường thì chúng ta sẽ tự định nghĩa cấu trúc).
Nhưng nhược điểm của `std::pair` là rõ ràng, chỉ có thể lưu trữ hai phần tử.
### Các thao tác cơ bản

Có ba hàm chính để sử dụng tuples:

1. `std::make_tuple`: tạo tuple
2. `std::get`: Lấy giá trị của một vị trí trong tuple
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

    // giải nén tuples
    std::tie(gpa, grade, name) = get_student(1);
    std::cout << "ID: 1, "
              << "GPA: "   << gpa << ", "
              << "Grade: " << grade << ", "
              << "Name: "  << name << '\n';
}
```

`std::get` Ngoài việc sử dụng hằng số để lấy đối tượng tuple, C++14 còn thêm kiểu sử dụng để lấy đối tượng trong tuples:

```cpp
std::tuple<std::string, double, double, int> t("123", 4.5, 6.7, 8);
std::cout << std::get<std::string>(t) << std::endl;
std::cout << std::get<double>(t) << std::endl; // không hợp lệ, lỗi runtime
std::cout << std::get<3>(t) << std::endl;
```

### Chỉ mục thời gian chạy

Nếu bạn suy nghĩ kỹ, bạn có thể thấy vấn đề với đoạn mã trên. `std::get<>` phụ thuộc vào hằng số thời gian biên dịch, vì vậy đoạn mã sau không hợp lệ:

```cpp
int index = 1;
std::get<index>(t);
```

Vậy bạn phải làm gì? Câu trả lời là sử dụng `std::variant<>` (được giới thiệu từ C++ 17) để cung cấp các tham số mẫu kiểu cho `variant<>`.
Bạn có thể có một `variant<>` để chứa nhiều loại biến khác nhau (trong các ngôn ngữ khác, như Python/JavaScript, v.v., như các kiểu động):

```cpp
#include <variant>
template <size_t n, typename... T>
constexpr std::variant<T...> _tuple_index(const std::tuple<T...>& tpl, size_t i) {
    if constexpr (n >= sizeof...(T))
        throw std::out_of_range("Vượt quá giới hạn.");
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

Vì vậy chúng ta có thể:

```cpp
int i = 1;
std::cout << tuple_index(t, i) << std::endl;
```

### Gộp và Duyệt

Một yêu cầu phổ biến khác là gộp hai tuple, điều này có thể thực hiện với `std::tuple_cat`:

```cpp
auto new_tuple = std::tuple_cat(get_student(1), std::move(t));
```

Bạn có thể thấy ngay cách duyệt qua một tuple nhanh chóng như thế nào? Nhưng chúng ta vừa giới thiệu cách để đánh chỉ mục một `tuple` bằng một số tại thời gian chạy, thì việc duyệt trở nên đơn giản hơn.
Đầu tiên, chúng ta cần biết độ dài của một tuple, điều này có thể thực hiện như sau:

```cpp
template <typename T>
auto tuple_len(T &tpl) {
    return std::tuple_size<T>::value;
}
```

Đoạn mã sau sẽ duyệt qua tuple:

```cpp
for(int i = 0; i != tuple_len(new_tuple); ++i)
    // đánh chỉ mục tại thời gian chạy
    std::cout << tuple_index(new_tuple, i) << std::endl;
```

## Kết luận

Chương này giới thiệu ngắn gọn về các container mới trong C++ hiện đại. Cách sử dụng của chúng tương tự như các container hiện có trong C++. Nó khá đơn giản, và bạn có thể chọn các container cần sử dụng tùy theo tình huống thực tế để đạt được hiệu suất tốt hơn.

Mặc dù `std::tuple` hiệu quả, thư viện chuẩn cung cấp chức năng hạn chế và không có cách nào để đáp ứng yêu cầu về đánh chỉ mục và duyệt tại thời gian chạy. May mắn thay, chúng ta có các phương pháp khác mà chúng ta có thể tự triển khai.

[Table of Content](./toc.md) | [Previous Chapter](./03-runtime.md) | [Next Chapter: Smart Pointers and Memory Management](./05-pointers.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).