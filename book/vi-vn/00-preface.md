---
title: Preface
type: book-vi-vn
order: 0
---
# Lời mở đầu

[TOC]

## Giới thiệu

Ngôn ngữ lập trình C++ sở hữu một nhóm người dùng khá lớn. Từ khi ra đời của C++98 cho đến khi hoàn thiện chính thức của C++11, nó đã không ngừng giữ được sự liên quan. C++14/17 là bổ sung và tối ưu hóa quan trọng cho C++11, và C++20 đưa ngôn ngữ này đến cửa của sự hiện đại hóa. Các tính năng mở rộng của tất cả các tiêu chuẩn mới này được tích hợp vào ngôn ngữ C++ và thổi vào đó sức sống mới.
Các lập trình viên C++ vẫn đang sử dụng **C++ truyền thống** (cuốn sách này chỉ đến C++98 và các tiêu chuẩn trước đó là C++ truyền thống) có thể thậm chí ngạc nhiên bởi việc họ không sử dụng cùng một ngôn ngữ khi đọc mã C++ hiện đại.

**C++ hiện đại** (cuốn sách này chỉ đến C++11/14/17/20) giới thiệu nhiều tính năng vào C++ truyền thống, đưa toàn bộ ngôn ngữ lên một cấp độ hiện đại hóa mới. C++ hiện đại không chỉ cải thiện khả năng sử dụng của chính ngôn ngữ C++, nhưng sự chỉnh sửa ngữ nghĩa của từ khóa `auto` mang lại cho chúng ta nhiều sự tự tin hơn trong việc thao tác với các kiểu mẫu cực kỳ phức tạp. Đồng thời, đã có rất nhiều cải tiến được thực hiện đối với thời gian chạy của ngôn ngữ. Sự xuất hiện của biểu thức Lambda đã mang đến cho C++ tính năng "đóng" của "hàm ẩn danh", mà gần như tất cả các ngôn ngữ lập trình hiện đại (như Python, Swift, v.v.) đều có. Điều này đã trở nên phổ biến, và sự xuất hiện của tham chiếu rvalue đã giải quyết vấn đề về hiệu suất của đối tượng tạm thời mà C++ đã bị chỉ trích từ lâu.

C++17 là hướng mà cộng đồng C++ đã thúc đẩy trong ba năm qua. Nó cũng chỉ ra một hướng phát triển quan trọng của lập trình **C++ hiện đại**. Mặc dù nó không xuất hiện nhiều như C++11, nhưng nó chứa đựng một số lượng lớn các ngôn ngữ nhỏ và đẹp cùng các tính năng (như ràng buộc cấu trúc), và sự xuất hiện của những tính năng này một lần nữa đã sửa đổi mô hình lập trình của chúng tôi trong C++.

C++ hiện đại cũng thêm rất nhiều công cụ và phương pháp vào thư viện chuẩn của nó như `std::thread` ở cấp độ của ngôn ngữ, hỗ trợ lập trình đồng thời và không còn phụ thuộc vào hệ thống cơ sở trên các nền tảng khác nhau. API thực hiện hỗ trợ đa nền tảng ở cấp độ ngôn ngữ; `std::regex` cung cấp hỗ trợ biểu thức chính quy đầy đủ và hơn thế nữa. C++98 đã được chứng minh là một "mô hình" rất thành công, và sự xuất hiện của C++ hiện đại thúc đẩy thêm mô hình này, làm cho C++ trở thành một ngôn ngữ tốt hơn cho lập trình hệ thống và phát triển thư viện. Khái niệm xác minh thời gian biên dịch của các tham số mẫu, tăng cường thêm khả năng sử dụng của ngôn ngữ.

Kết luận, như một người ủng hộ và thực hành C++, chúng tôi luôn duy trì một tư duy mở để chấp nhận những điều mới, và chúng tôi có thể thúc đẩy sự phát triển của C++ nhanh hơn, làm cho ngôn ngữ cũ và mới mẻ này trở nên sống động hơn.

## Mục tiêu

- Cuốn sách này giả định rằng người đọc đã quen thuộc với C++ truyền thống (tức là C++98 hoặc sớm hơn), ít nhất họ không gặp bất kỳ khó khăn nào trong việc đọc mã C++ truyền thống. Nói cách khác, những người có kinh nghiệm lâu dài trong C++ truyền thống và những người mong muốn nhanh chóng hiểu các tính năng của C++ hiện đại trong một khoảng thời gian ngắn rất phù hợp để đọc cuốn sách này;

- Cuốn sách này giới thiệu đến một mức độ nhất định về ma thuật đen của C++ hiện đại. Tuy nhiên, những ma thuật này rất hạn chế, chúng không phù hợp cho người đọc muốn học C++ nâng cao. Mục đích của cuốn sách này là cung cấp một khởi đầu nhanh chóng cho C++ hiện đại. Tất nhiên, người đọc nâng cao cũng có thể sử dụng cuốn sách này để ôn tập và tự kiểm tra về C++ hiện đại.

## Mục đích

Cuốn sách này mang tiêu đề "Ngay lập tức". Nó nhằm mục đích cung cấp một giới thiệu toàn diện về các tính năng liên quan đến C++ hiện đại (trước thập kỷ 2020).
Người đọc có thể chọn nội dung hấp dẫn theo mục lục sau để học và nhanh chóng làm quen với các tính năng mới có sẵn.
Người đọc nên nhận biết rằng tất cả những tính năng này không phải là bắt buộc. Nó nên được học khi bạn cần nó.

Đồng thời, thay vì chỉ là ngữ pháp, cuốn sách giới thiệu lịch sử nền tảng kỹ thuật một cách đơn giản nhất có thể, điều này cung cấp sự giúp đỡ lớn trong việc hiểu tại sao những tính năng này xuất hiện.

Ngoài ra, tác giả muốn khuyến khích rằng người đọc nên có thể sử dụng trực tiếp C++ hiện đại trong các dự án mới của họ và chuyển dần các dự án cũ của họ sang C++ hiện đại sau khi đọc cuốn sách.
## Mã

Mỗi chương của cuốn sách này có rất nhiều mã. Nếu bạn gặp vấn đề khi viết mã của riêng mình với các tính năng giới thiệu của sách, bạn có thể đọc mã nguồn đi kèm với sách. Bạn có thể tìm sách [tại đây](../../code). Tất cả mã được tổ chức theo chương, tên thư mục là số chương.

## Bài tập

Có ít bài tập ở cuối mỗi chương của sách. Nó dùng để kiểm tra xem bạn có thể sử dụng các điểm kiến thức trong chương hiện tại hay không. Bạn có thể tìm câu trả lời có thể cho vấn đề từ [tại đây](../../exercise). Tên thư mục là số chương.

[Mục lục](./toc.md) | [Chương tiếp theo: Hướng tới C++ hiện đại](./01-intro.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Công trình này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã của kho lưu trữ này được mở nguồn theo [giấy phép MIT](../../LICENSE).
