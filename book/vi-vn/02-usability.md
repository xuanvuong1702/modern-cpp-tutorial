---
title: "Chương 02: Cải Tiến Khả Năng Sử Dụng Ngôn Ngữ"
type: book-vi-vn
order: 2
---

# Chương 02: Cải Tiến Khả Năng Sử Dụng Ngôn Ngữ

[TOC]

Khi chúng ta khai báo, định nghĩa một biến hoặc hằng số, và kiểm soát luồng mã,
các hàm hướng đối tượng, lập trình mẫu, v.v., trước khi chương trình chạy,
nó có thể xảy ra khi viết mã hoặc khi trình biên dịch biên dịch mã.
Để đạt được điều này, chúng ta thường nói về **khả năng sử dụng ngôn ngữ**,
điều này đề cập đến hành vi ngôn ngữ xảy ra trước khi chương trình chạy.

## 2.1 Hằng số

### nullptr

Mục đích của `nullptr` là để thay thế `NULL`. Trong ngôn ngữ C và C++ có các **hằng số con trỏ null**,
có thể được chuyển đổi ngầm định thành giá trị con trỏ null của bất kỳ kiểu con trỏ nào,
hoặc giá trị con trỏ thành viên null của bất kỳ kiểu con trỏ thành viên nào trong C++.
`NULL` được cung cấp bởi thư viện chuẩn và được định nghĩa là một hằng số con trỏ null do việc triển khai định nghĩa.
Trong C, một số thư viện chuẩn định nghĩa `NULL` là `((void*)0)` và một số định nghĩa nó là `0`.

C++ **không cho phép** chuyển đổi ngầm định `void *` sang các kiểu khác, do đó `((void*)0)` không phải là một cách triển khai hợp lệ
của `NULL`. Nếu thư viện chuẩn cố gắng định nghĩa `NULL` là `((void*)0)`, thì lỗi biên dịch sẽ xảy ra trong đoạn mã sau:

```cpp
char *ch = NULL;
```

C++ mà không có chuyển đổi ngầm định `void *` phải định nghĩa `NULL` là `0`.
Điều này vẫn tạo ra một vấn đề mới. Định nghĩa `NULL` là `0` sẽ làm cho tính năng nạp chồng trong C++ trở nên khó hiểu.
Hãy xem xét hai hàm `foo` sau:

```cpp
void foo(char*);
void foo(int);
```

Khi đó, câu lệnh `foo(NULL);` sẽ gọi hàm `foo(int)`, điều này sẽ khiến mã trở nên khó hiểu.

Để giải quyết vấn đề này, C++11 đã giới thiệu từ khóa `nullptr`, được sử dụng để phân biệt rõ ràng con trỏ null và `0`. Kiểu của `nullptr` là `nullptr_t`, có thể được chuyển đổi ngầm định thành bất kỳ kiểu con trỏ hoặc con trỏ thành viên nào, và có thể so sánh bằng hoặc không bằng với chúng.

Bạn có thể thử biên dịch đoạn mã sau bằng clang++:

```cpp
#include <iostream>
#include <type_traits>

void foo(char *);
void foo(int);

int main() {
    if (std::is_same<decltype(NULL), decltype(0)>::value)
        std::cout << "NULL == 0" << std::endl;
    if (std::is_same<decltype(NULL), decltype((void*)0)>::value)
        std::cout << "NULL == (void *)0" << std::endl;
    if (std::is_same<decltype(NULL), std::nullptr_t>::value)
        std::cout << "NULL == nullptr" << std::endl;

    foo(0);          // will call foo(int)
    // foo(NULL);    // doesn't compile
    foo(nullptr);    // will call foo(char*)
    return 0;
}

void foo(char *) {
    std::cout << "foo(char*) is called" << std::endl;
}
void foo(int i) {
    std::cout << "foo(int) is called" << std::endl;
}
```

Kết quả đầu ra là:

```bash
foo(int) is called
foo(char*) is called
```

Từ kết quả đầu ra, chúng ta có thể thấy rằng `NULL` khác với `0` và `nullptr`.
Vì vậy, hãy tập thói quen sử dụng `nullptr` trực tiếp.

Ngoài ra, trong đoạn mã trên, chúng ta đã sử dụng `decltype` và
`std::is_same` là cú pháp hiện đại của C++.
Nói một cách đơn giản, `decltype` được sử dụng để suy luận kiểu,
và `std::is_same` được sử dụng để so sánh sự bằng nhau của hai kiểu.
Chúng ta sẽ thảo luận chi tiết về chúng sau trong phần [decltype](#decltype).

### constexpr

Bản thân C++ đã có khái niệm về các biểu thức hằng, chẳng hạn như 1+2, 3\*4.
Các biểu thức này luôn cho ra cùng một kết quả mà không có bất kỳ tác dụng phụ nào.
Nếu trình biên dịch có thể tối ưu hóa trực tiếp và nhúng các biểu thức này vào chương trình
tại thời điểm biên dịch, nó sẽ tăng hiệu suất của chương trình. Một ví dụ rất rõ ràng
là trong giai đoạn định nghĩa của một mảng:

```cpp
#include <iostream>
#define LEN 10

int len_foo() {
    int i = 2;
    return i;
}
constexpr int len_foo_constexpr() {
    return 5;
}

constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-1) + fibonacci(n-2);
}


int main() {
    char arr_1[10];                      // legal
    char arr_2[LEN];                     // legal

    int len = 10;
    // char arr_3[len];                  // illegal

    const int len_2 = len + 1;
    constexpr int len_2_constexpr = 1 + 2 + 3;
    // char arr_4[len_2];                // illegal, but ok for most of the compilers
    char arr_4[len_2_constexpr];         // legal

    // char arr_5[len_foo()+5];          // illegal
    char arr_6[len_foo_constexpr() + 1]; // legal

    // 1, 1, 2, 3, 5, 8, 13, 21, 34, 55
    std::cout << fibonacci(10) << std::endl;

    return 0;
}
```

Trong ví dụ trên, `char arr_4[len_2]` có thể gây nhầm lẫn vì `len_2` đã được định nghĩa là một hằng số.
Tại sao `char arr_4[len_2]` vẫn không hợp lệ?
Đó là vì độ dài của mảng trong tiêu chuẩn C++ phải là một biểu thức hằng,
và đối với `len_2`, đây là một hằng số `const`, không phải là một biểu thức hằng,
vì vậy ngay cả khi hành vi này được hầu hết các trình biên dịch hỗ trợ, nhưng nó vẫn là một hành vi không hợp lệ.
Chúng ta cần sử dụng tính năng `constexpr` được giới thiệu trong C++11, sẽ được giới thiệu tiếp theo,
để giải quyết vấn đề này; đối với `arr_5`, trước C++98, trình biên dịch không thể biết rằng `len_foo()`
thực sự trả về một hằng số tại thời điểm chạy, điều này gây ra lỗi không hợp lệ.

> Lưu ý rằng hầu hết các trình biên dịch hiện nay đều có các tối ưu hóa của riêng chúng.
> Nhiều hành vi không hợp lệ trở nên hợp lệ dưới sự tối ưu hóa của trình biên dịch.
> Nếu bạn cần tái tạo lỗi, bạn cần sử dụng phiên bản cũ của trình biên dịch.

C++11 cung cấp từ khóa `constexpr` để cho phép người dùng khai báo rõ ràng rằng hàm hoặc
hàm tạo đối tượng sẽ trở thành một biểu thức hằng tại thời điểm biên dịch.
Từ khóa này rõ ràng yêu cầu trình biên dịch xác minh rằng `len_foo`
phải là một biểu thức hằng tại thời điểm biên dịch.

Ngoài ra, hàm sử dụng `constexpr` có thể sử dụng đệ quy:

```cpp
constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-1) + fibonacci(n-2);
}
```

Bắt đầu từ C++14,
hàm `constexpr` có thể sử dụng các câu lệnh đơn giản như biến cục bộ,
vòng lặp và nhánh bên trong.
Ví dụ, đoạn mã sau không thể biên dịch được theo tiêu chuẩn C++11:

```cpp
constexpr int fibonacci(const int n) {
    if(n == 1) return 1;
    if(n == 2) return 1;
    return fibonacci(n-1) + fibonacci(n-2);
}
```

Để làm điều này, chúng ta có thể viết một phiên bản đơn giản hóa như sau
để hàm có thể sử dụng được từ C++11:

```cpp
constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-1) + fibonacci(n-2);
}
```

## 2.2 Biến và khởi tạo

### if-switch

Trong C++ truyền thống, việc khai báo một biến có thể khai báo một biến tạm thời `int`
mặc dù nó có thể được đặt ở bất kỳ đâu, thậm chí trong một câu lệnh `for`,
nhưng luôn không có cách nào để khai báo một biến tạm thời trong các câu lệnh `if` và `switch`.
E.g:

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    // since c++17, can be simplified by using `auto`
    const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 2);
    if (itr != vec.end()) {
        *itr = 3;
    }

    if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3);
        itr != vec.end()) {
        *itr = 4;
    }

    // should output: 1, 4, 3, 4. can be simplified using `auto`
    for (std::vector<int>::iterator element = vec.begin(); element != vec.end(); 
        ++element)
        std::cout << *element << std::endl;
}
```

Trong đoạn mã trên, chúng ta có thể thấy rằng biến `itr` được định nghĩa trong phạm vi của toàn bộ hàm `main()`,
điều này khiến chúng ta phải đổi tên biến khi cần duyệt lại toàn bộ `std::vector`.
C++17 đã loại bỏ hạn chế này để chúng ta có thể làm điều này trong câu lệnh `if` (hoặc `switch`):

```cpp
if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3);
    itr != vec.end()) {
    *itr = 4;
}
```

Nó có giống với Go không?

### Danh sách khởi tạo

Khởi tạo là một tính năng ngôn ngữ rất quan trọng, phổ biến nhất là khi đối tượng được khởi tạo.
Trong C++ truyền thống, các đối tượng khác nhau có các phương pháp khởi tạo khác nhau,
chẳng hạn như mảng thông thường, PODs (**P**lain **O**ld **D**ata,
tức là các lớp không có hàm tạo, hàm hủy và hàm ảo)
hoặc kiểu struct có thể được khởi tạo bằng `{}`,
đây là cái mà chúng ta gọi là danh sách khởi tạo.
Đối với việc khởi tạo đối tượng của lớp,
bạn cần sử dụng hàm tạo sao chép,
hoặc bạn cần sử dụng `()`.
Những phương pháp khác nhau này là đặc thù và không thể thay thế cho nhau.
Ví dụ:

```cpp
#include <iostream>
#include <vector>

class Foo {
public:
    int value_a;
    int value_b;
    Foo(int a, int b) : value_a(a), value_b(b) {}
};

int main() {
    // before C++11
    int arr[3] = {1, 2, 3};
    Foo foo(1, 2);
    std::vector<int> vec = {1, 2, 3, 4, 5};

    std::cout << "arr[0]: " << arr[0] << std::endl;
    std::cout << "foo:" << foo.value_a << ", " << foo.value_b << std::endl;
    for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << std::endl;
    }
    return 0;
}
```

Để giải quyết vấn đề này,
C++11 trước tiên liên kết khái niệm danh sách khởi tạo với kiểu dữ liệu
và gọi nó là `std::initializer_list`,
cho phép hàm tạo hoặc các hàm khác sử dụng danh sách khởi tạo
như một tham số, điều này cung cấp một cầu nối thống nhất
giữa các phương pháp khởi tạo mảng thông thường và POD,
chẳng hạn như:
```cpp
#include <initializer_list>
#include <vector>
#include <iostream>

class MagicFoo {
public:
    std::vector<int> vec;
    MagicFoo(std::initializer_list<int> list) {
        for (std::initializer_list<int>::iterator it = list.begin();
             it != list.end(); ++it)
            vec.push_back(*it);
    }
};
int main() {
    // sau C++11
    MagicFoo magicFoo = {1, 2, 3, 4, 5};

    std::cout << "magicFoo: ";
    for (std::vector<int>::iterator it = magicFoo.vec.begin(); 
        it != magicFoo.vec.end(); ++it) 
        std::cout << *it << std::endl;
}
```

Hàm khởi tạo này được gọi là hàm khởi tạo danh sách, và kiểu với hàm khởi tạo này sẽ được xử lý đặc biệt trong quá trình khởi tạo.

Ngoài việc xây dựng đối tượng, danh sách khởi tạo cũng có thể được sử dụng như một tham số chính thức của một hàm thông thường, ví dụ:

```cpp
public:
    void foo(std::initializer_list<int> list) {
        for (std::initializer_list<int>::iterator it = list.begin();
            it != list.end(); ++it) vec.push_back(*it);
    }

magicFoo.foo({6,7,8,9});
```

Thứ hai, C++11 cũng cung cấp một cú pháp thống nhất để khởi tạo các đối tượng tùy ý, chẳng hạn như:

```cpp
Foo foo2 {3, 4};
```

### Ràng buộc có cấu trúc

Ràng buộc có cấu trúc cung cấp chức năng tương tự như các giá trị trả về nhiều lần trong các ngôn ngữ khác. Trong chương về container, chúng ta sẽ học rằng C++11 đã thêm container std::tuple để xây dựng một tuple bao gồm nhiều giá trị trả về. Nhưng nhược điểm là C++11/14 không cung cấp cách đơn giản để lấy và định nghĩa các phần tử trong tuple từ tuple, mặc dù chúng ta có thể giải nén tuple bằng cách sử dụng std::tie. Nhưng chúng ta vẫn phải rất rõ ràng về số lượng đối tượng mà tuple này chứa, kiểu của từng đối tượng là gì, rất phiền phức.

C++17 hoàn thiện điều này, và ràng buộc có cấu trúc cho phép chúng ta viết mã như sau:
```cpp
#include <iostream>
#include <tuple>

std::tuple<int, double, std::string> f() {
    return std::make_tuple(1, 2.3, "456");
}

int main() {
    auto [x, y, z] = f();
    std::cout << x << ", " << y << ", " << z << std::endl;
    return 0;
}
```

Kiểu suy luận `auto` được mô tả trong phần
[suy luận kiểu auto](#auto).

## 2.3 Suy diễn kiểu

Trong C và C++ truyền thống, các kiểu của tham số phải được định nghĩa rõ ràng, điều này không giúp chúng ta viết chương trình nhanh chóng, đặc biệt khi chúng ta phải đối mặt với một số lượng lớn các kiểu mẫu phức tạp, chúng ta phải chỉ định kiểu của các biến để tiếp tục viết chương trình. Điều này không chỉ làm chậm hiệu quả phát triển của chúng ta mà còn làm cho mã trở nên dài dòng và khó đọc.

C++11 giới thiệu hai từ khóa `auto` và `decltype` để thực hiện suy diễn kiểu, cho phép trình biên dịch lo lắng về kiểu của biến. Điều này làm cho C++ trở nên giống như các ngôn ngữ lập trình hiện đại khác, theo cách mà chúng ta không cần phải lo lắng về kiểu của biến.

### auto

`auto` đã tồn tại trong C++ từ lâu, nhưng nó luôn tồn tại như một chỉ báo của kiểu lưu trữ, cùng tồn tại với `register`. Trong C++ truyền thống, nếu một biến không được khai báo là biến `register`, nó sẽ tự động được coi là biến `auto`. Và với việc `register` bị loại bỏ (được sử dụng như một từ khóa dự trữ trong C++17 và sau này, hiện tại nó không còn ý nghĩa), sự thay đổi ngữ nghĩa của `auto` là rất tự nhiên.

Một trong những ví dụ phổ biến và đáng chú ý nhất về suy diễn kiểu sử dụng `auto` là iterator. Bạn có thể thấy cách viết lặp dài dòng trong C++ truyền thống ở phần trước:

```cpp
// before C++11
// cbegin() returns vector<int>::const_iterator
// and therefore it is type vector<int>::const_iterator
for(vector<int>::const_iterator it = vec.cbegin(); it != vec.cend(); ++it)
```
Khi chúng ta có `auto`:

```cpp
#include <initializer_list>
#include <vector>
#include <iostream>

class MagicFoo {
public:
    std::vector<int> vec;
    MagicFoo(std::initializer_list<int> list) {
        for (auto it = list.begin(); it != list.end(); ++it) {
            vec.push_back(*it);
        }
    }
};

int main() {
    MagicFoo magicFoo = {1, 2, 3, 4, 5};
    std::cout << "magicFoo: ";
    for (auto it = magicFoo.vec.begin(); it != magicFoo.vec.end(); ++it) {
        std::cout << *it << ", ";
    }
    std::cout << std::endl;
    return 0;
}
```

Một số cách sử dụng phổ biến khác:

```cpp
auto i = 5;              // i là int
auto arr = new auto(10); // arr là int *
```

Từ C++ 14, `auto` thậm chí có thể được sử dụng như các tham số hàm trong các biểu thức lambda tổng quát, và chức năng này được mở rộng cho các hàm thông thường trong C++ 20. Hãy xem xét ví dụ sau:

```cpp
auto add14 = [](auto x, auto y) -> int {
    return x + y;
}

int add20(auto x, auto y) {
    return x + y;
}

auto i = 5; // kiểu int
auto j = 6; // kiểu int
std::cout << add14(i, j) << std::endl;
std::cout << add20(i, j) << std::endl;
```

> **Lưu ý**: `auto` chưa thể được sử dụng để suy luận kiểu mảng:
>
> ```cpp
> auto auto_arr2[10] = {arr};   // không hợp lệ, không thể suy luận kiểu mảng
>
> 2.6.auto.cpp:30:19: error: 'auto_arr2' được khai báo như mảng của 'auto'
>     auto auto_arr2[10] = {arr};
> ```

### decltype

Từ khóa `decltype` được sử dụng để giải quyết nhược điểm của từ khóa `auto`
chỉ có thể suy diễn kiểu của biến. Cách sử dụng của nó rất giống với `typeof`:

```cpp
decltype(exTrong đó, `std::is_same<T, U>` được sử dụng để xác định
hai kiểu `T` và `U` có bằng nhau hay không. Kết quả đầu ra là:type(x+y) z;
```
Bạn đã thấy trong ví dụ trước rằng `decltype` được sử dụng để suy luận kiểu dữ liệu. Ví dụ sau đây sẽ xác định xem các biến `x, y, z` ở trên có cùng kiểu hay không:

```cpp
if (std::is_same<decltype(x), int>::value)
    std::cout << "kiểu x == int" << std::endl;
if (std::is_same<decltype(x), float>::value)
    std::cout << "kiểu x == float" << std::endl;
if (std::is_same<decltype(x), decltype(z)>::value)
    std::cout << "kiểu z == kiểu x" << std::endl;
```

Trong đó, `std::is_same<T, U>` được sử dụng để xác định
hai kiểu `T` và `U` có bằng nhau hay không. Kết quả đầu ra là:

```
type x == int
type z == type x
```

### Suy diễn kiểu trả về

Bạn có thể nghĩ rằng liệu `auto` có thể được sử dụng để suy diễn kiểu trả về của một hàm hay không. Hãy xem xét một ví dụ về hàm cộng, mà chúng ta phải viết trong C++ truyền thống:

```cpp
template<typename R, typename T, typename U>
R add(T x, U y) {
    return x+y;
}
```

> Lưu ý: Không có sự khác biệt giữa typename và class trong danh sách tham số của template. Trước khi từ khóa typename xuất hiện, class được sử dụng để định nghĩa các tham số của template. Tuy nhiên, khi định nghĩa một biến với [kiểu phụ thuộc lồng nhau](https://en.cppreference.com/w/cpp/language/dependent_name#The_typename_disambiguator_for_dependent_names) trong template, bạn cần sử dụng typename để loại bỏ sự mơ hồ.

Mã này rất xấu vì lập trình viên phải chỉ rõ kiểu trả về khi sử dụng hàm template này. Nhưng thực tế, chúng ta không biết hàm `add()` sẽ thực hiện loại phép toán nào và sẽ có kiểu trả về gì.

Vấn đề này đã được giải quyết trong C++11. Mặc dù bạn có thể ngay lập tức nghĩ đến việc sử dụng `decltype` để suy diễn kiểu của `x+y`, viết như sau:

```cpp
decltype(x+y) add(T x, U y)
```

Nhưng thực tế, cách viết này không thể biên dịch được. Điều này là do `x` và `y` chưa được định nghĩa khi trình biên dịch đọc decltype(x+y). Để giải quyết vấn đề này, C++11 cũng giới thiệu một kiểu trả về theo sau, sử dụng từ khóa auto để chỉ định kiểu trả về:

```cpp
template<typename T, typename U>
auto add2(T x, U y) -> decltype(x+y){
    return x + y;
}
```
Tin tốt là từ C++14, chúng ta có thể trực tiếp suy diễn giá trị trả về của một hàm thông thường, vì vậy cách viết sau đây trở nên hợp lệ:

```cpp
template<typename T, typename U>
auto add3(T x, U y){
    return x + y;
}
```

Bạn có thể kiểm tra xem việc suy diễn kiểu có chính xác hay không:

```cpp
// sau c++11
auto w = add2<int, double>(1, 2.0);
if (std::is_same<decltype(w), double>::value) {
    std::cout << "w là double: ";
}
std::cout << w << std::endl;

// sau c++14
auto q = add3<double, int>(1.0, 2);
std::cout << "q: " << q << std::endl;
```
### decltype(auto)

[`decltype(auto)`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fvi-vn%2F02-usability.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A492%2C%22character%22%3A0%7D%5D "book/vi-vn/02-usability.md") là một cách sử dụng phức tạp hơn một chút của C++14.

> Để hiểu được nó, bạn cần biết khái niệm chuyển tiếp tham số
> trong C++, mà chúng ta sẽ đề cập chi tiết trong chương
> [Cải tiến Runtime Ngôn ngữ](./03-runtime.md),
> và bạn có thể quay lại nội dung của phần này sau.

Nói một cách đơn giản, [`decltype(auto)`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fvi-vn%2F02-usability.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A492%2C%22character%22%3A0%7D%5D "book/vi-vn/02-usability.md") chủ yếu được sử dụng để suy diễn
kiểu trả về của một hàm chuyển tiếp hoặc gói,
mà không yêu cầu chúng ta phải chỉ rõ
biểu thức tham số của [`decltype`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fvi-vn%2F02-usability.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A492%2C%22character%22%3A0%7D%5D "book/vi-vn/02-usability.md").
Xem xét ví dụ sau, khi chúng ta cần bao bọc hai hàm sau:

```cpp
std::string  lookup1();
std::string& lookup2();
```

Trong C++11:

```cpp
std::string look_up_a_string_1() {
    return lookup1();
}
std::string& look_up_a_string_2() {
    return lookup2();
}
```

Với [`decltype(auto)`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fvi-vn%2F02-usability.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A492%2C%22character%22%3A0%7D%5D "book/vi-vn/02-usability.md"), chúng ta có thể để trình biên dịch thực hiện việc chuyển tiếp tham số phiền phức này:

```cpp
decltype(auto) look_up_a_string_1() {
    return lookup1();
}
decltype(auto) look_up_a_string_2() {
    return lookup2();
}
```

## 2.4 Luồng điều khiển

### if constexpr

Như chúng ta đã thấy ở đầu chương này, chúng ta biết rằng C++11 giới thiệu từ khóa `constexpr`, từ khóa này biên dịch các biểu thức hoặc hàm thành kết quả hằng. Một ý tưởng tự nhiên là nếu chúng ta giới thiệu tính năng này vào phán đoán điều kiện, để mã hoàn thành phán đoán nhánh tại thời gian biên dịch, liệu nó có thể làm cho chương trình hiệu quả hơn không? C++17 giới thiệu từ khóa `constexpr` vào câu lệnh `if`, cho phép bạn khai báo điều kiện của một biểu thức hằng trong mã của mình. Xem xét đoạn mã sau:

```cpp
#include <iostream>

template<typename T>
auto print_type_info(const T& t) {
    if constexpr (std::is_integral<T>::value) {
        return t + 1;
    } else {
        return t + 0.001;
    }
}
int main() {
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```
Tại thời gian biên dịch, mã thực tế sẽ hoạt động như sau:

```cpp
int print_type_info(const int& t) {
    return t + 1;
}
double print_type_info(const double& t) {
    return t + 0.001;
}
int main() {
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```

### Vòng lặp dựa trên phạm vi

Cuối cùng, C++11 giới thiệu một phương pháp lặp dựa trên phạm vi, và chúng ta có thể viết các vòng lặp ngắn gọn như trong Python, và chúng ta có thể đơn giản hóa ví dụ trước:
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};
    if (auto itr = std::find(vec.begin(), vec.end(), 3); itr != vec.end()) *itr = 4;
    for (auto element : vec)
        std::cout << element << std::endl; // chỉ đọc
    for (auto &element : vec) {
        element += 1;                      // có thể ghi
    }
    for (auto element : vec)
        std::cout << element << std::endl; // chỉ đọc
}
```

## 2.5 Templates

Templates trong C++ luôn là một nghệ thuật đặc biệt của ngôn ngữ này, và templates thậm chí có thể được sử dụng độc lập như một ngôn ngữ mới. Triết lý của template là đưa tất cả các vấn đề có thể xử lý tại thời gian biên dịch vào thời gian biên dịch, và chỉ xử lý những dịch vụ động cốt lõi tại thời gian chạy, để tối ưu hóa hiệu suất của thời gian chạy. Do đó, templates cũng được nhiều người coi là một trong những phép thuật đen của C++.

### Extern templates

Trong C++ truyền thống, templates chỉ được khởi tạo bởi trình biên dịch khi chúng được sử dụng. Nói cách khác, miễn là một template được định nghĩa đầy đủ xuất hiện trong mã được biên dịch trong mỗi đơn vị biên dịch (file), nó sẽ được khởi tạo. Điều này dẫn đến việc tăng thời gian biên dịch do khởi tạo lặp lại. Ngoài ra, chúng ta không có cách nào để yêu cầu trình biên dịch không kích hoạt việc khởi tạo template.

Để giải quyết vấn đề này, C++11 giới thiệu một template bên ngoài mở rộng cú pháp của trình biên dịch bắt buộc để khởi tạo một template tại một vị trí cụ thể, cho phép chúng ta rõ ràng yêu cầu trình biên dịch khi nào nên khởi tạo template:

```cpp
template class std::vector<bool>;          // bắt buộc khởi tạo
extern template class std::vector<double>; // không nên khởi tạo trong file hiện tại
```

### Dấu ">"

Trong trình biên dịch C++ truyền thống, `>>` luôn được coi là toán tử dịch phải. Nhưng thực tế chúng ta có thể dễ dàng viết mã cho template lồng nhau:

```cpp
std::vector<std::vector<int>> matrix;
```

Điều này không được biên dịch dưới trình biên dịch C++ truyền thống,
và từ C++11 trở đi, các dấu ngoặc nhọn liên tiếp trở nên hợp pháp
và có thể được biên dịch thành công.
Thậm chí cách viết sau đây cũng có thể được biên dịch:

```cpp
template<bool T>
class MagicType {
    bool magic = T;
};

// trong hàm main:
std::vector<MagicType<(1>2)>> magic; // hợp pháp, nhưng không khuyến khích
```

### Template alias kiểu

Trước khi bạn hiểu template alias kiểu, bạn cần hiểu sự khác biệt giữa "template" và "kiểu". Hãy hiểu kỹ câu này: **Templates được sử dụng để tạo ra các kiểu.** Trong C++ truyền thống, `typedef` có thể định nghĩa một tên mới cho kiểu, nhưng không có cách nào để định nghĩa một tên mới cho template. Bởi vì template không phải là một kiểu. Ví dụ:

```cpp
template<typename T, typename U>
class MagicType {
public:
    T dark;
    U magic;
};

```cpp
// không được phép
template<typename T>
typedef MagicType<std::vector<T>, std::string> FakeDarkMagic;
```

C++11 sử dụng `using` để giới thiệu cách viết sau, và đồng thời hỗ trợ hiệu quả tương tự như `typedef` truyền thống:

> Thông thường, chúng ta sử dụng `typedef` để định nghĩa cú pháp bí danh: `typedef tên gốc tên mới;`, nhưng cú pháp định nghĩa cho các bí danh như con trỏ hàm lại khác, điều này thường gây ra một mức độ khó khăn nhất định cho việc đọc trực tiếp.

```cpp
typedef int (*process)(void *);
using NewProcess = int(*)(void *);
template<typename T>
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;

int main() {
    TrueDarkMagic<bool> you;
}
```
### Variadic templates

Template luôn là một trong những **Phép Thuật Đen** độc đáo của C++.
Trong C++ truyền thống,
cả template lớp và template hàm chỉ có thể chấp nhận
một tập hợp cố định các tham số template như đã chỉ định;
C++11 đã thêm một biểu diễn mới, cho phép bất kỳ số lượng,
tham số template của bất kỳ loại nào,
và không cần phải cố định số lượng tham số khi định nghĩa.

```cpp
template<typename... Ts> class Magic;
```

Lớp template Magic có thể chấp nhận một số lượng không giới hạn các typename
như một tham số chính thức của template, ví dụ như định nghĩa sau:

```cpp
class Magic<int,
            std::vector<int>,
            std::map<std::string,
            std::vector<int>>> darkMagic;
```

Vì nó là tùy ý, một tham số template với số lượng 0 cũng có thể: `class Magic<> nothing;`.

Nếu bạn không muốn tạo ra 0 tham số template, bạn có thể tự định nghĩa ít nhất một tham số template:

```cpp
template<typename Require, typename... Args> class Magic;
```

Tham số template có độ dài biến cũng có thể được điều chỉnh trực tiếp thành hàm template.
Hàm `printf` trong C truyền thống, mặc dù cũng có thể gọi với số lượng tham số không xác định, nhưng không an toàn với lớp. Ngoài các hàm tham số có độ dài biến định nghĩa an toàn với lớp, C++11 cũng có thể làm cho các hàm giống như printf xử lý tự nhiên các đối tượng không tự chứa. Ngoài việc sử dụng `...` trong các tham số template để chỉ ra độ dài không xác định của các tham số template, các tham số hàm cũng sử dụng cùng một biểu diễn để đại diện cho các tham số có độ dài không xác định, điều này cung cấp một phương tiện thuận tiện để chúng ta đơn giản viết các hàm tham số có độ dài biến, chẳng hạn như:

```cpp
template<typename... Args> void printf(const std::string &str, Args... args);
```

Sau khi chúng ta định nghĩa các tham số template có độ dài biến,
làm thế nào để giải nén các tham số?

Trước tiên, chúng ta có thể sử dụng `sizeof...` để tính toán số lượng tham số:
```cpp
#include <iostream>
template<typename... Ts>
void magic(Ts... args) {
    std::cout << sizeof...(args) << std::endl;
}
```

Chúng ta có thể truyền bất kỳ số lượng tham số nào cho hàm `magic`:

```cpp
magic();      // 0
magic(1);     // 1
magic(1, ""); // 2
```

Thứ hai, các tham số được giải nén. Cho đến nay, không có cách đơn giản nào để xử lý
gói tham số, nhưng có hai phương pháp xử lý kinh điển:

**1. Hàm template đệ quy**

Đệ quy là một cách rất dễ nghĩ đến và là phương pháp kinh điển nhất. Phương pháp này liên tục truyền các tham số template cho hàm một cách đệ quy, do đó đạt được mục đích duyệt qua tất cả các tham số template:
```cpp
#include <iostream>
template<typename T0>
void printf1(T0 value) {
    std::cout << value << std::endl;
}
template<typename T, typename... Ts>
void printf1(T value, Ts... args) {
    std::cout << value << std::endl;
    printf1(args...);
}
int main() {
    printf1(1, 2, "123", 1.1);
    return 0;
}
```

**2. Mở rộng template tham số biến**

Bạn có thể cảm thấy rằng điều này rất phức tạp. C++17 đã thêm hỗ trợ cho việc mở rộng template tham số biến, vì vậy bạn có thể viết `printf` trong một hàm:

```cpp
template<typename T0, typename... T>
void printf2(T0 t0, T... t) {
    std::cout << t0 << std::endl;
    if constexpr (sizeof...(t) > 0) printf2(t...);
}
```
> Thực tế, đôi khi chúng ta sử dụng template tham số biến, nhưng không nhất thiết phải duyệt qua các tham số từng cái một. Chúng ta có thể sử dụng tính năng của `std::bind` và chuyển tiếp hoàn hảo để đạt được việc liên kết các hàm và tham số, từ đó đạt được mục đích gọi hàm.

**3. Mở rộng danh sách khởi tạo**

Hàm template đệ quy là phương pháp tiêu chuẩn, nhưng nhược điểm rõ ràng là bạn phải định nghĩa một hàm để kết thúc đệ quy.

Dưới đây là mô tả về phép thuật đen được mở rộng bằng cách sử dụng danh sách khởi tạo:

```cpp
template<typename T, typename... Ts>
auto printf3(T value, Ts... args) {
    std::cout << value << std::endl;
    (void) std::initializer_list<T>{([&args] {
        std::cout << args << std::endl;
    }(), value)...};
}
```

Trong đoạn mã này, danh sách khởi tạo được cung cấp trong C++11 và các thuộc tính của biểu thức Lambda (được đề cập trong phần tiếp theo) cũng được sử dụng thêm.

Bằng cách khởi tạo danh sách, `(biểu thức lambda, giá trị)...` sẽ được mở rộng. Do sự xuất hiện của biểu thức dấu phẩy, biểu thức lambda trước đó được thực thi trước, và việc xuất tham số được hoàn thành.
Để tránh cảnh báo của trình biên dịch, chúng ta có thể chuyển đổi rõ ràng `std::initializer_list` sang `void`.

### Biểu thức gấp (Fold expression)

Trong C++ 17, tính năng của tham số có độ dài biến được mở rộng thêm vào biểu thức, hãy xem xét ví dụ sau:

```cpp
#include <iostream>
template<typename ... T>
auto sum(T ... t) {
    return (t + ...);
}
int main() {
    std::cout << sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) << std::endl;
}
```

### Suy luận tham số template không kiểu

Những gì chúng ta đã đề cập ở trên chủ yếu là một dạng của tham số template: tham số template kiểu.

```cpp
template <typename T, typename U>
auto add(T t, U u) {
    return t + u;
}
```
Các tham số của template `T` và `U` là các kiểu cụ thể.
Nhưng cũng có một dạng phổ biến của tham số template cho phép các literal khác nhau
làm tham số template, tức là tham số template không kiểu:

```cpp
template <typename T, int BufSize>
class buffer_t {
public:
    T& alloc();
    void free(T& item);
private:
    T data[BufSize];
}

buffer_t<int, 100> buf; // 100 là tham số template
```

Trong dạng tham số template này, chúng ta có thể truyền `100` như một tham số cho template.
Sau khi C++11 giới thiệu tính năng suy luận kiểu, chúng ta sẽ tự nhiên hỏi, vì các tham số template ở đây.
Truyền với một literal cụ thể, liệu trình biên dịch có thể hỗ trợ chúng ta trong việc suy luận kiểu,
Bằng cách sử dụng placeholder `auto`, không còn cần phải chỉ định rõ ràng kiểu nữa?
May mắn thay, C++17 giới thiệu tính năng này, và chúng ta thực sự có thể sử dụng từ khóa `auto` để cho phép trình biên dịch hỗ trợ hoàn thành việc suy luận kiểu cụ thể.
Ví dụ:

```cpp
template <auto value> void foo() {
    std::cout << value << std::endl;
    return;
}

int main() {
    foo<10>();  // value as int
}
```
## 2.6 Hướng đối tượng

### Hàm khởi tạo ủy nhiệm

C++11 giới thiệu khái niệm hàm khởi tạo ủy nhiệm, cho phép một hàm khởi tạo gọi một hàm khởi tạo khác
trong cùng một lớp, từ đó đơn giản hóa mã:

```cpp
#include <iostream>
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() { // ủy nhiệm hàm khởi tạo Base()
        value2 = value;
    }
};

int main() {
    Base b(2);
    std::cout << b.value1 << std::endl;
    std::cout << b.value2 << std::endl;
}
```

### Hàm khởi tạo kế thừa

Trong C++ truyền thống, các hàm khởi tạo cần phải truyền các tham số từng cái một nếu chúng cần kế thừa, điều này dẫn đến sự kém hiệu quả. C++11 giới thiệu khái niệm hàm khởi tạo kế thừa bằng cách sử dụng từ khóa `using`:

```cpp
#include <iostream>
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() { // ủy nhiệm hàm khởi tạo Base()
        value2 = value;
    }
};
class Subclass : public Base {
public:
    using Base::Base; // hàm khởi tạo kế thừa
};
int main() {
    Subclass s(3);
    std::cout << s.value1 << std::endl;
    std::cout << s.value2 << std::endl;
}
```

### Ghi đè hàm ảo rõ ràng

Trong C++ truyền thống, rất dễ xảy ra lỗi khi vô tình nạp chồng các hàm ảo. Ví dụ:

```cpp
struct Base {
    virtual void foo();
};
struct SubClass: Base {
    void foo();
};
```

`SubClass::foo` có thể không phải là một lập trình viên cố gắng nạp chồng một hàm ảo, chỉ là thêm một hàm có cùng tên. Một kịch bản khác có thể xảy ra là khi hàm ảo của lớp cơ sở bị xóa, lớp con vẫn giữ hàm cũ và không còn nạp chồng hàm ảo nữa mà biến nó thành một phương thức lớp bình thường, điều này có thể gây ra hậu quả nghiêm trọng.

C++11 giới thiệu hai từ khóa `override` và `final` để ngăn chặn điều này xảy ra.

### override

Khi ghi đè một hàm ảo, việc sử dụng từ khóa `override` sẽ rõ ràng thông báo cho trình biên dịch rằng đây là một sự ghi đè, và trình biên dịch sẽ kiểm tra xem lớp cơ sở có hàm ảo với chữ ký hàm tương ứng hay không, nếu không sẽ không biên dịch:

```cpp
struct Base {
    virtual void foo(int);
};
struct SubClass: Base {
    virtual void foo(int) override; // legal
    virtual void foo(float) override; // illegal, no virtual function in super class
};
```
### final

[`final`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fvi-vn%2F02-usability.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A923%2C%22character%22%3A0%7D%5D "book/vi-vn/02-usability.md") được sử dụng để ngăn chặn lớp tiếp tục được kế thừa và để chấm dứt việc hàm ảo tiếp tục bị ghi đè.

```cpp
struct Base {
    virtual void foo() final;
};
struct SubClass1 final: Base {
}; // hợp lệ

struct SubClass2 : SubClass1 {
}; // không hợp lệ, SubClass1 có final

struct SubClass3: Base {
    void foo(); // không hợp lệ, foo có final
};
```

### Xóa hàm mặc định rõ ràng

Trong C++ truyền thống, nếu lập trình viên không cung cấp, trình biên dịch sẽ mặc định tạo ra các hàm khởi tạo mặc định, hàm sao chép, toán tử gán và hàm hủy cho đối tượng. Ngoài ra, C++ cũng định nghĩa các toán tử như `new` và `delete` cho tất cả các lớp. Phần này của hàm có thể được ghi đè khi lập trình viên cần.

Điều này đặt ra một số yêu cầu: khả năng kiểm soát chính xác việc tạo ra các hàm mặc định không thể kiểm soát được. Ví dụ, khi cấm sao chép một lớp, hàm sao chép và toán tử gán phải được khai báo là `private`. Cố gắng sử dụng các hàm không được định nghĩa này sẽ dẫn đến lỗi biên dịch hoặc liên kết, đây là một cách rất không thông thường.

Ngoài ra, hàm khởi tạo mặc định do trình biên dịch tạo ra không thể tồn tại cùng lúc với hàm khởi tạo do người dùng định nghĩa. Nếu người dùng định nghĩa bất kỳ hàm khởi tạo nào, trình biên dịch sẽ không còn tạo ra hàm khởi tạo mặc định nữa, nhưng đôi khi chúng ta muốn có cả hai hàm khởi tạo cùng lúc, điều này khá bất tiện.

C++11 cung cấp một giải pháp cho các yêu cầu trên, cho phép khai báo rõ ràng để chấp nhận hoặc từ chối các hàm đi kèm với trình biên dịch. Ví dụ:

```cpp
class Magic {
    public:
    Magic() = default; // rõ ràng cho phép trình biên dịch sử dụng hàm khởi tạo mặc định
    Magic& operator=(const Magic&) = delete; // rõ ràng từ chối hàm khởi tạo sao chép
    Magic(int magic_number);
}
```

### Kiểu liệt kê mạnh

Trong C++ truyền thống, các kiểu liệt kê không an toàn về kiểu, và các kiểu liệt kê được coi như số nguyên, điều này cho phép hai kiểu liệt kê hoàn toàn khác nhau có thể so sánh trực tiếp (mặc dù trình biên dịch có kiểm tra, nhưng không phải tất cả), ** Ngay cả tên giá trị liệt kê của các kiểu enum khác nhau trong cùng một không gian tên cũng không thể giống nhau**, điều này thường không phải là điều chúng ta mong muốn.

C++11 giới thiệu một lớp liệt kê và khai báo nó bằng cú pháp `enum class`:
```cpp
enum class new_enum : unsigned int {
    value1,
    value2,
    value3 = 100,
    value4 = 100
};
```

Kiểu liệt kê được định nghĩa như vậy sẽ đảm bảo an toàn về kiểu. Đầu tiên, nó không thể được chuyển đổi ngầm định sang số nguyên, cũng không thể so sánh với các số nguyên, và càng ít khả năng so sánh các giá trị liệt kê của các kiểu liệt kê khác nhau. Nhưng nếu các giá trị được chỉ định giống nhau giữa các giá trị liệt kê cùng kiểu, thì bạn có thể so sánh:

```cpp
if (new_enum::value3 == new_enum::value4) { // true
    std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
}
```

Trong cú pháp này, kiểu liệt kê được theo sau bởi dấu hai chấm và một từ khóa kiểu để chỉ định kiểu của giá trị liệt kê trong kiểu liệt kê, điều này cho phép chúng ta gán giá trị cho kiểu liệt kê (kiểu int được sử dụng mặc định khi không được chỉ định).

Và nếu chúng ta muốn lấy giá trị của giá trị liệt kê, chúng ta sẽ phải chuyển đổi kiểu một cách rõ ràng, nhưng chúng ta có thể nạp chồng toán tử `<<` để xuất ra, bạn có thể tham khảo đoạn mã sau:
```cpp
#include <iostream>
template<typename T>
std::ostream& operator<<(
    typename std::enable_if<std::is_enum<T>::value,
        std::ostream>::type& stream, const T& e)
{
    return stream << static_cast<typename std::underlying_type<T>::type>(e);
}
```

Tại thời điểm này, đoạn mã sau sẽ có thể biên dịch được:

```cpp
std::cout << new_enum::value3 << std::endl
```

## Kết luận

Phần này giới thiệu các cải tiến về khả năng sử dụng ngôn ngữ trong C++ hiện đại, mà tôi tin rằng là những tính năng quan trọng nhất mà hầu như ai cũng cần biết và sử dụng:

1. Suy luận kiểu tự động
2. Phạm vi cho vòng lặp
3. Danh sách khởi tạo
4. Mẫu tham số biến

## Bài tập

1. Sử dụng ràng buộc cấu trúc, hãy triển khai các hàm sau chỉ với một dòng mã:

   ```cpp
   #include <string>
   #include <map>
   #include <iostream>

   template <typename Key, typename Value, typename F>
   void update(std::map<Key, Value>& m, F foo) {
       // TODO:
   }
   int main() {
       std::map<std::string, long long int> m {
           {"a", 1},
           {"b", 2},
           {"c", 3}
       };
       update(m, [](std::string key) {
           return std::hash<std::string>{}(key);
       });
       for (auto&& [key, value] : m)
           std::cout << key << ":" << value << std::endl;
   }
   ```

2. Hãy thử triển khai một hàm để tính giá trị trung bình bằng [Fold Expression](#Fold-expression), cho phép truyền vào bất kỳ đối số nào.

> Tham khảo câu trả lời [xem tại đây](../../exercises/2).

[Table of Content](./toc.md) | [Chương trước](./01-intro.md) | [Chương tiếp theo: Cải tiến Runtime ngôn ngữ](./03-runtime.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).