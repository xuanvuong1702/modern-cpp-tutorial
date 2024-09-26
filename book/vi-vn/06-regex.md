---
title: "Chương 06: Biểu thức chính quy"
type: book-vi-vn
order: 6
---

# Chương 06: Biểu thức chính quy

[TOC]

## 6.1 Giới thiệu

Biểu thức chính quy không phải là một phần của ngôn ngữ C++ nên chúng ta sẽ chỉ giới thiệu sơ lược
về chúng trong chương này.

Biểu thức chính quy được sử dụng để mô tả các mẫu khớp chuỗi.
Thông thường, biểu thức chính quy được dùng để thực hiện ba tác vụ chính:

1. Kiểm tra xem một chuỗi có chứa một chuỗi con nào đó hay không;
2. Thay thế các chuỗi con khớp với mẫu;
3. Trích xuất các chuỗi con khớp với mẫu từ một chuỗi.

Biểu thức chính quy là các mẫu văn bản bao gồm các ký tự thông thường
(ví dụ: các chữ cái từ a đến z) và các ký tự đặc biệt.
Mỗi mẫu biểu thức chính quy mô tả một hoặc nhiều chuỗi cần tìm trong văn bản.
Biểu thức chính quy hoạt động như một khuôn mẫu để so khớp các chuỗi ký tự
với chuỗi văn bản đầu vào.

### Ký tự thông thường

Ký tự thông thường bao gồm tất cả các ký tự in được và không in được
mà không được định nghĩa là ký tự meta.
Chúng bao gồm tất cả các chữ cái viết hoa và viết thường, tất cả các chữ số,
tất cả các dấu câu và một số ký hiệu khác.

### Ký tự đặc biệt (metacharacter)

Ký tự đặc biệt là ký tự mang ý nghĩa đặc biệt trong biểu thức chính quy,
chúng là cốt lõi của cú pháp khớp mẫu.
Dưới đây là bảng liệt kê một số ký tự đặc biệt:

| Ký tự | Ý nghĩa                                                             |
| :-----: | :------------------------------------------------------------------- |
|   `$`   | Khớp với vị trí kết thúc của chuỗi đầu vào.                        |
| `(`,`)` | Đánh dấu bắt đầu và kết thúc của một nhóm con.                      |
|   `*`   | Khớp với ký tự hoặc nhóm con đứng trước 0 hoặc nhiều lần.       |
|   `+`   | Khớp với ký tự hoặc nhóm con đứng trước 1 hoặc nhiều lần.       |
|   `.`   | Khớp với bất kỳ ký tự nào ngoại trừ ký tự xuống dòng (`\n`).      |
|   `[`   | Đánh dấu bắt đầu của một lớp ký tự.                               |
|   `?`   | Khớp với ký tự hoặc nhóm con đứng trước 0 hoặc 1 lần.            |
|   `\`   | Ký tự thoát, được sử dụng để thoát các ký tự đặc biệt.            |
|   `^`   | Khớp với vị trí bắt đầu của chuỗi đầu vào.                        |
|   `{`   | Đánh dấu bắt đầu của một bộ định lượng.                          |
|   `\|`   | Toán tử OR, khớp với biểu thức con bên trái hoặc bên phải.      |

### Bộ định lượng (quantifier)

Bộ định lượng được sử dụng để chỉ định số lần lặp lại của một ký tự hoặc
một nhóm con trong biểu thức chính quy.
Dưới đây là bảng liệt kê một số bộ định lượng:

| Ký tự  | Ý nghĩa                                                            |
| :------ | :------------------------------------------------------------------ |
|   `*`   | Khớp với ký tự hoặc nhóm con đứng trước 0 hoặc nhiều lần.       |
|   `+`   | Khớp với ký tự hoặc nhóm con đứng trước 1 hoặc nhiều lần.       |
|   `?`   | Khớp với ký tự hoặc nhóm con đứng trước 0 hoặc 1 lần.            |
|  `{n}`  | Khớp với ký tự hoặc nhóm con đứng trước đúng `n` lần.              |
| `{n,}`  | Khớp với ký tự hoặc nhóm con đứng trước ít nhất `n` lần.           |
| `{n,m}` | Khớp với ký tự hoặc nhóm con đứng trước từ `n` đến `m` lần.         |

Với hai bảng trên, chúng ta có thể hiểu được hầu hết các biểu thức chính quy.

## 6.2 `std::regex` và các thành phần liên quan

Biểu thức chính quy là công cụ phổ biến để so khớp chuỗi.
Tuy nhiên, trong C++ truyền thống, biểu thức chính quy không được hỗ trợ ở cấp độ ngôn ngữ,
và cũng không được bao gồm trong thư viện chuẩn.
C++ là một ngôn ngữ hiệu năng cao. Trong quá trình phát triển các dịch vụ nền,
biểu thức chính quy cũng thường được sử dụng để phân tích các URL.
Đây là một kỹ thuật phổ biến và đã được kiểm chứng trong thực tế.

Giải pháp thường được sử dụng là thư viện biểu thức chính quy của `boost`.
C++11 đã chính thức đưa biểu thức chính quy vào thư viện chuẩn,
cung cấp hỗ trợ chuẩn cho biểu thức chính quy ở cấp độ ngôn ngữ
và không còn phụ thuộc vào thư viện của bên thứ ba nữa.

Thư viện biểu thức chính quy trong C++11 hoạt động trên các đối tượng `std::string`.
Mẫu biểu thức chính quy được biểu diễn bởi lớp `std::regex`
(thực chất là `std::basic_regex`).
Hàm `std::regex_match` được sử dụng để so khớp chuỗi với biểu thức chính quy,
kết quả của quá trình so khớp được lưu trữ trong một đối tượng `std::smatch`
(thực chất là `std::match_results`).

Chúng ta sẽ sử dụng một ví dụ đơn giản để minh họa cách sử dụng thư viện này.
Hãy xem xét biểu thức chính quy sau:

- `[a-z]+\.txt`: Trong biểu thức này, `[a-z]` khớp với một chữ cái thường,
  `+` khớp với ký tự hoặc nhóm con đứng trước 1 hoặc nhiều lần,
  do đó `[a-z]+` khớp với một chuỗi các chữ cái thường.
  Trong biểu thức chính quy, `.` khớp với bất kỳ ký tự nào,
  còn `\.` khớp với ký tự ".".
  Cuối cùng, `txt` khớp với chuỗi "txt".
  Tóm lại, biểu thức này khớp với tên của các tệp văn bản
  có tên chỉ chứa các chữ cái thường.

Hàm `std::regex_match` được sử dụng để so khớp chuỗi với biểu thức chính quy,
nó có nhiều dạng quá tải khác nhau.
Dạng đơn giản nhất nhận vào hai tham số là chuỗi `std::string` và biểu thức
chính quy `std::regex`, trả về `true` nếu chuỗi khớp với biểu thức,
ngược lại trả về `false`. Ví dụ:

```cpp
#include <iostream>
#include <string>
#include <regex>

int main() {
  std::string fnames[] = {"foo.txt", "bar.txt", "test", "a0.txt", "AAA.txt"};
  // Trong C++, `\` là ký tự thoát trong chuỗi.
  // Do đó, để sử dụng ký tự `.` trong biểu thức chính quy,
  // chúng ta cần thoát nó hai lần, thành `\\.`.
  std::regex txt_regex("[a-z]+\\.txt");
  for (const auto &fname : fnames)
    std::cout << fname << ": " << std::regex_match(fname, txt_regex)
              << std::endl;
}
```

Một dạng quá tải khác của `std::regex_match` nhận vào ba tham số:
`std::string`, `std::smatch` và `std::regex`.
`std::smatch` thực chất là một `std::match_results`.
Trong thư viện chuẩn, `std::smatch` được định nghĩa là
`std::match_results<std::string::const_iterator>`,
có nghĩa là `match_results` của một iterator trỏ đến các ký tự trong chuỗi.
Sử dụng `std::smatch`, chúng ta có thể dễ dàng lấy ra các chuỗi con
khớp với biểu thức chính quy. Ví dụ:

```cpp
std::regex base_regex("([a-z]+)\\.txt");
std::smatch base_match;
for (const auto &fname : fnames) {
  if (std::regex_match(fname, base_match, base_regex)) {
    // Phần tử đầu tiên của std::smatch là chuỗi khớp với toàn bộ biểu thức.
    // Phần tử thứ hai là chuỗi khớp với nhóm con đầu tiên
    // trong biểu thức.
    if (base_match.size() == 2) {
      std::string base = base_match[1].str();
      std::cout << "sub-match[0]: " << base_match[0].str() << std::endl;
      std::cout << fname << " sub-match[1]: " << base << std::endl;
    }
  }
}
```

Kết quả đầu ra của hai đoạn mã trên:

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

Chương này đã giới thiệu sơ lược về biểu thức chính quy và cách sử dụng
thư viện biểu thức chính quy trong C++ thông qua một ví dụ thực tế.

## Bài tập

Trong quá trình phát triển máy chủ web, chúng ta thường muốn xử lý các yêu cầu
đến các đường dẫn (URL) khác nhau. Biểu thức chính quy là một trong những công
cụ hữu ích để thực hiện điều này.
Cho cấu trúc request sau:

```cpp
struct Request {
  // Phương thức của request (POST, GET, etc.), đường dẫn, phiên bản HTTP
  std::string method, path, http_version;
  // Con trỏ thông minh đến nội dung của request
  std::shared_ptr<std::istream> content;
  // Header của request
  std::unordered_map<std::string, std::string> header;
  // Kết quả khớp biểu thức chính quy với đường dẫn
  std::smatch path_match;
};
```

Kiểu dữ liệu `resource_type` để lưu trữ các tài nguyên:

```cpp
typedef std::map<
    std::string,
    std::unordered_map<std::string,
                       std::function<void(std::ostream &, Request &)>>>
    resource_type;
```

Và lớp `ServerBase` để xử lý các request:

```cpp
template <typename socket_type> class ServerBase {
public:
  resource_type resource;
  resource_type default_resource;

  void start() {
    // TODO
  }

protected:
  Request parse_request(std::istream &stream) const {
    // TODO
  }
};
```

Hãy triển khai các hàm thành viên `start()` và `parse_request`
sao cho người dùng có thể sử dụng lớp `ServerBase` như sau:

```cpp
template <typename SERVER_TYPE> void start_server(SERVER_TYPE &server) {

  // Xử lý yêu cầu GET đến đường dẫn /match/[chuỗi số],
  // ví dụ: yêu cầu GET đến /match/12345 sẽ trả về 12345
  server.resource["\\/match\\/([0-9]+)"] ["GET"] =
      [](ostream &response, Request &request) {
        string number = request.path_match[1];
        response << "HTTP/1.1 200 OK\r\nContent-Length: " << number.length()
                  << "\r\n\r\n" << number;
      };

  // Xử lý yêu cầu GET mặc định;
  // Hàm ẩn danh này sẽ được gọi nếu không có đường dẫn nào khớp
  // Trả về nội dung của tệp tin trong thư mục www/
  // Mặc định là index.html
  server.default_resource[".*"] ["GET"] =
      [](ostream &response, Request &request) {
        string filename = "www/";

        string path = request.path_match[1];

        // Không cho phép truy cập các tệp tin bên ngoài thư mục www/
        // bằng cách sử dụng `..` trong đường dẫn
        size_t last_pos = path.rfind(".");
        size_t current_pos = 0;
        size_t pos;
        while ((pos = path.find('.', current_pos)) != string::npos &&
               pos != last_pos) {
          current_pos = pos;
          path.erase(pos, 1);
          last_pos--;
        }

        // (...)
      };

  server.start();
}
```

Bạn có thể tham khảo giải pháp gợi ý [tại đây](../../exercises/6).

[Mục lục](./toc.md) | [Chương trước](./05-pointers.md) | [Chương tiếp theo: Luồng và đồng thời](./07-thread.md)

## Đọc thêm

1. [Bình luận từ tác giả của `std::regex`](https://zhihu.com/question/23070203/answer/84248248)
2. [Tài liệu thư viện về Biểu thức chính quy](https://en.cppreference.com/w/cpp/regex)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 Quốc tế</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).


