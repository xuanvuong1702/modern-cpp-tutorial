---
title: "Chương 06 Biểu thức chính quy"
type: book-vi-vn
order: 6
---

# Chương 06 Biểu thức chính quy

[TOC]

## 6.1 Giới thiệu

Biểu thức chính quy không phải là một phần của ngôn ngữ C++ và do đó chúng tôi chỉ giới thiệu ngắn gọn ở đây.

Biểu thức chính quy mô tả một mẫu của việc khớp chuỗi.
Việc sử dụng chung của biểu thức chính quy chủ yếu để đạt được
ba yêu cầu sau:

1. Kiểm tra xem một chuỗi có chứa một dạng chuỗi con nào đó hay không;
2. Thay thế các chuỗi con khớp;
3. Lấy chuỗi con đủ điều kiện từ một chuỗi.

Biểu thức chính quy là các mẫu văn bản bao gồm các ký tự thông thường (như a đến z)
và các ký tự đặc biệt. Một mẫu mô tả một hoặc nhiều chuỗi để khớp khi tìm kiếm văn bản.
Biểu thức chính quy hoạt động như một mẫu để khớp một mẫu ký tự với chuỗi đang được tìm kiếm.

### Ký tự thông thường

Ký tự thông thường bao gồm tất cả các ký tự có thể in và không thể in mà không được chỉ định rõ ràng là ký tự đặc biệt. Điều này bao gồm tất cả các chữ cái viết hoa và viết thường, tất cả các số, tất cả các dấu câu và một số ký hiệu khác.

### Ký tự đặc biệt

Ký tự đặc biệt là ký tự có ý nghĩa đặc biệt trong biểu thức chính quy và cũng là cú pháp khớp lõi của biểu thức chính quy. Xem bảng dưới đây:

| Ký hiệu | Mô tả |
|:----------------:|:---|
| `$` | Khớp với vị trí kết thúc của chuỗi đầu vào.|
| `(`,`)` | Đánh dấu bắt đầu và kết thúc của một biểu thức con. Các biểu thức con có thể được lấy ra để sử dụng sau.|
| `*` | Khớp với biểu thức con trước đó không hoặc nhiều lần. |
| `+` | Khớp với biểu thức con trước đó một hoặc nhiều lần.|
| `.` | Khớp với bất kỳ ký tự đơn nào ngoại trừ ký tự xuống dòng `\n`.|
| `[` | Đánh dấu bắt đầu của một biểu thức ngoặc vuông.|
| `?` | Khớp với biểu thức con trước đó không hoặc một lần, hoặc chỉ định một bộ định lượng không tham lam.|
| `\` | Đánh dấu ký tự tiếp theo là ký tự đặc biệt, ký tự gốc, tham chiếu ngược hoặc ký tự thoát bát phân. Ví dụ, `n` khớp với ký tự `n`. `\n` khớp với ký tự xuống dòng. Chuỗi `\\` khớp với ký tự `'\'`, trong khi `\(` khớp với ký tự `'('`. |
| `^` | Khớp với vị trí bắt đầu của chuỗi đầu vào, trừ khi nó được sử dụng trong biểu thức ngoặc vuông, lúc đó nó chỉ định rằng tập hợp các ký tự không được chấp nhận.|
| `{` | Đánh dấu bắt đầu của một biểu thức định lượng.|
| `\|` | Chỉ định một lựa chọn giữa hai. |

### Bộ định lượng

Bộ định lượng được sử dụng để chỉ định số lần một thành phần nhất định của biểu thức chính quy phải xuất hiện để thỏa mãn khớp. Xem bảng dưới đây:

| Ký hiệu | Mô tả |
|:-------:|:-----|
| `*` | khớp với biểu thức con trước đó không hoặc nhiều lần. Ví dụ, `foo*` khớp với `fo` và `foooo`. `*` tương đương với `{0,}`.|
| `+` | khớp với biểu thức con trước đó một hoặc nhiều lần. Ví dụ, `foo+` khớp với `foo` và `foooo` nhưng không khớp với `fo`. `+` tương đương với `{1,}`.|
| `?` | khớp với biểu thức con trước đó không hoặc một lần. Ví dụ, `Your(s)?` có thể khớp với `Your` trong `Your` hoặc `Yours`. `?` tương đương với `{0,1}`.|
| `{n}` | `n` là một số nguyên không âm. Khớp đúng `n` lần. Ví dụ, `o{2}` không thể khớp với `o` trong `for`, nhưng có thể khớp với hai `o` trong `foo`.|
| `{n,}` | `n` là một số nguyên không âm. Khớp ít nhất `n` lần. Ví dụ, `o{2,}` không thể khớp với `o` trong `for`, nhưng khớp với tất cả `o` trong `foooooo`. `o{1,}` tương đương với `o+`. `o{0,}` tương đương với `o*`.|
| `{n,m}` | `m` và `n` là các số nguyên không âm, trong đó `n` nhỏ hơn hoặc bằng `m`. Khớp ít nhất `n` lần và tối đa `m` lần. Ví dụ, `o{1,3}` sẽ khớp với ba `o` đầu tiên trong `foooooo`. `o{0,1}` tương đương với `o?`. Lưu ý rằng không có khoảng trắng giữa dấu phẩy và hai số. |

Với hai bảng này, chúng ta có thể đọc hầu hết các biểu thức chính quy.

## 6.2 `std::regex` và Liên Quan

Cách phổ biến nhất để khớp nội dung chuỗi là sử dụng biểu thức chính quy. Đáng tiếc là, trong C++ truyền thống, biểu thức chính quy không được hỗ trợ ở mức ngôn ngữ và không được bao gồm trong thư viện chuẩn. C++ là một ngôn ngữ hiệu suất cao. Trong phát triển dịch vụ nền, việc sử dụng biểu thức chính quy cũng được sử dụng khi đánh giá các liên kết tài nguyên URL. Đây là một thực tiễn trưởng thành và phổ biến trong ngành.

Giải pháp chung là sử dụng thư viện biểu thức chính quy của `boost`. C++11 chính thức đưa việc xử lý biểu thức chính quy vào thư viện chuẩn, cung cấp hỗ trợ chuẩn từ mức ngôn ngữ và không còn phụ thuộc vào bên thứ ba.

Thư viện biểu thức chính quy được cung cấp bởi C++11 hoạt động trên đối tượng `std::string`, và mẫu `std::regex` (thực chất là `std::basic_regex`) được khởi tạo và khớp bởi `std::regex_match`, tạo ra `std::smatch` (thực chất là đối tượng `std::match_results`).

Chúng ta sẽ sử dụng một ví dụ đơn giản để giới thiệu ngắn gọn về cách sử dụng thư viện này. Hãy xem xét biểu thức chính quy sau:

- `[a-z]+\.txt`: Trong biểu thức chính quy này, `[a-z]` có nghĩa là khớp với một chữ cái thường, `+` có thể khớp với biểu thức trước đó nhiều lần, vì vậy `[a-z]+` có thể khớp với một chuỗi các chữ cái thường. Trong biểu thức chính quy, [`.`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding/modern-cpp-tutorial") có nghĩa là khớp với bất kỳ ký tự nào, và `\.` có nghĩa là khớp với ký tự [`.`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%2Fmodern-cpp-tutorial%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding/modern-cpp-tutorial") cụ thể, và cuối cùng `txt` có nghĩa là khớp chính xác với ba chữ cái `txt`. Vì vậy, nội dung của biểu thức chính quy này là để khớp với một tệp văn bản bao gồm các chữ cái thường.

`std::regex_match` được sử dụng để khớp chuỗi và biểu thức chính quy, và có nhiều dạng quá tải khác nhau. Dạng đơn giản nhất là truyền `std::string` và một `std::regex` để khớp. Khi khớp thành công, nó sẽ trả về `true`, ngược lại, nó sẽ trả về `false`. Ví dụ:

```cpp
#include <iostream>
#include <string>
#include <regex>

int main() {
    std::string fnames[] = {"foo.txt", "bar.txt", "test", "a0.txt", "AAA.txt"};
    // Trong C++, `\` sẽ được sử dụng như một ký tự thoát trong chuỗi.
    // Để `\.` được truyền như một biểu thức chính quy,
    // cần phải thực hiện thoát lần thứ hai cho `\`, do đó chúng ta có `\\.`
    std::regex txt_regex("[a-z]+\\.txt");
    for (const auto &fname: fnames)
        std::cout << fname << ": " << std::regex_match(fname, txt_regex) << std::endl;
}
```

Một dạng phổ biến khác là truyền vào ba đối số `std::string`/`std::smatch`/`std::regex`.
Bản chất của `std::smatch` thực ra là `std::match_results`.
Trong thư viện chuẩn, `std::smatch` được định nghĩa là `std::match_results<std::string::const_iterator>`,
có nghĩa là `match_results` của một kiểu iterator chuỗi con.
Sử dụng `std::smatch` để dễ dàng lấy kết quả khớp, ví dụ:

```cpp
std::regex base_regex("([a-z]+)\\.txt");
std::smatch base_match;
for(const auto &fname: fnames) {
    if (std::regex_match(fname, base_match, base_regex)) {
        // phần tử đầu tiên của std::smatch khớp với toàn bộ chuỗi
        // phần tử thứ hai của std::smatch khớp với biểu thức đầu tiên
        // có dấu ngoặc
        if (base_match.size() == 2) {
            std::string base = base_match[1].str();
            std::cout << "sub-match[0]: " << base_match[0].str() << std::endl;
            std::cout << fname << " sub-match[1]: " << base << std::endl;
        }
    }
}
```

Kết quả đầu ra của hai đoạn mã trên là:

```
foo.txt: 1
bar.txt: 1
test: 0
a0.txt: 0
AAA.txt: 0
sub-match[0]: foo.txt
foo.txt sub-match[1]: foo
sub-match[0]: bar.txt
bar.txt sub-match[1]: bar
```

## Kết luận

Phần này giới thiệu ngắn gọn về biểu thức chính quy,
sau đó giới thiệu cách sử dụng thư viện biểu thức chính quy
thông qua một ví dụ thực tế dựa trên các yêu cầu chính
khi sử dụng biểu thức chính quy.

## Bài tập

Trong phát triển máy chủ web, chúng ta thường muốn phục vụ một số tuyến đường thỏa mãn một điều kiện nhất định.
Biểu thức chính quy là một trong những công cụ để thực hiện điều này.
Dựa vào cấu trúc yêu cầu sau:

```cpp
struct Request {
    // phương thức request, POST, GET; đường dẫn; phiên bản HTTP
    std::string method, path, http_version;
    // sử dụng smart pointer để đếm tham chiếu của nội dung
    std::shared_ptr<std::istream> content;
    // container băm, từ điển key-value
    std::unordered_map<std::string, std::string> header;
    // sử dụng biểu thức chính quy để khớp đường dẫn
    std::smatch path_match;
};
```

Loại tài nguyên được yêu cầu:

```cpp
typedef std::map<
    std::string, std::unordered_map<
        std::string,std::function<void(std::ostream&, Request&)>>> resource_type;
```

Và mẫu máy chủ:

```cpp
template <typename socket_type>
class ServerBase {
public:
    resource_type resource;
    resource_type default_resource;

    void start() {
        // TODO
    }
protected:
    Request parse_request(std::istream& stream) const {
        // TODO
    }
}
```

Vui lòng triển khai các hàm thành viên `start()` và `parse_request`. Cho phép người dùng mẫu máy chủ chỉ định các tuyến đường như sau:

```cpp
template<typename SERVER_TYPE>
void start_server(SERVER_TYPE &server) {

    // xử lý yêu cầu GET cho /match/[digit+numbers],
    // ví dụ: yêu cầu GET là /match/abc123, sẽ trả về abc123
    server.resource["fill_your_reg_ex"]["GET"] =
        [](ostream& response, Request& request)
    {
        string number=request.path_match[1];
        response << "HTTP/1.1 200 OK\r\nContent-Length: " << number.length()
            << "\r\n\r\n" << number;
    };

    // xử lý yêu cầu GET mặc định;
    // hàm ẩn danh sẽ được gọi
    // nếu không có khớp nào khác, trả về các tệp trong thư mục web/
    // mặc định: index.html
    server.default_resource["fill_your_reg_ex"]["GET"] =
        [](ostream& response, Request& request)
    {
        string filename = "www/";

        string path = request.path_match[1];

        // cấm sử dụng [`..`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FUsers%2Fphungvuong%2FDocuments%2Fcoding%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/Users/phungvuong/Documents/coding") để truy cập nội dung ngoài thư mục web/
        size_t last_pos = path.rfind(".");
        size_t current_pos = 0;
        size_t pos;
        while((pos=path.find('.', current_pos)) != string::npos && pos != last_pos) {
            current_pos = pos;
            path.erase(pos, 1);
            last_pos--;
        }

        // (...)
    };

    server.start();
}
```

Một giải pháp gợi ý có thể được tìm thấy [tại đây](../../exercises/6).

[Table of Content](./toc.md) | [Chương trước](./05-pointers.md) | [Chương tiếp theo: Threads and Concurrency](./07-thread.md)

## Đọc thêm

1. [Bình luận từ tác giả của `std::regex`](https://zhihu.com/question/23070203/answer/84248248)
2. [Tài liệu thư viện về Biểu thức chính quy](https://en.cppreference.com/w/cpp/regex)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 Quốc tế</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).