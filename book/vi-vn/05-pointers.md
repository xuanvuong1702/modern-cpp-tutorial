---
title: "Chương 05 Con trỏ thông minh và Quản lý bộ nhớ"
type: book-vi-vn
order: 5
---

# Chương 05 Con trỏ thông minh và Quản lý bộ nhớ

[TOC]

## 5.1 RAII và Đếm tham chiếu

Các lập trình viên hiểu về `Objective-C`/`Swift`/`JavaScript` nên biết khái niệm đếm tham chiếu. Đếm tham chiếu được sử dụng để ngăn chặn rò rỉ bộ nhớ.
Ý tưởng cơ bản là đếm số lượng đối tượng được cấp phát động. Mỗi khi bạn thêm một tham chiếu đến cùng một đối tượng, số đếm tham chiếu của đối tượng đó sẽ tăng lên một lần.
Mỗi khi một tham chiếu bị xóa, số đếm tham chiếu sẽ giảm đi một. Khi số đếm tham chiếu của một đối tượng giảm xuống còn không, bộ nhớ heap được trỏ tới sẽ tự động bị xóa.

Trong C++ truyền thống, "nhớ" để giải phóng tài nguyên thủ công không phải lúc nào cũng là một thực hành tốt nhất. Bởi vì chúng ta có thể quên giải phóng tài nguyên và dẫn đến rò rỉ bộ nhớ.
Vì vậy, thực hành thông thường là đối với một đối tượng, chúng ta xin cấp phát không gian khi khởi tạo, và giải phóng không gian khi hủy đối tượng (được gọi khi rời khỏi phạm vi).
Đó là, chúng ta thường nói rằng việc thu nhận tài nguyên RAII là công nghệ khởi tạo tài nguyên.

Có ngoại lệ cho mọi thứ, chúng ta luôn cần cấp phát đối tượng trên bộ nhớ tự do. Trong C++ truyền thống, chúng ta phải sử dụng `new` và `delete` để "nhớ" giải phóng tài nguyên. C++11 giới thiệu khái niệm con trỏ thông minh, sử dụng ý tưởng đếm tham chiếu để lập trình viên không còn cần quan tâm đến việc giải phóng bộ nhớ thủ công.
Những con trỏ thông minh này bao gồm `std::shared_ptr`/`std::unique_ptr`/`std::weak_ptr`, cần bao gồm tệp tiêu đề `<memory>`.

> Lưu ý: Đếm tham chiếu không phải là thu gom rác. Đếm tham chiếu có thể thu hồi các đối tượng không còn được sử dụng càng sớm càng tốt, và sẽ không gây ra chờ đợi lâu trong quá trình thu hồi.
> Rõ ràng hơn và chỉ ra vòng đời của tài nguyên.

## 5.2 `std::shared_ptr`

`std::shared_ptr` là một con trỏ thông minh ghi lại bao nhiêu `shared_ptr` trỏ đến một đối tượng, loại bỏ việc gọi `delete`, tự động xóa đối tượng khi số đếm tham chiếu trở về không.

Nhưng điều đó chưa đủ, vì sử dụng `std::shared_ptr` vẫn cần được gọi với `new`, điều này làm cho mã có một mức độ không đối xứng nhất định.

`std::make_shared` có thể được sử dụng để loại bỏ việc sử dụng rõ ràng của `new`, vì vậy `std::make_shared` sẽ cấp phát các đối tượng trong các tham số được tạo ra.
Và trả về con trỏ `std::shared_ptr` của loại đối tượng này. Ví dụ:
```cpp
#include <iostream>
#include <memory>
void foo(std::shared_ptr<int> i) {
    (*i)++;
}
int main() {
    // auto pointer = new int(10); // không hợp lệ, không gán trực tiếp
    // Tạo một std::shared_ptr
    auto pointer = std::make_shared<int>(10);
    foo(pointer);
    std::cout << *pointer << std::endl; // 11
    // shared_ptr sẽ được hủy trước khi rời khỏi phạm vi
    return 0;
}
```

`std::shared_ptr` có thể lấy con trỏ thô thông qua phương thức `get()` và giảm số lượng tham chiếu bằng `reset()`.
Và xem số lượng tham chiếu của một đối tượng bằng `use_count()`. Ví dụ:

```cpp
auto pointer = std::make_shared<int>(10);
auto pointer2 = pointer; // số lượng tham chiếu +1
auto pointer3 = pointer; // số lượng tham chiếu +1
int *p = pointer.get();  // không tăng số lượng tham chiếu

std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;   // 3
std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 3
std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 3

pointer2.reset();
std::cout << "reset pointer2:" << std::endl;

std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;   // 2
std::cout << "pointer2.use_count() = " 
    << pointer2.use_count() << std::endl;                // pointer2 đã reset, 0
std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl; // 2

pointer3.reset();
std::cout << "reset pointer3:" << std::endl;

std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;   // 1
std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl; // 0
std::cout << "pointer3.use_count() = " 
    << pointer3.use_count() << std::endl;                // pointer3 đã reset, 0
```

## 5.3 [`std::unique_ptr`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fen-us%2F05-pointers.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A82%2C%22character%22%3A0%7D%5D "book/en-us/05-pointers.md")

[`std::unique_ptr`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fen-us%2F05-pointers.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A82%2C%22character%22%3A0%7D%5D "book/en-us/05-pointers.md") là một con trỏ thông minh độc quyền, ngăn cấm các con trỏ thông minh khác chia sẻ cùng một đối tượng, do đó giữ cho mã an toàn:

```cpp
std::unique_ptr<int> pointer = std::make_unique<int>(10); // make_unique, từ C++14
std::unique_ptr<int> pointer2 = pointer; // không hợp lệ
```

> `make_unique` không phức tạp. C++11 không cung cấp `std::make_unique`, nhưng có thể tự triển khai:
>
> ```cpp
> template<typename T, typename ...Args>
> std::unique_ptr<T> make_unique( Args&& ...args ) {
>   return std::unique_ptr<T>( new T( std::forward<Args>(args)... ) );
> }
> ```
>
> Về lý do tại sao nó không được cung cấp, Herb Sutter, chủ tịch Ủy ban Tiêu chuẩn C++, đã đề cập trong [blog](https://herbsutter.com/gotw/_102/) của mình rằng đó là vì họ đã quên.

Vì nó là độc quyền, nói cách khác, nó không thể được sao chép. Tuy nhiên, chúng ta có thể sử dụng `std::move` để chuyển nó sang `unique_ptr` khác, ví dụ:

```cpp
#include <iostream>
#include <memory>

struct Foo {
    Foo()      { std::cout << "Foo::Foo" << std::endl;  }
    ~Foo()     { std::cout << "Foo::~Foo" << std::endl; }
    void foo() { std::cout << "Foo::foo" << std::endl;  }
};

void f(const Foo &) {
    std::cout << "f(const Foo&)" << std::endl;
}

int main() {
    std::unique_ptr<Foo> p1(std::make_unique<Foo>());

    // p1 không rỗng, in ra
    if (p1) p1->foo();
    {
        std::unique_ptr<Foo> p2(std::move(p1));

        // p2 không rỗng, in ra
        f(*p2);

        // p2 không rỗng, in ra
        if(p2) p2->foo();

        // p1 rỗng, không in ra
        if(p1) p1->foo();

        p1 = std::move(p2);

        // p2 rỗng, không in ra
        if(p2) p2->foo();
        std::cout << "p2 đã bị hủy" << std::endl;
    }
    // p1 không rỗng, in ra
    if (p1) p1->foo();

    // Đối tượng Foo sẽ bị hủy khi ra khỏi phạm vi
}
```

## 5.4 [`std::weak_ptr`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fen-us%2F05-pointers.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A148%2C%22character%22%3A0%7D%5D "book/en-us/05-pointers.md")

Nếu bạn suy nghĩ kỹ về [`std::shared_ptr`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%2Fbook%2Fen-us%2F05-pointers.md%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A148%2C%22character%22%3A0%7D%5D "book/en-us/05-pointers.md"), bạn sẽ thấy rằng vẫn còn một vấn đề là tài nguyên không thể được giải phóng. Hãy xem ví dụ sau:

```cpp
#include <iostream>
#include <memory>

class A;
class B;

class A {
public:
    std::shared_ptr<B> pointer;
    ~A() {
        std::cout << "A đã bị hủy" << std::endl;
    }
};
class B {
public:
    std::shared_ptr<A> pointer;
    ~B() {
        std::cout << "B đã bị hủy" << std::endl;
    }
};
int main() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();
    a->pointer = b;
    b->pointer = a;

    return 0;
}
```

Kết quả là A và B sẽ không bị hủy. Điều này là do con trỏ bên trong a và b cũng tham chiếu đến `a, b`, làm cho số lượng tham chiếu của `a, b` trở thành 2 khi rời khỏi phạm vi. Khi con trỏ thông minh `a, b` bị hủy, nó chỉ có thể làm giảm số lượng tham chiếu của khu vực này đi một. Điều này khiến số lượng tham chiếu của vùng nhớ được trỏ bởi đối tượng `a, b` không bằng không, nhưng bên ngoài không có cách nào để tìm thấy vùng này, dẫn đến rò rỉ bộ nhớ, như minh họa trong Hình 5.1:

![Hình 5.1](../../assets/figures/pointers1_en.png)

Giải pháp cho vấn đề này là sử dụng con trỏ tham chiếu yếu `std::weak_ptr`, đây là một tham chiếu yếu (so với `std::shared_ptr` là một tham chiếu mạnh). Một tham chiếu yếu không làm tăng số lượng tham chiếu. Khi sử dụng tham chiếu yếu, quá trình giải phóng cuối cùng được thể hiện trong Hình 5.2:

![Hình 5.2](../../assets/figures/pointers2.png)

Trong hình trên, chỉ còn lại B ở bước cuối cùng, và B không có bất kỳ con trỏ thông minh nào tham chiếu đến nó, vì vậy tài nguyên bộ nhớ này cũng sẽ được giải phóng.

`std::weak_ptr` không có các toán tử `*` và `->` được triển khai, do đó nó không thể thao tác trên tài nguyên. `std::weak_ptr` cho phép chúng ta kiểm tra xem một `std::shared_ptr` có tồn tại hay không. Phương thức `expired()` của `std::weak_ptr` trả về `false` khi tài nguyên chưa được giải phóng; ngược lại, nó trả về `true`.
Hơn nữa, nó cũng có thể được sử dụng để lấy `std::shared_ptr`, trỏ đến đối tượng gốc. Phương thức `lock()` trả về một `std::shared_ptr` đến đối tượng gốc khi tài nguyên chưa được giải phóng, hoặc `nullptr` nếu không.

## Kết luận

Công nghệ con trỏ thông minh không phải là mới. Đây là một công nghệ phổ biến trong nhiều ngôn ngữ. C++ hiện đại giới thiệu công nghệ này, giúp loại bỏ việc lạm dụng `new`/`delete` ở một mức độ nhất định. Đây là một công nghệ trưởng thành hơn. Một mô hình lập trình.

[Table of Content](./toc.md) | [Chương trước](./04-containers.md) | [Chương tiếp theo: Biểu thức chính quy](./06-regex.md)

## Đọc thêm

- [Tại sao C++11 có `make_shared` nhưng không có `make_unique`](https://stackoverflow.com/questions/12580432/why-does-c11-have-make-shared-but-not-make-unique)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).