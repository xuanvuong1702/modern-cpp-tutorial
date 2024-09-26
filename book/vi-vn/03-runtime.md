---
title: "Chương 03: Cải Tiến Thời Gian Chạy Ngôn Ngữ"
type: book-vi-vn
order: 3
---

# Chương 03: Cải tiến thời gian chạy của ngôn ngữ

[TOC]

## 3.1 Biểu thức Lambda

Biểu thức lambda là một trong những tính năng quan trọng nhất trong C++ hiện đại, và biểu thức lambda cung cấp tính năng tương tự như hàm ẩn danh.
Hàm ẩn danh được sử dụng khi cần một hàm, nhưng bạn không muốn sử dụng tên để gọi hàm đó. Có rất nhiều tình huống như thế này.
Vì vậy, hàm ẩn danh gần như là tiêu chuẩn trong các ngôn ngữ lập trình hiện đại.

### Cơ bản

Cú pháp cơ bản của biểu thức Lambda như sau:

```
[danh sách bắt] (danh sách tham số) mutable(tùy chọn) thuộc tính ngoại lệ -> kiểu trả về {
// thân hàm
}
```

Các quy tắc ngữ pháp trên đều dễ hiểu ngoại trừ phần `[danh sách bắt]`,
ngoại trừ việc tên hàm của hàm thông thường bị lược bỏ.
Kiểu trả về được chỉ định sau dấu `->`
(chúng ta đã đề cập đến điều này trong phần kiểu trả về theo sau ở chương trước).

Cái gọi là danh sách bắt có thể được hiểu như một loại tham số.
Thân hàm bên trong của một biểu thức lambda không thể sử dụng các biến bên ngoài
thân hàm theo mặc định.
Lúc này, danh sách bắt có thể dùng để chuyển dữ liệu bên ngoài vào.
Tùy theo hành vi truyền vào,
danh sách bắt được chia thành các loại sau:

#### 1. Bắt giá trị

Tương tự như truyền tham số, bắt giá trị dựa trên việc
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
    // Bởi vì copy_value đã sao chép giá trị khi nó được tạo ra.
}
```

#### 2. Bắt tham chiếu

Tương tự như truyền tham chiếu, bắt tham chiếu lưu tham chiếu và giá trị sẽ thay đổi khi biến được tham chiếu thay đổi.

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
    // Bởi vì copy_value lưu tham chiếu đến biến.
}
```

#### 3. Bắt ngầm định

Viết danh sách bắt thủ công đôi khi rất phức tạp.
Công việc máy móc này có thể được xử lý bởi trình biên dịch.
Lúc này, bạn có thể viết `&` hoặc `=` để trình biên dịch tự động
xác định danh sách bắt theo tham chiếu hoặc giá trị.

Tóm lại, bắt cung cấp khả năng cho biểu thức lambda
sử dụng các giá trị bên ngoài. Bốn hình thức phổ biến nhất của
danh sách bắt có thể là:

- `[]` danh sách bắt rỗng
- `[name1, name2, ...]` bắt một loạt các biến
- `[&]` bắt theo tham chiếu, xác định danh sách bắt theo tham chiếu từ các biến được sử dụng trong thân hàm
- `[=]` bắt theo giá trị, xác định danh sách bắt theo giá trị từ các biến được sử dụng trong thân hàm

#### 4. Bắt biểu thức

> Phần này cần hiểu về tham chiếu rvalue và con trỏ thông minh,
> những khái niệm sẽ được đề cập sau.

Bắt giá trị và bắt tham chiếu được đề cập ở trên là các biến đã được
khai báo trong phạm vi bên ngoài, vì vậy các phương thức bắt này bắt lvalue
và không bắt rvalue.

C++14 mang đến cho chúng ta sự tiện lợi khi cho phép các thành viên được bắt có thể được khởi tạo
bởi các biểu thức tùy ý, điều này cho phép bắt các rvalue.
Kiểu của biến được bắt được khai báo sẽ được suy ra từ biểu thức,
việc suy luận kiểu giống như khi sử dụng `auto`:

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

Trong đoạn mã trên, `important` là một con trỏ độc quyền, không thể bị bắt bởi bắt giá trị sử dụng `=`.
Lúc này, chúng ta cần chuyển nó thành rvalue và
khởi tạo nó trong biểu thức.

### Lambda tổng quát

Trong phần trước, chúng ta đã đề cập rằng từ khóa `auto` không thể được sử dụng
trong danh sách tham số vì nó sẽ xung đột với chức năng của template.
Nhưng biểu thức lambda không phải là hàm thông thường, nếu không có sự chỉ định rõ ràng về kiểu của các tham số, biểu thức lambda không thể sử dụng template. May mắn thay, vấn đề này
chỉ tồn tại trong C++11, bắt đầu từ C++14, các tham số hình thức của hàm lambda
có thể sử dụng từ khóa `auto` để tận dụng tính năng template tổng quát:

```cpp
void lambda_generic() {
    auto generic = [](auto x, auto y) {
        return x + y;
    };

    std::cout << generic(1, 2) << std::endl;
    std::cout << generic(1.1, 2.2) << std::endl;
}
```

## 3.2 Bao bọc đối tượng hàm

Mặc dù các tính năng được đề cập trong phần này là một phần của thư viện chuẩn và không được tìm thấy trong thời gian chạy,
nhưng chúng tăng cường khả năng thời gian chạy của ngôn ngữ C++.
Phần nội dung này cũng rất quan trọng, vì vậy chúng ta sẽ giới thiệu chúng ở đây.

### `std::function`

Bản chất của một biểu thức Lambda là một đối tượng của một kiểu lớp (được gọi là kiểu đóng)
tương tự như kiểu đối tượng hàm (được gọi là đối tượng đóng).
Khi danh sách bắt của một biểu thức Lambda trống, đối tượng đóng
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
Kiểu đối tượng có thể được gọi được gọi chung là kiểu có thể gọi.
Kiểu này được giới thiệu bởi `std::function`.

`std::function` trong C++11 là một trình bao bọc hàm tổng quát và đa hình,
các thể hiện của nó có thể lưu trữ, sao chép và gọi bất kỳ thực thể đích nào có thể gọi được.
Nó cũng là một trình bao bọc an toàn kiểu cho các thực thể có thể gọi hiện có trong C++ (ngược lại,
việc gọi một con trỏ hàm không an toàn kiểu), nói cách khác,
nó là một container chứa các hàm. Khi chúng ta có một container cho các hàm,
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
Nó giải quyết yêu cầu rằng chúng ta có thể không luôn có đủ tất cả các tham số
của một hàm cùng một lúc. Thông qua hàm này, chúng ta có thể ràng buộc một phần các tham số gọi
vào hàm trước để tạo thành một đối tượng mới,
và sau đó hoàn thành lời gọi sau khi đã có đủ tham số. Ví dụ:

```cpp
int foo(int a, int b, int c) {
    ;
}
int main() {
    // ràng buộc tham số 1, 2 vào hàm foo,
    // và sử dụng std::placeholders::_1 làm chỗ dành sẵn cho tham số đầu tiên
    auto bindFoo = std::bind(foo, std::placeholders::_1, 1, 2);
    // khi gọi bindFoo, chúng ta chỉ cần truyền vào một tham số nữa
    bindFoo(1);
}
```

> **Mẹo:** Lưu ý sự tiện lợi của từ khóa `auto`. Đôi khi chúng ta có thể không biết
> kiểu trả về của một hàm, nhưng chúng ta có thể bỏ qua vấn đề này bằng cách sử dụng `auto`.

## 3.3 Tham chiếu rvalue

Tham chiếu rvalue là một trong những tính năng quan trọng được giới thiệu trong C++11
và có liên quan mật thiết với biểu thức Lambda. Sự ra đời của nó giải quyết
rất nhiều vấn đề tồn tại từ lâu trong C++,
loại bỏ chi phí dư thừa như khi sử dụng `std::vector`, `std::string`,
và làm cho container đối tượng hàm `std::function` khả thi.

### lvalue, rvalue, prvalue, xvalue

Để hiểu rõ về tham chiếu rvalue, trước tiên bạn cần nắm vững
khái niệm về lvalue và rvalue.

**lvalue, left value, giá trị bên trái**, như tên gọi, là giá trị nằm bên trái toán tử gán.
Chính xác hơn, một lvalue là một đối tượng tồn tại lâu dài vẫn còn tồn tại sau
khi một biểu thức (không nhất thiết là biểu thức gán) kết thúc.

**rvalue, right value, giá trị bên phải**, đề cập đến đối tượng tạm thời
sẽ không còn tồn tại sau khi biểu thức kết thúc.

Trong C++11, để giới thiệu tham chiếu rvalue mạnh mẽ,
khái niệm rvalue được phân chia nhỏ hơn nữa thành:
prvalue và xvalue.

**prvalue, pure rvalue, rvalue thuần túy**, hoặc là một giá trị nguyên thủy,
ví dụ như `10`, `true`, hoặc là kết quả của một biểu thức,
tương đương với một giá trị nguyên thủy hoặc một đối tượng tạm thời ẩn danh,
ví dụ `1+2`. Các biến tạm thời được trả về bởi các hàm không phải hàm tham chiếu,
các biến tạm thời được tạo ra bởi các biểu thức toán tử, các giá trị nguyên thủy,
và các biểu thức Lambda đều là prvalue.

Lưu ý rằng một giá trị nguyên thủy (ngoại trừ giá trị nguyên thủy chuỗi) là một prvalue.
Tuy nhiên, một giá trị nguyên thủy chuỗi lại là một lvalue với kiểu mảng `const char`.
Xem xét ví dụ sau:

```cpp
#include <type_traits>

int main() {
    // Hợp lệ. Kiểu của "01234" là const char[6], vì vậy nó là một lvalue
    const char (&left)[6] = "01234";

    // Khẳng định thành công. Nó thực sự là const char[6]. Lưu ý rằng decltype(expr)
    // trả về tham chiếu lvalue nếu expr là một lvalue và không phải là biểu thức id
    // không có dấu ngoặc đơn hoặc biểu thức truy cập thành viên lớp không có dấu ngoặc đơn.
    static_assert(std::is_same<decltype("01234"), const char(&)[6]>::value, "");

    // Lỗi. "01234" là một lvalue, không thể được tham chiếu bởi một rvalue reference
    // const char (&&right)[6] = "01234";
}
```

Tuy nhiên, một mảng có thể được chuyển đổi ngầm định thành một con trỏ tương ứng.
Kết quả của việc chuyển đổi, nếu không phải là một tham chiếu lvalue, là một rvalue
(xvalue nếu kết quả là một rvalue reference, prvalue nếu không):

```cpp
const char*   p    = "01234"; // Hợp lệ. "01234" được chuyển đổi ngầm định thành const char*
const char*&& pr   = "01234"; // Hợp lệ. "01234" được chuyển đổi ngầm định thành const char*, là một prvalue
// const char*& pl = "01234"; // Lỗi. Không có kiểu lvalue const char*
```

**xvalue, expiring value, giá trị sắp hết hạn**, là một khái niệm được đề xuất bởi C++11 để giới thiệu
tham chiếu rvalue (vì vậy trong C++ truyền thống, pure rvalue và rvalue là cùng một khái niệm),
là một giá trị sẽ bị hủy nhưng có thể được di chuyển.

Sẽ hơi khó hiểu về xvalue,
hãy xem xét đoạn mã sau:

```cpp
std::vector<int> foo() {
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}

std::vector<int> v = foo();
```

Theo cách hiểu truyền thống,
giá trị trả về `temp` của hàm `foo` được tạo ra bên trong hàm
và sau đó được gán cho `v`, khi `v` nhận được đối tượng này, toàn bộ `temp` được sao chép.
Sau đó, `temp` bị hủy. Nếu `temp` rất lớn, điều này sẽ dẫn đến
chi phí sao chép rất lớn (đây là một vấn đề mà C++ truyền thống bị chỉ trích).
Trong dòng cuối cùng, `v` là lvalue, và giá trị trả về của `foo()` là
rvalue (cũng là prvalue).

Tuy nhiên, `v` có thể "bắt" được giá trị trả về bởi các biến khác,
và giá trị trả về của `foo()` được sử dụng như một giá trị tạm thời.
Một khi `v` đã sao chép giá trị trả về, nó sẽ bị hủy ngay lập tức
và không thể truy cập hoặc sửa đổi.
xvalue định nghĩa hành vi trong đó một giá trị tạm thời có thể được
nhận dạng và di chuyển.

Kể từ C++11, trình biên dịch đã thực hiện một số công việc cho chúng ta,
trong đó lvalue `temp` được chuyển đổi ngầm định thành rvalue,
tương đương với `static_cast<std::vector<int> &&>(temp)`,
lúc này, `v` sẽ di chuyển giá trị trả về bởi `foo` vào bộ nhớ của nó.
Đây là ngữ nghĩa di chuyển, một khái niệm sẽ được đề cập sau.

### Tham chiếu rvalue và tham chiếu lvalue

Để lấy xvalue, bạn cần sử dụng khai báo tham chiếu rvalue: `T &&`,
trong đó `T` là kiểu dữ liệu.
Khai báo tham chiếu rvalue kéo dài vòng đời của giá trị tạm thời,
miễn là biến tham chiếu còn tồn tại, xvalue sẽ tiếp tục tồn tại.

C++11 cung cấp hàm `std::move` để chuyển đổi vô điều kiện
các tham số lvalue thành rvalue.
Sử dụng hàm này, chúng ta có thể dễ dàng lấy một đối tượng tạm thời rvalue,
ví dụ:

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
    std::string&& rv1 = std::move(lv1); // hợp lệ, std::move chuyển lvalue thành rvalue
    std::cout << rv1 << std::endl;      // string,

    const std::string& lv2 = lv1 + lv1; // hợp lệ, tham chiếu lvalue hằng số có thể
                                        // kéo dài vòng đời của biến tạm thời
    // lv2 += "Test";                   // không hợp lệ, tham chiếu hằng số không thể bị thay đổi
    std::cout << lv2 << std::endl;      // string,string,

    std::string&& rv2 = lv1 + lv2;      // hợp lệ, tham chiếu rvalue kéo dài vòng đời
    rv2 += "string";                    // hợp lệ, tham chiếu không hằng số có thể bị thay đổi
    std::cout << rv2 << std::endl;      // string,string,string,string

    reference(rv2);                     // in ra: lvalue

    return 0;
}
```

`rv2` tham chiếu đến một rvalue, nhưng do bản thân nó là một tham chiếu,
nên `rv2` vẫn là một lvalue.

Lưu ý rằng có một vấn đề lịch sử rất thú vị, hãy xem đoạn mã sau:

```cpp
#include <iostream>

int main() {
    // int &a = std::move(1); // không hợp lệ, tham chiếu lvalue không hằng số không thể tham chiếu đến rvalue
    const int &b = std::move(1); // hợp lệ, tham chiếu lvalue hằng số có thể

    std::cout << b << std::endl;
}
```

Câu hỏi đầu tiên, tại sao không cho phép tham chiếu không hằng số liên kết với non-lvalue?
Bởi vì cách tiếp cận này có lỗi logic:

```cpp
void increase(int & v) {
    v++;
}
void foo() {
    double s = 1;
    increase(s);
}
```

Vì `int&` không thể tham chiếu đến tham số kiểu `double`,
phải tạo ra một biến tạm thời để lưu giá trị của `s`.
Do đó, khi `increase()` sửa đổi biến tạm thời này,
bản thân `s` không bị thay đổi sau khi hàm kết thúc.

Câu hỏi thứ hai, tại sao tham chiếu hằng số cho phép liên kết với non-lvalue?
Lý do đơn giản là vì Fortran cần điều đó.

### Ngữ nghĩa di chuyển

C++ truyền thống đã thiết kế khái niệm sao chép cho các đối tượng lớp
thông qua hàm tạo sao chép và toán tử gán,
nhưng để thực hiện việc di chuyển tài nguyên,
người gọi phải sử dụng phương pháp sao chép và sau đó hủy,
nếu không, bạn cần tự mình triển khai giao diện di chuyển cho đối tượng.
Hãy tưởng tượng việc di chuyển nhà của bạn trực tiếp đến ngôi nhà mới thay vì
sao chép mọi thứ (mua lại) sang ngôi nhà mới
và vứt bỏ (hủy) tất cả đồ đạc cũ, đây là một điều phi logic.

C++ truyền thống không phân biệt giữa khái niệm "di chuyển" và "sao chép",
dẫn đến việc sao chép dữ liệu với khối lượng lớn, lãng phí thời gian và không gian.
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

Phân tích đoạn mã trên:

1. Đầu tiên, hai đối tượng `A` được tạo ra bên trong hàm `return_rvalue`,
   và chúng ta thấy đầu ra của hai hàm tạo.
2. Sau khi hàm trả về, một xvalue được tạo ra, xvalue này được tham chiếu bởi
   hàm tạo di chuyển của `A` (`A(A&&)`), do đó kéo dài vòng đời,
   lấy con trỏ trong rvalue và lưu nó vào `obj`. Đồng thời,
   con trỏ trong xvalue được đặt thành `nullptr` để tránh vùng nhớ bị hủy.

Ngữ nghĩa di chuyển tránh việc sao chép đối tượng vô nghĩa và cải thiện hiệu suất.
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
    // không sao chép, chuỗi sẽ được di chuyển vào vector,
    // do đó std::move giúp giảm chi phí sao chép
    v.push_back(std::move(str));
    // str hiện tại rỗng
    std::cout << "str: " << str << std::endl;

    return 0;
}
```

### Chuyển tiếp hoàn hảo

Như đã đề cập trước đó, tham chiếu rvalue trong một khai báo bản chất là một lvalue.
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

Đối với `pass(1)`, mặc dù tham số truyền vào là rvalue, nhưng do `v` là tham chiếu, nên nó là lvalue.
Do đó, `reference(v)` sẽ gọi `reference(int&)` và in ra "lvalue".
Đối với `pass(l)`, `l` là lvalue, vậy tại sao nó được truyền thành công vào `pass(T&&)`?

Điều này dựa trên **quy tắc sụp đổ tham chiếu**: Trong C++ truyền thống, chúng ta không thể tham chiếu đến một kiểu tham chiếu.
Tuy nhiên, C++ đã nới lỏng quy tắc này với sự xuất hiện của tham chiếu rvalue,
tạo ra quy tắc sụp đổ tham chiếu cho phép chúng ta tham chiếu đến các tham chiếu,
bao gồm cả lvalue và rvalue, nhưng phải tuân theo các quy tắc sau:

| Kiểu tham số hàm | Kiểu tham số đối số | Kiểu tham số hàm sau suy luận |
| :----------------: | :----------------: | :---------------------------: |
|         T&        |    lvalue ref     |              T&               |
|         T&        |    rvalue ref     |              T&               |
|         T&&       |    lvalue ref     |              T&               |
|         T&&       |    rvalue ref     |              T&&              |

Do đó, việc sử dụng `T&&` trong hàm template có thể không tạo ra một tham chiếu rvalue, và khi một lvalue được truyền vào, tham chiếu đến hàm này sẽ được suy luận thành lvalue.
Nói chính xác hơn, **bất kể kiểu tham chiếu của tham số template là gì, tham số template chỉ có thể được suy luận thành rvalue reference** khi kiểu đối số là rvalue reference.
Điều này giúp `v` có thể truyền lvalue thành công.

Chuyển tiếp hoàn hảo dựa trên quy tắc này. Cái gọi là chuyển tiếp hoàn hảo là việc truyền tham số,
giữ nguyên kiểu tham số ban đầu (lvalue reference vẫn là lvalue reference, rvalue reference vẫn là rvalue reference).
Để thực hiện chuyển tiếp hoàn hảo, chúng ta nên sử dụng `std::forward` để chuyển tiếp (truyền) tham số:

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
    std::cout << "       truyền tham số với std::move: ";
    reference(std::move(v));
    std::cout << "    truyền tham số với std::forward: ";
    reference(std::forward<T>(v));
    std::cout << "truyền tham số với static_cast<T&&>: ";
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

Kết quả đầu ra:

```
truyền rvalue:
          truyền tham số bình thường: lvalue reference
       truyền tham số với std::move: rvalue reference
    truyền tham số với std::forward: rvalue reference
truyền tham số với static_cast<T&&>: rvalue reference
truyền lvalue:
          truyền tham số bình thường: lvalue reference
       truyền tham số với std::move: rvalue reference
    truyền tham số với std::forward: lvalue reference
truyền tham số với static_cast<T&&>: lvalue reference
```

Cho dù tham số truyền vào là lvalue hay rvalue, cách truyền thông thường
sẽ luôn chuyển tiếp tham số như một lvalue.
Do đó, `std::move` sẽ luôn nhận được lvalue và chuyển tiếp cuộc gọi
đến `reference(int&&)`, in ra "rvalue reference".

Chỉ có `std::forward` không tạo ra thêm bản sao và **chuyển tiếp hoàn hảo**
các tham số của hàm đến các hàm khác được gọi bên trong.

`std::forward` giống `std::move` ở chỗ chúng không thực sự di chuyển hay
chuyển tiếp bất cứ thứ gì. `std::move` chỉ đơn giản là chuyển đổi một lvalue
thành rvalue. `std::forward` chỉ là một phép ép kiểu đơn giản.
Xét về kết quả, `std::forward<T>(v)` tương đương với `static_cast<T&&>(v)`.

Độc giả có thể thắc mắc tại sao một câu lệnh có thể trả về giá trị cho hai
kiểu trả về khác nhau. Hãy cùng xem nhanh cách triển khai `std::forward`.
`std::forward` bao gồm hai overload:

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

Trong cách triển khai này, `std::remove_reference` có tác dụng loại bỏ tham chiếu
trong kiểu. `std::is_lvalue_reference` được sử dụng để kiểm tra xem kiểu suy luận
có chính xác hay không, trong overload thứ hai của `std::forward`,
nó kiểm tra xem giá trị nhận được có thực sự là lvalue hay không,
điều này cũng phản ánh quy tắc sụp đổ tham chiếu.

Khi `std::forward` nhận một lvalue, `_Tp` được suy luận thành lvalue,
vì vậy giá trị trả về là lvalue; và khi nó nhận một rvalue,
`_Tp` được suy luận thành rvalue reference, và theo quy tắc sụp đổ tham chiếu,
giá trị trả về trở thành rvalue (`&& + && = &&`).
Có thể thấy, nguyên tắc hoạt động của `std::forward` là tận dụng một cách
thông minh sự khác biệt trong suy luận kiểu template.

Tại thời điểm này, chúng ta có thể trả lời câu hỏi: Tại sao `auto&&` là cách an
toàn nhất để sử dụng trong câu lệnh lặp? Bởi vì khi `auto` được suy luận
thành các tham chiếu lvalue hoặc rvalue khác nhau, sự kết hợp của nó với `&&`
sẽ tạo ra chuyển tiếp hoàn hảo theo quy tắc sụp đổ tham chiếu.

## Kết luận

Chương này giới thiệu những cải tiến quan trọng nhất về thời gian chạy
trong C++ hiện đại, và tôi tin rằng tất cả các tính năng được đề cập
trong phần này đều đáng để tìm hiểu:

1. Biểu thức Lambda
2. Container chứa đối tượng hàm `std::function`
3. Tham chiếu rvalue

[Mục lục](./toc.md) | [Chương trước](./02-usability.md) | [Chương tiếp theo: Container](./04-containers.md)

## Đọc thêm

- [Bjarne Stroustrup, The Design and Evolution of C++](https://www.amazon.com/Design-Evolution-C-Bjarne-Stroustrup/dp/0201543303)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).

