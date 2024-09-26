# C++ 11/14/17/20 Ngay lập tức

## Mục lục

- [**Lời mở đầu**](./00-preface.md)
- [**Chương 01 Hướng tới C++ hiện đại**](./01-intro.md)
    + 1.1 Các tính năng không còn được khuyến khích sử dụng
    + 1.2 Tương thích với C
    + Đọc thêm
- [**Chương 02 Cải tiến khả năng sử dụng ngôn ngữ**](./02-usability.md)
    + 2.1 Hằng số
        - nullptr
        - constexpr
    + 2.2 Biến và khởi tạo
        - if-switch
        - Danh sách khởi tạo
        - Ràng buộc có cấu trúc
    + 2.3 Suy luận kiểu
        - auto
        - decltype
        - Suy luận kiểu trả về
        - decltype(auto)
    + 2.4 Luồng điều khiển
        - if constexpr
        - Vòng lặp dựa trên phạm vi
    + 2.5 Template
        - Extern template
        - Dấu ">"
        - Template bí danh kiểu
        - Tham số template mặc định
        - Template tham số biến đổi
        - Biểu thức gấp
        - Suy luận tham số template không kiểu
    + 2.6 Hướng đối tượng
        - Hàm tạo ủy nhiệm
        - Hàm tạo kế thừa
        - Ghi đè hàm ảo rõ ràng
        - override
        - final
        - Xóa hàm mặc định rõ ràng
        - Kiểu liệt kê mạnh
- [**Chương 03 Cải tiến thời gian chạy của ngôn ngữ**](./03-runtime.md)
    + 3.1 Biểu thức Lambda
        + Cơ bản
        + Lambda tổng quát
    + 3.2 Bao bọc đối tượng hàm
        + std::function
        + std::bind và std::placeholder
    + 3.3 Tham chiếu rvalue
        + lvalue, rvalue, prvalue, xvalue
        + Tham chiếu rvalue và tham chiếu lvalue
        + Ngữ nghĩa di chuyển
        + Chuyển tiếp hoàn hảo
- [**Chương 04: Container**](./04-containers.md)
    + 4.1 Container tuyến tính
        + `std::array`
        + `std::forward_list`
    + 4.2 Container không có thứ tự
        + `std::unordered_set`
        + `std::unordered_map`
    + 4.3 Tuple (`std::tuple`)
        + Thao tác cơ bản
        + Truy cập phần tử bằng chỉ mục thời gian chạy (`std::variant`)
        + Nối và duyệt tuple
- [**Chương 05: Con trỏ thông minh và Quản lý bộ nhớ**](./05-pointers.md)
    + 5.1 RAII và đếm tham chiếu
    + 5.2 `std::shared_ptr`
    + 5.3 `std::unique_ptr`
    + 5.4 `std::weak_ptr`
- [**Chương 06: Biểu thức chính quy**](./06-regex.md)
    + 6.1 Giới thiệu
        + Ký tự thông thường
        + Ký tự đặc biệt (metacharacter)
        + Bộ định lượng (quantifier)
    + 6.2 `std::regex` và các thành phần liên quan
        + `std::regex`
        + `std::regex_match`
        + `std::match_results`
- [**Chương 07: Lập trình song song và đồng thời**](./07-thread.md)
    + 7.1 Cơ bản về lập trình song song
    + 7.2 Mutex và đoạn mã tới hạn
    + 7.3 Future
    + 7.4 Biến điều kiện
    + 7.5 Thao tác nguyên tử và mô hình bộ nhớ
        + Thao tác nguyên tử
        + Mô hình nhất quán
        + Thứ tự bộ nhớ (memory order)
- [**Chương 08: Hệ thống tệp**](./08-filesystem.md)
    + 8.1 Tài liệu và liên kết
    + 8.2 `std::filesystem`
- [**Chương 09: Các tính năng nhỏ**](./09-others.md)
    + 9.1 Kiểu dữ liệu mới
        + `long long int`
    + 9.2 `noexcept` và các toán tử liên quan
    + 9.3 Literal
        + Raw string literal
        + User-defined literal
    + 9.4 Căn chỉnh bộ nhớ
- [**Chương 10: C++20**](./10-cpp20.md)
    + 10.1 Concept
    + 10.2 Range
    + 10.3 Module
    + 10.4 Coroutine
    + 10.5 Transaction Memory
- [**Phụ lục 1: Tài liệu học tập thêm**](./appendix1.md)
- [**Phụ lục 2: Thực hành tốt nhất với C++ hiện đại**](./appendix2.md)

Mục lục | Chương trước | [Chương tiếp theo: Lời mở đầu](./00-preface.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Công việc này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).
