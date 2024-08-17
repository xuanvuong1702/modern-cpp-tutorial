---
title: "Chương 03: Cải Tiến Thời Gian Chạy Ngôn Ngữ"
type: book-vi-vn
order: 3
---

# Chương 03: Cải Tiến Thời Gian Chạy Ngôn Ngữ

[TOC]

## 3.1 Biểu Thức Lambda

Biểu thức lambda là một trong những tính năng quan trọng nhất trong C++ hiện đại, và biểu thức lambda cung cấp một tính năng giống như hàm ẩn danh.
Hàm ẩn danh được sử dụng khi cần một hàm, nhưng bạn không muốn sử dụng tên để gọi hàm đó. Có rất nhiều, rất nhiều tình huống như thế này.
Vì vậy, hàm ẩn danh gần như là tiêu chuẩn trong các ngôn ngữ lập trình hiện đại.

### Cơ Bản

Cú pháp cơ bản của biểu thức Lambda như sau:

```
[danh sách bắt] (danh sách tham số) mutable(tùy chọn) exception attribute -> kiểu trả về {
// thân hàm
}
```

Các quy tắc ngữ pháp trên đều dễ hiểu ngoại trừ phần trong `[danh sách bắt]`,
ngoại trừ việc tên hàm của hàm thông thường bị lược bỏ.
Giá trị trả về có dạng `->`
(chúng ta đã đề cập đến điều này trong phần kiểu trả về đuôi ở phần trước).

Cái gọi là danh sách bắt có thể được hiểu như một loại tham số.
Thân hàm bên trong của một biểu thức lambda không thể sử dụng các biến bên ngoài
thân hàm theo mặc định.
Lúc này, danh sách bắt có thể dùng để chuyển dữ liệu bên ngoài vào.
Theo hành vi truyền vào,
danh sách bắt cũng được chia thành các loại sau:

#### 1. Bắt theo giá trị

Tương tự như việc truyền tham số, bắt theo giá trị dựa trên việc
biến có thể được sao chép, ngoại trừ việc biến được bắt sẽ được sao chép
khi biểu thức lambda được tạo ra, không phải khi nó được gọi:
```cpp
void lambda_value_capture() {
    int value = 1;
    auto copy_value = [value] {
        return value;
    };
    value = 100;
    auto stored_value = copy_value();
    std::cout << "stored_value = " << stored_value << std::endl;
    // Tại thời điểm này, stored_value == 1, và value == 100.
    // Bởi vì copy_value đã sao chép khi nó được tạo ra.
}
```

#### 2. Bắt theo tham chiếu

Tương tự như việc truyền tham chiếu, bắt theo tham chiếu lưu tham chiếu và giá trị thay đổi.

```cpp
void lambda_reference_capture() {
    int value = 1;
    auto copy_value = [&value] {
        return value;
    };
    value = 100;
    auto stored_value = copy_value();
    std::cout << "stored_value = " << stored_value << std::endl;
    // Tại thời điểm này, stored_value == 100, value == 100.
    // Bởi vì copy_value lưu tham chiếu
}
```
#### 3. Bắt ngầm định

Việc viết danh sách bắt thủ công đôi khi rất phức tạp.
Công việc cơ học này có thể được xử lý bởi trình biên dịch.
Lúc này, bạn có thể viết `&` hoặc `=` để trình biên dịch
tuyên bố bắt theo tham chiếu hoặc giá trị.

Tóm lại, bắt cung cấp khả năng cho các biểu thức lambda
sử dụng các giá trị bên ngoài. Bốn hình thức phổ biến nhất của
danh sách bắt có thể là:

- \[\] danh sách bắt rỗng
- \[name1, name2, ...\] bắt một loạt các biến
- \[&\] bắt theo tham chiếu, xác định danh sách bắt theo tham chiếu từ các biến được sử dụng trong thân hàm
- \[=\] bắt theo giá trị, xác định danh sách bắt theo giá trị từ các biến được sử dụng trong thân hàm

#### 4. Bắt biểu thức

> Phần này cần hiểu về tham chiếu rvalue và con trỏ thông minh sẽ được đề cập sau.

Các bắt giá trị và bắt tham chiếu được đề cập ở trên là các biến đã được
khai báo trong phạm vi bên ngoài, vì vậy các phương pháp bắt này bắt lvalue
và không bắt rvalue.

C++14 mang lại cho chúng ta sự tiện lợi khi cho phép các thành viên được bắt được khởi tạo
với các biểu thức tùy ý, điều này cho phép bắt các rvalue.
Kiểu của biến được bắt được khai báo được xác định theo biểu thức,
và việc xác định giống như sử dụng `auto`:

```cpp
#include <iostream>
#include <memory>  // std::make_unique
#include <utility> // std::move

void lambda_expression_capture() {
    auto important = std::make_unique<int>(1);
    auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int {
        return x+y+v1+(*v2);
    };
    std::cout << add(3,4) << std::endl;
}
```
Trong đoạn mã trên, `important` là một con trỏ độc quyền không thể bị bắt bởi bắt giá trị sử dụng `=`.
Lúc này, chúng ta cần chuyển nó thành rvalue và
khởi tạo nó trong biểu thức.

### Lambda Tổng Quát

Trong phần trước, chúng ta đã đề cập rằng từ khóa `auto` không thể được sử dụng
trong danh sách tham số vì nó sẽ xung đột với chức năng của template.
Nhưng biểu thức lambda không phải là hàm thông thường, nếu không có sự chỉ định thêm về danh sách tham số kiểu, biểu thức lambda không thể sử dụng template. May mắn thay, vấn đề này
chỉ tồn tại trong C++11, bắt đầu từ C++14. Các tham số chính thức của hàm lambda
có thể sử dụng từ khóa `auto` để tận dụng template tổng quát:

```cpp
void lambda_generic() {
    auto generic = [](auto x, auto y) {
        return x + y;
    };

    std::cout << generic(1, 2) << std::endl;
    std::cout << generic(1.1, 2.2) << std::endl;
}
```

## 3.2 Bao Bọc Đối Tượng Hàm

Mặc dù các tính năng này là một phần của thư viện chuẩn và không được tìm thấy trong thời gian chạy,
nhưng nó tăng cường khả năng thời gian chạy của ngôn ngữ C++.
Phần nội dung này cũng rất quan trọng, vì vậy đặt nó ở đây để giới thiệu.

### `std::function`

Bản chất của một biểu thức Lambda là một đối tượng của kiểu lớp (gọi là kiểu closure)
tương tự như kiểu đối tượng hàm (gọi là đối tượng closure).
Khi danh sách bắt của một biểu thức Lambda trống, đối tượng closure
cũng có thể được chuyển đổi thành giá trị con trỏ hàm để truyền, ví dụ:

```cpp
#include <iostream>
using foo = void(int);  // con trỏ hàm
void functional(foo f) {
    f(1);
}
int main() {
    auto f = [](int value) {
        std::cout << value << std::endl;
    };
    functional(f);  // gọi bằng con trỏ hàm
    f(1);           // gọi bằng biểu thức lambda
    return 0;
}
```

Đoạn mã trên cung cấp hai hình thức gọi khác nhau, một là gọi Lambda
như một kiểu hàm, và một là gọi trực tiếp biểu thức Lambda.
Trong C++11, các khái niệm này được thống nhất.
Kiểu đối tượng có thể được gọi được gọi chung là kiểu callable.
Kiểu này được giới thiệu bởi `std::function`.C++11 `std::function` là một bao bọc hàm tổng quát và đa hình
có thể lưu trữ, sao chép và gọi bất kỳ thực thể nào có thể được gọi.
Nó cũng là một callable hiện có trong C++. Một gói thực thể an toàn kiểu (tương đối,
việc gọi một con trỏ hàm không an toàn kiểu), nói cách khác,
là một container của các hàm. Khi chúng ta có một container cho các hàm,
chúng ta có thể dễ dàng xử lý các hàm và con trỏ hàm như các đối tượng. Ví dụ:

```cpp
#include <functional>
#include <iostream>

int foo(int para) {
    return para;
}

int main() {
    // std::function bao bọc một hàm nhận tham số kiểu int và trả về giá trị kiểu int
    std::function<int(int)> func = foo;

    int important = 10;
    std::function<int(int)> func2 = [&](int value) -> int {
        return 1 + value + important;
    };
    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}
```

### `std::bind` và `std::placeholder`

`std::bind` được sử dụng để ràng buộc các tham số của lời gọi hàm.
Nó giải quyết yêu cầu rằng chúng ta có thể không luôn có được tất cả các tham số
của một hàm cùng một lúc. Thông qua hàm này, chúng ta có thể ràng buộc một phần các tham số
của lời gọi vào hàm trước để trở thành một đối tượng mới,
và sau đó hoàn thành lời gọi sau khi các tham số đã đầy đủ. Ví dụ:

```cpp
int foo(int a, int b, int c) {
    ;
}
int main() {
    // ràng buộc tham số 1, 2 vào hàm foo,
    // và sử dụng std::placeholders::_1 làm chỗ trống cho tham số đầu tiên.
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1, 2);
    // khi gọi bindFoo, chúng ta chỉ cần một tham số còn lại
    bindFoo(1);
}
```

> **Mẹo:** Lưu ý sự kỳ diệu của từ khóa `auto`. Đôi khi chúng ta có thể không quen
> với kiểu trả về của một hàm, nhưng chúng ta có thể vượt qua vấn đề này bằng cách sử dụng `auto`.
## 3.3 Tham chiếu rvalue

Tham chiếu rvalue là một trong những tính năng quan trọng được giới thiệu bởi C++11
và có liên quan mật thiết với biểu thức Lambda. Sự xuất hiện của nó giải quyết
một số lượng lớn các vấn đề lịch sử trong C++.
Loại bỏ các chi phí thừa như `std::vector`, `std::string`,
và làm cho container đối tượng hàm `std::function` trở nên khả thi.

### lvalue, rvalue, prvalue, xvalue

Để hiểu rõ về tham chiếu rvalue, bạn cần phải nắm vững
khái niệm về lvalue và rvalue.

**lvalue, giá trị bên trái**, như tên gọi, là giá trị nằm bên trái của dấu gán.
Chính xác hơn, một lvalue là một đối tượng tồn tại lâu dài vẫn còn tồn tại sau
một biểu thức (không nhất thiết phải là biểu thức gán).

**Rvalue, giá trị bên phải**, giá trị bên phải đề cập đến đối tượng tạm thời
không còn tồn tại sau khi biểu thức kết thúc.

Trong C++11, để giới thiệu tham chiếu rvalue mạnh mẽ,
khái niệm về giá trị rvalue được chia nhỏ hơn nữa thành:
prvalue và xvalue.

**prvalue, pure rvalue**, rvalue thuần túy, hoặc là giá trị thuần túy,
như `10`, `true`; hoặc kết quả của việc đánh giá tương đương với
một giá trị thuần túy hoặc đối tượng tạm thời ẩn danh, ví dụ `1+2`.
Các biến tạm thời được trả về bởi các hàm không tham chiếu, các biến tạm thời được tạo ra
bởi các biểu thức toán học, các giá trị nguyên thủy, và các biểu thức Lambda
đều là các giá trị rvalue thuần túy.

Lưu ý rằng một giá trị nguyên thủy (trừ giá trị nguyên thủy chuỗi) là một prvalue. Tuy nhiên, một giá trị nguyên thủy chuỗi là một lvalue với kiểu mảng `const char`. Xem xét các ví dụ sau:
```cpp
#include <type_traits>

int main() {
    // Đúng. Kiểu của "01234" là const char [6], vì vậy nó là một lvalue
    const char (&left)[6] = "01234";

    // Kiểm tra thành công. Nó thực sự là const char [6]. Lưu ý rằng decltype(expr)
    // trả về lvalue reference nếu expr là một lvalue và không phải là một id-expression
    // không có dấu ngoặc hoặc một biểu thức truy cập thành viên lớp không có dấu ngoặc.
    static_assert(std::is_same<decltype("01234"), const char(&)[6]>::value, "");

    // Lỗi. "01234" là một lvalue, không thể được tham chiếu bởi một rvalue reference
    // const char (&&right)[6] = "01234";
}
```

Tuy nhiên, một mảng có thể được chuyển đổi ngầm định thành một con trỏ tương ứng. Kết quả, nếu không phải là một lvalue reference, là một rvalue (xvalue nếu kết quả là một rvalue reference, prvalue nếu không):

```cpp
const char*   p    = "01234"; // Đúng. "01234" được chuyển đổi ngầm định thành const char*
const char*&& pr   = "01234"; // Đúng. "01234" được chuyển đổi ngầm định thành const char*, là một prvalue.
// const char*& pl = "01234"; // Lỗi. Không có kiểu const char* lvalue
```

**xvalue, giá trị sắp hết hạn** là khái niệm được đề xuất bởi C++11 để giới thiệu
rvalue references (vì vậy trong C++ truyền thống, pure rvalue và rvalue là cùng một khái niệm),
một giá trị sẽ bị hủy nhưng có thể được di chuyển.

Sẽ hơi khó để hiểu xvalue,
hãy xem đoạn mã như sau:

```cpp
std::vector<int> foo() {
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}

std::vector<int> v = foo();
```
Trong đoạn mã như vậy, theo hiểu biết truyền thống,
giá trị trả về `temp` của hàm `foo` được tạo ra nội bộ
và sau đó gán cho `v`, trong khi khi `v` nhận được đối tượng này, toàn bộ `temp` được sao chép.
Và sau đó hủy `temp`, nếu `temp` này rất lớn, điều này sẽ gây ra rất nhiều
chi phí thừa (đây là vấn đề mà C++ truyền thống đã bị chỉ trích).
Trong dòng cuối cùng, `v` là lvalue, và giá trị trả về của `foo()` là
rvalue (cũng là một pure rvalue).

Tuy nhiên, `v` có thể bị bắt bởi các biến khác, và giá trị trả về được tạo bởi `foo()`
được sử dụng như một giá trị tạm thời. Một khi được sao chép bởi `v`, nó sẽ bị hủy ngay lập tức, và
không thể được lấy hoặc sửa đổi. Xvalue định nghĩa hành vi trong đó các giá trị tạm thời có thể được
xác định trong khi vẫn có thể di chuyển được.

Sau C++11, trình biên dịch đã làm một số công việc cho chúng ta, trong đó lvalue `temp`
được chuyển đổi ngầm định thành rvalue,
tương đương với `static_cast<std::vector<int> &&>(temp)`,
trong đó `v` ở đây di chuyển giá trị trả về của `foo` cục bộ.
Đây là ngữ nghĩa di chuyển mà chúng ta sẽ đề cập sau.

### Tham chiếu rvalue và tham chiếu lvalue

Để có được xvalue, bạn cần sử dụng khai báo của tham chiếu rvalue: `T &&`,
trong đó `T` là kiểu dữ liệu.
Khai báo của tham chiếu rvalue mở rộng vòng đời của giá trị tạm thời này,
và miễn là biến còn sống, xvalue sẽ tiếp tục tồn tại.

C++11 cung cấp phương thức `std::move` để chuyển đổi vô điều kiện
các tham số lvalue thành rvalues.
Với nó, chúng ta có thể dễ dàng có được một đối tượng tạm thời rvalue, ví dụ:

```cpp
#include <iostream>
#include <string>

void reference(std::string& str) {
    std::cout << "lvalue" << std::endl;
}
void reference(std::string&& str) {
    std::cout << "rvalue" << std::endl;
}

int main()
{
    std::string  lv1 = "string,";       // lv1 là một lvalue
    // std::string&& r1 = lv1;          // không hợp lệ, rvalue không thể tham chiếu đến lvalue
    std::string&& rv1 = std::move(lv1); // hợp lệ, std::move có thể chuyển đổi lvalue thành rvalue
    std::cout << rv1 << std::endl;      // string,

    const std::string& lv2 = lv1 + lv1; // hợp lệ, const lvalue reference có thể
                                        // mở rộng vòng đời của biến tạm thời
    // lv2 += "Test";                   // không hợp lệ, const ref không thể bị thay đổi
    std::cout << lv2 << std::endl;      // string,string,

    std::string&& rv2 = lv1 + lv2;      // hợp lệ, rvalue ref mở rộng vòng đời
    rv2 += "string";                    // hợp lệ, non-const reference có thể bị thay đổi
    std::cout << rv2 << std::endl;      // string,string,string,string

    reference(rv2);                     // output: lvalue

    return 0;
}
```

`rv2` tham chiếu đến một rvalue, nhưng vì nó là một tham chiếu,
`rv2` vẫn là một lvalue.

Lưu ý rằng có một vấn đề lịch sử rất thú vị ở đây,
hãy xem đoạn mã sau:

```cpp
#include <iostream>

int main() {
    // int &a = std::move(1); // không hợp lệ, non-const lvalue reference không thể tham chiếu đến rvalue
    const int &b = std::move(1); // hợp lệ, const lvalue reference có thể

    std::cout << b << std::endl;
}
```

Câu hỏi đầu tiên, tại sao không cho phép các tham chiếu không hằng liên kết với các non-lvalues?
Điều này là do có một lỗi logic trong cách tiếp cận này:

```cpp
void increase(int & v) {
    v++;
}
void foo() {
    double s = 1;
    increase(s);
}
```

Vì `int&` không thể tham chiếu đến một tham số kiểu `double`,
bạn phải tạo ra một giá trị tạm thời để giữ giá trị của `s`.
Do đó, khi `increase()` thay đổi giá trị tạm thời này,
bản thân `s` không bị thay đổi sau khi cuộc gọi hoàn tất.

Câu hỏi thứ hai, tại sao các tham chiếu hằng lại cho phép liên kết với các non-lvalues?
Lý do rất đơn giản vì Fortran cần điều đó.
### Ngữ nghĩa di chuyển

C++ truyền thống đã thiết kế khái niệm sao chép cho các đối tượng lớp
thông qua các hàm tạo sao chép và toán tử gán,
nhưng để thực hiện việc di chuyển tài nguyên,
người gọi phải sử dụng phương pháp sao chép và sau đó hủy trước,
nếu không, bạn cần tự mình triển khai giao diện của đối tượng di chuyển.
Hãy tưởng tượng việc di chuyển nhà của bạn trực tiếp đến ngôi nhà mới thay vì
sao chép mọi thứ (mua lại) đến ngôi nhà mới.
Vứt bỏ (hủy) tất cả những thứ ban đầu là một điều rất phản nhân văn.

C++ truyền thống không phân biệt giữa khái niệm "di chuyển" và "sao chép",
dẫn đến việc sao chép dữ liệu lớn, lãng phí thời gian và không gian.
Sự xuất hiện của tham chiếu rvalue giải quyết sự nhầm lẫn giữa hai khái niệm này,
ví dụ:

```cpp
#include <iostream>
class A {
public:
    int *pointer;
    A():pointer(new int(1)) {
        std::cout << "construct" << pointer << std::endl;
    }
    A(A& a):pointer(new int(*a.pointer)) {
        std::cout << "copy" << pointer << std::endl;
    } // sao chép đối tượng vô nghĩa
    A(A&& a):pointer(a.pointer) {
        a.pointer = nullptr;
        std::cout << "move" << pointer << std::endl;
    }
    ~A(){
        std::cout << "destruct" << pointer << std::endl;
        delete pointer;
    }
};
// tránh tối ưu hóa của trình biên dịch
A return_rvalue(bool test) {
    A a,b;
    if(test) return a; // tương đương với static_cast<A&&>(a);
    else return b;     // tương đương với static_cast<A&&>(b);
}
int main() {
    A obj = return_rvalue(false);
    std::cout << "obj:" << std::endl;
    std::cout << obj.pointer << std::endl;
    std::cout << *obj.pointer << std::endl;
    return 0;
}
```
Trong đoạn mã trên:

1. Đầu tiên tạo hai đối tượng `A` bên trong `return_rvalue`, và nhận được đầu ra của hai hàm tạo;
2. Sau khi hàm trả về, nó sẽ tạo ra một xvalue, được tham chiếu bởi cấu trúc di chuyển của `A` (`A(A&&)`), do đó kéo dài vòng đời, và lấy con trỏ trong rvalue và lưu nó vào `obj`. Trong quá trình này, con trỏ đến xvalue được đặt thành `nullptr`, điều này ngăn vùng nhớ bị hủy.

Điều này tránh việc sao chép vô nghĩa và cải thiện hiệu suất.
Hãy xem một ví dụ liên quan đến thư viện chuẩn:

```cpp
#include <iostream> // std::cout
#include <utility>  // std::move
#include <vector>   // std::vector
#include <string>   // std::string

int main() {

    std::string str = "Hello world.";
    std::vector<std::string> v;

    // sử dụng push_back(const T&), sao chép
    v.push_back(str);
    // "str: Hello world."
    std::cout << "str: " << str << std::endl;

    // sử dụng push_back(const T&&),
    // không sao chép chuỗi sẽ được di chuyển vào vector,
    // và do đó std::move có thể giảm chi phí sao chép
    v.push_back(std::move(str));
    // str bây giờ trống
    std::cout << "str: " << str << std::endl;

    return 0;
}
```

### Chuyển tiếp hoàn hảo

Như đã đề cập trước đó, tham chiếu rvalue của một khai báo thực sự là một lvalue.
Điều này tạo ra vấn đề khi chúng ta truyền tham số:

```cpp
#include <iostream>
#include <utility>
void reference(int& v) {
    std::cout << "lvalue reference" << std::endl;
}
void reference(int&& v) {
    std::cout << "rvalue reference" << std::endl;
}
template <typename T>
void pass(T&& v) {
    std::cout << "          truyền tham số bình thường: ";
    reference(v);
}
int main() {
    std::cout << "truyền rvalue:" << std::endl;
    pass(1);

    std::cout << "truyền lvalue:" << std::endl;
    int l = 1;
    pass(l);

    return 0;
}
```
Đối với `pass(1)`, mặc dù giá trị là rvalue, nhưng vì `v` là một tham chiếu, nó cũng là một lvalue.
Do đó, `reference(v)` sẽ gọi `reference(int&)` và xuất ra lvalue.
Đối với `pass(l)`, `l` là một lvalue, tại sao nó lại được truyền thành công vào `pass(T&&)`?

Điều này dựa trên **quy tắc sụp đổ tham chiếu**: Trong C++ truyền thống, chúng ta không thể tiếp tục tham chiếu một kiểu tham chiếu.
Tuy nhiên, C++ đã nới lỏng quy tắc này với sự xuất hiện của tham chiếu rvalue,
dẫn đến một quy tắc sụp đổ tham chiếu cho phép chúng ta tham chiếu các tham chiếu,
cả lvalue và rvalue. Nhưng tuân theo các quy tắc dưới đây:

| Kiểu tham số hàm | Kiểu tham số đối số | Kiểu tham số hàm sau suy diễn |
| :---------------: | :-----------------: | :---------------------------: |
|        T&         |     lvalue ref      |              T&               |
|        T&         |     rvalue ref      |              T&               |
|        T&&        |     lvalue ref      |              T&               |
|        T&&        |     rvalue ref      |             T&&               |

Do đó, việc sử dụng `T&&` trong một hàm mẫu có thể không tạo ra một tham chiếu rvalue, và khi một lvalue được truyền vào, một tham chiếu đến hàm này sẽ được suy diễn là một lvalue.
Chính xác hơn, **bất kể kiểu tham chiếu của tham số mẫu là gì, tham số mẫu có thể được suy diễn là kiểu tham chiếu phải** nếu và chỉ nếu kiểu đối số là tham chiếu phải.
Điều này làm cho `v` truyền thành công các lvalue.

Chuyển tiếp hoàn hảo dựa trên các quy tắc trên. Chuyển tiếp hoàn hảo là để chúng ta truyền các tham số,
giữ nguyên kiểu tham số gốc (tham chiếu lvalue giữ tham chiếu lvalue, tham chiếu rvalue giữ tham chiếu rvalue).
Để giải quyết vấn đề này, chúng ta nên sử dụng `std::forward` để chuyển tiếp (truyền) các tham số:

```cpp
#include <iostream>
#include <utility>
void reference(int& v) {
    std::cout << "lvalue reference" << std::endl;
}
void reference(int&& v) {
    std::cout << "rvalue reference" << std::endl;
}
template <typename T>
void pass(T&& v) {
    std::cout << "          truyền tham số bình thường: ";
    reference(v);
    std::cout << "       std::move truyền tham số: ";
    reference(std::move(v));
    std::cout << "    std::forward truyền tham số: ";
    reference(std::forward<T>(v));
    std::cout << "static_cast<T&&> truyền tham số: ";
    reference(static_cast<T&&>(v));
}
int main() {
    std::cout << "truyền rvalue:" << std::endl;
    pass(1);

    std::cout << "truyền lvalue:" << std::endl;
    int l = 1;
    pass(l);

    return 0;
}
```

Các đầu ra là:
```
truyền rvalue:
          truyền tham số bình thường: lvalue reference
       std::move truyền tham số: rvalue reference
    std::forward truyền tham số: rvalue reference
static_cast<T&&> truyền tham số: rvalue reference
truyền lvalue:
          truyền tham số bình thường: lvalue reference
       std::move truyền tham số: rvalue reference
    std::forward truyền tham số: lvalue reference
static_cast<T&&> truyền tham số: lvalue reference
```

Bất kể tham số truyền vào là lvalue hay rvalue, tham số truyền bình thường sẽ chuyển tiếp tham số như một lvalue.
Vì vậy, `std::move` sẽ luôn chấp nhận một lvalue, điều này chuyển tiếp cuộc gọi đến `reference(int&&)` để xuất ra rvalue reference.

Chỉ có `std::forward` không gây ra bất kỳ bản sao nào thêm và **chuyển tiếp hoàn hảo** (passes) các tham số của hàm đến các hàm khác được gọi nội bộ.

`std::forward` giống như `std::move`, và không làm gì cả. `std::move` chỉ đơn giản chuyển đổi lvalue thành rvalue.
`std::forward` chỉ là một chuyển đổi đơn giản của các tham số. Từ quan điểm của hiện tượng,
`std::forward<T>(v)` giống như `static_cast<T&&>(v)`.

Độc giả có thể tò mò tại sao một câu lệnh có thể trả về giá trị cho hai loại trả về.
Hãy cùng xem nhanh cách triển khai cụ thể của `std::forward`. `std::forward` chứa hai overload:

```
truyền rvalue:
          truyền tham số bình thường: lvalue reference
       std::move truyền tham số: rvalue reference
    std::forward truyền tham số: rvalue reference
static_cast<T&&> truyền tham số: rvalue reference
truyền lvalue:
          truyền tham số bình thường: lvalue reference
       std::move truyền tham số: rvalue reference
    std::forward truyền tham số: lvalue reference
static_cast<T&&> truyền tham số: lvalue reference
```

Bất kể tham số truyền vào là lvalue hay rvalue, tham số truyền bình thường sẽ chuyển tiếp tham số như một lvalue.
Vì vậy, `std::move` sẽ luôn chấp nhận một lvalue, điều này chuyển tiếp cuộc gọi đến `reference(int&&)` để xuất ra rvalue reference.

Chỉ có `std::forward` không gây ra bất kỳ bản sao nào thêm và **chuyển tiếp hoàn hảo** (passes) các tham số của hàm đến các hàm khác được gọi nội bộ.

`std::forward` giống như `std::move`, và không làm gì cả. `std::move` chỉ đơn giản chuyển đổi lvalue thành rvalue.
`std::forward` chỉ là một chuyển đổi đơn giản của các tham số. Từ quan điểm của hiện tượng,
`std::forward<T>(v)` giống như `static_cast<T&&>(v)`.

Độc giả có thể tò mò tại sao một câu lệnh có thể trả về giá trị cho hai loại trả về.
Hãy cùng xem nhanh cách triển khai cụ thể của `std::forward`. `std::forward` chứa hai overload:

```cpp
template<typename _Tp>
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type& __t) noexcept
{ return static_cast<_Tp&&>(__t); }

template<typename _Tp>
constexpr _Tp&& forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
{
    static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
        " substituting _Tp is an lvalue reference type");
    return static_cast<_Tp&&>(__t);
}
```

Trong triển khai này, chức năng của `std::remove_reference` là loại bỏ tham chiếu trong kiểu.
Và `std::is_lvalue_reference` được sử dụng để kiểm tra xem kiểu suy diễn có đúng không, trong triển khai thứ hai của `std::forward`.
Kiểm tra rằng giá trị nhận được thực sự là một lvalue, điều này phản ánh quy tắc sụp đổ.

Khi `std::forward` chấp nhận một lvalue, `_Tp` được suy diễn thành lvalue, vì vậy giá trị trả về là lvalue; và khi nó chấp nhận rvalue,
`_Tp` được suy diễn là một rvalue reference, và dựa trên quy tắc sụp đổ, giá trị trả về trở thành rvalue của `&& + &&`.
Có thể thấy rằng nguyên tắc của `std::forward` là tận dụng sự khác biệt trong suy diễn kiểu mẫu một cách thông minh.

Tại thời điểm này, chúng ta có thể trả lời câu hỏi: Tại sao `auto&&` là cách an toàn nhất để sử dụng các câu lệnh lặp?
Bởi vì khi `auto` được đẩy vào một lvalue và rvalue reference khác nhau, sự kết hợp sụp đổ với `&&` được chuyển tiếp hoàn hảo.

## Kết luận

Chương này giới thiệu những cải tiến quan trọng nhất về runtime trong C++ hiện đại, và tôi tin rằng tất cả các tính năng được đề cập trong phần này đều đáng để biết:

Biểu thức Lambda

1. Container đối tượng hàm std::function
2. Tham chiếu rvalue

[Table of Content](./toc.md) | [Previous Chapter](./02-usability.md) | [Next Chapter: Containers](./04-containers.md)

## Đọc thêm

- [Bjarne Stroustrup, The Design and Evolution of C++](https://www.amazon.com/Design-Evolution-C-Bjarne-Stroustrup/dp/0201543303)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).