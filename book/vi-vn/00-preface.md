---
title: Lời Nói Đầu
type: book-vi-vn
order: 0
---
# Lời Nói Đầu

[TOC]

## Giới thiệu

Ngôn ngữ lập trình C++ sở hữu một cộng đồng người dùng khá lớn. Từ khi ra đời của C++98 cho đến khi C++11 được hoàn thiện chính thức, nó vẫn luôn giữ được sự phù hợp. C++14/17 là một bản bổ sung và tối ưu hóa quan trọng cho C++11, và C++20 đưa ngôn ngữ này đến gần hơn với sự hiện đại hóa. Các tính năng mở rộng của tất cả các tiêu chuẩn mới này được tích hợp vào ngôn ngữ C++ và thổi vào đó sức sống mới.
Các lập trình viên C++ vẫn đang sử dụng **C++ truyền thống** (trong cuốn sách này, C++98 và các tiêu chuẩn trước đó được gọi là C++ truyền thống) thậm chí có thể ngạc nhiên khi nhận ra rằng họ không sử dụng cùng một ngôn ngữ khi đọc mã C++ hiện đại.

**C++ hiện đại** (trong cuốn sách này, C++11/14/17/20 được gọi là C++ hiện đại) giới thiệu nhiều tính năng vào C++ truyền thống, đưa toàn bộ ngôn ngữ lên một tầm cao mới của hiện đại hóa. C++ hiện đại không chỉ nâng cao khả năng sử dụng của chính ngôn ngữ C++ mà việc sửa đổi ngữ nghĩa của từ khóa `auto` còn mang đến cho chúng ta sự tự tin hơn trong việc thao tác với các kiểu template cực kỳ phức tạp. Đồng thời, rất nhiều cải tiến đã được áp dụng cho thời gian chạy của ngôn ngữ. Sự xuất hiện của biểu thức Lambda đã mang đến cho C++ tính năng "bao đóng" của "hàm ẩn danh", vốn đã phổ biến trong hầu hết các ngôn ngữ lập trình hiện đại (như Python, Swift, v.v.). Sự xuất hiện của tham chiếu rvalue đã giải quyết vấn đề về hiệu quả của đối tượng tạm thời mà C++ đã bị chỉ trích từ lâu.

C++17 là hướng đi được cộng đồng C++ thúc đẩy trong ba năm qua. Nó cũng chỉ ra một hướng phát triển quan trọng của lập trình **C++ hiện đại**. Mặc dù nó không xuất hiện nhiều như C++11, nhưng nó chứa một số lượng lớn các tính năng ngôn ngữ nhỏ gọn và hữu ích (như structured binding), và sự xuất hiện của những tính năng này một lần nữa điều chỉnh mô hình lập trình của chúng ta trong C++.

C++ hiện đại cũng bổ sung rất nhiều công cụ và phương thức vào thư viện chuẩn của nó, chẳng hạn như `std::thread` ở cấp độ ngôn ngữ, hỗ trợ lập trình đồng thời và không còn phụ thuộc vào hệ thống cơ sở trên các nền tảng khác nhau. API thực hiện hỗ trợ đa nền tảng ở cấp độ ngôn ngữ; `std::regex` cung cấp hỗ trợ đầy đủ cho biểu thức chính quy và hơn thế nữa. C++98 đã được chứng minh là một "mô hình" rất thành công, và sự xuất hiện của C++ hiện đại tiếp tục thúc đẩy mô hình này, khiến C++ trở thành một ngôn ngữ tốt hơn cho lập trình hệ thống và phát triển thư viện. Concepts xác minh các tham số template trong thời gian biên dịch, nâng cao hơn nữa khả năng sử dụng của ngôn ngữ.

Tóm lại, với tư cách là người ủng hộ và thực hành C++, chúng ta nên luôn duy trì một tư duy cởi mở để chấp nhận những điều mới, và chúng ta có thể thúc đẩy sự phát triển của C++ nhanh hơn, làm cho ngôn ngữ vừa cũ vừa mới này trở nên sống động hơn.

## Mục tiêu

- Cuốn sách này giả định rằng người đọc đã quen thuộc với C++ truyền thống (tức là C++98 hoặc phiên bản cũ hơn), ít nhất họ không gặp bất kỳ khó khăn nào trong việc đọc mã C++ truyền thống. Nói cách khác, những người có kinh nghiệm lâu năm với C++ truyền thống và những người mong muốn nhanh chóng hiểu các tính năng của C++ hiện đại trong một khoảng thời gian ngắn sẽ rất phù hợp để đọc cuốn sách này;

- Cuốn sách này giới thiệu ở một mức độ nhất định về những kỹ thuật cao cấp của C++ hiện đại. Tuy nhiên, những kỹ thuật này còn khá hạn chế, chúng không phù hợp cho những độc giả muốn tìm hiểu C++ nâng cao. Mục đích của cuốn sách này là cung cấp một điểm khởi đầu nhanh chóng cho C++ hiện đại. Tất nhiên, độc giả nâng cao cũng có thể sử dụng cuốn sách này để ôn tập và tự kiểm tra kiến thức về C++ hiện đại.

## Mục đích

Cuốn sách này có tên "Ngay lập tức". Nó nhằm mục đích cung cấp một cái nhìn tổng quan toàn diện về các tính năng liên quan đến C++ hiện đại (trước những năm 2020).
Độc giả có thể lựa chọn nội dung mình quan tâm theo mục lục dưới đây để tìm hiểu và nhanh chóng làm quen với các tính năng mới hiện có.
Lưu ý rằng không phải tất cả các tính năng này đều bắt buộc. Bạn nên học chúng khi cần đến.

Đồng thời, thay vì chỉ tập trung vào ngữ pháp, cuốn sách này sẽ giới thiệu một cách đơn giản nhất có thể về bối cảnh lịch sử và các yêu cầu kỹ thuật, giúp bạn hiểu rõ hơn lý do tại sao những tính năng này ra đời.

Ngoài ra, tác giả muốn khuyến khích độc giả nên sử dụng C++ hiện đại trực tiếp trong các dự án mới của mình và dần dần chuyển đổi các dự án cũ sang C++ hiện đại sau khi đọc xong cuốn sách.

## Mã nguồn

Mỗi chương của cuốn sách này đều có rất nhiều mã nguồn. Nếu bạn gặp sự cố khi viết mã của riêng mình bằng các tính năng được giới thiệu trong sách, bạn có thể tham khảo mã nguồn đính kèm với sách. Bạn có thể tìm thấy mã nguồn [tại đây](../../code). Tất cả mã nguồn được sắp xếp theo chương, tên thư mục là số chương.

## Bài tập

Có một số bài tập ở cuối mỗi chương của cuốn sách. Mục đích của chúng là để kiểm tra xem bạn có thể áp dụng các điểm kiến thức trong chương hiện tại hay không. Bạn có thể tìm thấy các đáp án gợi ý cho các bài tập [tại đây](../../exercise). Tên thư mục là số chương.

[Mục lục](./toc.md) | [Chương tiếp theo: Hướng tới C++ hiện đại](./01-intro.md)

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Quốc tế Creative Commons Attribution-NonCommercial-NoDerivatives 4.0</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).