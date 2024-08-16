# C++ 11/14/17/20 Ngay lập tức

## Mục lục

- [**Lời mở đầu**](./00-preface.md)
- [**Chương 01 Hướng tới C++ hiện đại**](./01-intro.md)
    + 1.1 Các tính năng không còn được sử dụng
    + 1.2 Tương thích với C
    + Đọc thêm
- [**Chương 02 Cải tiến Khả năng Sử dụng Ngôn ngữ**](./02-usability.md)
    + 2.1 Hằng số
        - nullptr
        - constexpr
    + 2.2 Biến & Khởi tạo
        - Câu lệnh Điều kiện
        - Danh sách Khởi tạo
        - Ràng buộc cấu trúc
    + 2.3 Suy luận Kiểu
        - auto
        - decltype
        - Kiểu trả về cuối
        - decltype(auto)
    + 2.4 Kiểm soát Luồng
        - if constexpr
        - Vòng lặp dựa trên phạm vi
    + 2.5 Mẫu
        - Mẫu bên ngoài
        - Dấu ">"
        - Bí danh mẫu kiểu
        - Tham số mẫu mặc định
        - Mẫu biến quy
        - Biểu thức gấp
        - Suy luận tham số mẫu không kiểu
    + 2.6 Lập trình hướng đối tượng
        - Hàm tạo ủy quyền
        - Hàm tạo kế thừa
        - Ghi đè rõ ràng hàm ảo
        - override
        - final
        - Xóa rõ ràng hàm mặc định
        - Liệt kê kiểu mạnh
- [**Chương 03 Cải tiến Thời gian Chạy Ngôn ngữ**](./03-runtime.md)
    + 3.1 Biểu thức Lambda
        + Cơ bản
        + Tổng quát
    + 3.2 Bộ đóng gói đối tượng hàm
        + std::function
        + std::bind/std::placeholder
    + 3.3 Tham chiếu rvalue
        + lvalue, rvalue, prvalue, xvalue
        + Tham chiếu rvalue và tham chiếu lvalue
        + Ngữ cảnh di chuyển
        + Chuyển tiếp hoàn hảo
- [**Chương 04 Các Container**](./04-containers.md)
    + 4.1 Các container tuyến tính
        + `std::array`
        + `std::forward_list`
    + 4.2 Các container không có thứ tự
        + `std::unordered_set`
        + `std::unordered_map`
    + 4.3 Bộ `std::tuple`
        + Thao tác cơ bản
        + Chỉ mục thời gian chạy `std::variant`
        + Hợp nhất và lặp
- [**Chapter 05 Smart Pointers and Memory Management**](./05-pointers.md)
    + 5.1 RAII và đếm tham chiếu
    + 5.2 `std::shared_ptr`
    + 5.3 `std::unique_ptr`
    + 5.4 `std::weak_ptr`
- [**Chương 06 Biểu thức Chính quy**](./06-regex.md)
    + 6.1 Giới thiệu
        + Ký tự thông thường
        + Ký tự đặc biệt
        + Số lượng
    + 6.2 `std::regex` và liên quan
        + `std::regex`
        + `std::regex_match`
        + `std::match_results`
- [**Chương 07 Song song và Đồng thời**](./07-thread.md)
    + 7.1 Cơ bản về Song song
    + 7.2 Mutex và Phần tử quan trọng
    + 7.3 Tương lai
    + 7.4 Biến điều kiện
    + 7.5 Thao tác Nguyên tử và Mô hình Bộ nhớ
        + Thao tác Nguyên tử
        + Mô hình Nhất quán
        + Trật tự Bộ nhớ
- [**Chương 08 Hệ thống Tệp**](./08-filesystem.md)
    + 8.1 Tài liệu và liên kết
    + 8.2 `std::filesystem`
- [**Chương 09 Các tính năng Nhỏ**](./09-others.md)
    + 9.1 Các kiểu mới
        + `long long int`
    + 9.2 `noexcept` và các thao tác của nó
    + 9.3 Literal
        + Raw String Literal
        + Custom String Literal
    + 9.4 Căn chỉnh Bộ nhớ
- [**Chương 10 Triển vọng: Giới thiệu về C++20**](./10-cpp20.md)
    + 10.1 Khái niệm
    + 10.2 Phạm vi
    + 10.3 Module
    + 10.4 Coroutine
    + 10.5 Bộ nhớ Giao dịch
- [**Phụ lục 1: Tài liệu Học thêm**](./appendix1.md)
- [**Phụ lục 2: Thực hành tốt nhất C++ hiện đại**](./appendix2.md)

Mục lục | Chương cuối | [Chương tiếp theo: Lời mở đầu](./00-preface.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Công việc này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã nguồn của kho lưu trữ này được mở dưới [Giấy phép MIT](../../LICENSE).