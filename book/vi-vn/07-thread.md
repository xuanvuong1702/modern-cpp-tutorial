---
title: "Chương 07 Song song và Đồng thời"
type: book-vi-vn
order: 7
---

# Chương 07 Song song và Đồng thời

[TOC]

## 7.1 Cơ bản về Song song

`std::thread` được sử dụng để tạo một instance luồng thực thi, vì vậy nó là cơ sở cho tất cả lập trình đồng thời. Khi sử dụng, cần bao gồm tệp tiêu đề `<thread>`.
Nó cung cấp một số thao tác luồng cơ bản, chẳng hạn như `get_id()` để lấy ID của luồng đang được tạo, sử dụng `join()` để kết hợp một luồng, v.v., ví dụ:

```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([](){
        std::cout << "hello world." << std::endl;
    });
    t.join();
    return 0;
}
```

## 7.2 Mutex và Vùng phê bình

Chúng ta đã học các kiến thức cơ bản về công nghệ đồng thời trong hệ điều hành hoặc cơ sở dữ liệu, và `mutex` là một trong những lõi.
C++11 giới thiệu một lớp liên quan đến `mutex`, với tất cả các chức năng liên quan trong tệp tiêu đề `<mutex>`.

`std::mutex` là lớp mutex cơ bản nhất trong C++11, và một mutex có thể được tạo bằng cách xây dựng một đối tượng `std::mutex`.
Nó có thể được khóa bằng hàm thành viên `lock()`, và `unlock()` có thể mở khóa.
Nhưng trong quá trình viết mã thực tế, tốt nhất không nên gọi trực tiếp hàm thành viên,
Bởi vì khi gọi hàm thành viên, bạn cần gọi `unlock()` tại lối ra của mỗi vùng phê bình, và tất nhiên, ngoại lệ.
Lúc này, C++11 cũng cung cấp một lớp mẫu `std::lock_guard` cho cơ chế RAII cho mutex.

RAII đảm bảo tính an toàn ngoại lệ của mã trong khi giữ cho mã đơn giản.

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
    static std::mutex mtx;
    std::lock_guard<std::mutex> lock(mtx);

    // thực hiện công việc tranh chấp
    v = change_v;

    // mtx sẽ được giải phóng sau khi ra khỏi phạm vi
}

int main() {
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();

    std::cout << v << std::endl;
    return 0;
}
```

Vì C++ đảm bảo rằng tất cả các đối tượng stack sẽ bị hủy vào cuối thời gian khai báo, nên mã như vậy cũng cực kỳ an toàn.
Dù `critical_section()` trả về bình thường hay nếu một ngoại lệ được ném ra giữa chừng, một quá trình giải phóng stack sẽ được thực hiện, và `unlock()` sẽ được gọi tự động.

> Một ngoại lệ được ném ra và không được bắt (việc có thực hiện giải phóng stack hay không là do triển khai định nghĩa).

`std::unique_lock` linh hoạt hơn `std::lock_guard`.
Các đối tượng của `std::unique_lock` quản lý các thao tác khóa và mở khóa trên đối tượng `mutex` với quyền sở hữu độc quyền (không có đối tượng `unique_lock` nào khác sở hữu quyền sở hữu của một đối tượng `mutex`). Vì vậy, trong lập trình đồng thời, nên sử dụng `std::unique_lock`.

`std::lock_guard` không thể gọi `lock` và `unlock` một cách rõ ràng, còn `std::unique_lock` có thể được gọi ở bất kỳ đâu sau khi khai báo.
Nó có thể giảm phạm vi của khóa và cung cấp mức độ đồng thời cao hơn.

Nếu bạn sử dụng biến điều kiện `std::condition_variable::wait`, bạn phải sử dụng `std::unique_lock` làm tham số.

Ví dụ:

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
    static std::mutex mtx;
    std::unique_lock<std::mutex> lock(mtx);
    // thực hiện các thao tác tranh chấp
    v = change_v;
    std::cout << v << std::endl;
    // giải phóng khóa
    lock.unlock();

    // trong khoảng thời gian này,
    // các luồng khác được phép truy cập v

    // bắt đầu một nhóm thao tác tranh chấp khác
    // khóa lại
    lock.lock();
    v += 1;
    std::cout << v << std::endl;
}

int main() {
    std::thread t1(critical_section, 2), t2(critical_section, 3);
    t1.join();
    t2.join();
    return 0;
}
```

## 7.3 Future

Future được đại diện bởi `std::future`, cung cấp một cách để truy cập kết quả của các hoạt động bất đồng bộ. Câu này rất khó hiểu.
Để hiểu tính năng này, chúng ta cần hiểu hành vi đa luồng trước C++11.

Hãy tưởng tượng nếu luồng chính của chúng ta A muốn mở một luồng mới B để thực hiện một số nhiệm vụ mà chúng ta mong đợi và trả lại cho chúng ta một kết quả.
Lúc này, luồng A có thể đang bận rộn với những việc khác và không có thời gian để quan tâm đến kết quả của B.
Vì vậy, chúng ta tự nhiên hy vọng có thể nhận được kết quả của luồng B vào một thời điểm nào đó.

Trước khi giới thiệu `std::future` trong C++11, cách làm thông thường là:
Tạo một luồng A, bắt đầu nhiệm vụ B trong luồng A, gửi một sự kiện khi nó sẵn sàng và lưu kết quả vào một biến toàn cục.
Luồng chính A đang làm những việc khác. Khi cần kết quả, một luồng được gọi để chờ hàm lấy kết quả của việc thực thi.

`std::future` được cung cấp bởi C++11 đơn giản hóa quá trình này và có thể được sử dụng để lấy kết quả của các nhiệm vụ bất đồng bộ.
Tự nhiên, chúng ta có thể dễ dàng tưởng tượng nó như một phương tiện đơn giản của đồng bộ hóa luồng, tức là rào chắn.

Để xem một ví dụ, chúng ta sử dụng thêm `std::packaged_task`, có thể được sử dụng để bao bọc bất kỳ mục tiêu nào có thể được gọi cho các cuộc gọi bất đồng bộ. Ví dụ:

```cpp
#include <iostream>
#include <thread>
#include <future>

int main() {
    // đóng gói một biểu thức lambda trả về 7 vào std::packaged_task
    std::packaged_task<int()> task([](){return 7;});
    // lấy đối tượng future của task
    std::future<int> result = task.get_future();    // chạy task trong một luồng
    std::thread(std::move(task)).detach();
    std::cout << "đang chờ...";
    result.wait(); // chặn cho đến khi future có kết quả
    // xuất kết quả
    std::cout << "xong!" << std::endl << "kết quả future là " 
              << result.get() << std::endl;
    return 0;
}
```

Sau khi đóng gói mục tiêu cần gọi, bạn có thể sử dụng `get_future()` để lấy một đối tượng `std::future` nhằm thực hiện đồng bộ hóa luồng sau này.

## 7.4 Biến Điều Kiện

Biến điều kiện `std::condition_variable` được tạo ra để giải quyết vấn đề deadlock và được giới thiệu khi các thao tác với mutex không đủ. Ví dụ, một luồng có thể cần phải chờ một điều kiện nào đó trở thành đúng để tiếp tục thực thi. Một vòng lặp chờ đợi vô tận có thể khiến tất cả các luồng khác không thể vào được vùng quan trọng, dẫn đến deadlock khi điều kiện trở thành đúng. Do đó, đối tượng `condition_variable` được tạo ra chủ yếu để đánh thức các luồng đang chờ và tránh deadlock. `notify_one()` của `std::condition_variable` được sử dụng để đánh thức một luồng; `notify_all()` là để thông báo cho tất cả các luồng. Dưới đây là một ví dụ về mô hình nhà sản xuất và người tiêu dùng:

```cpp
#include <queue>
#include <chrono>
#include <mutex>
#include <thread>
#include <iostream>
#include <condition_variable>

int main() {
    std::queue<int> produced_nums;
    std::mutex mtx;
    std::condition_variable cv;
    bool notified = false;  // dấu hiệu thông báo

    auto producer = [&]() {
        for (int i = 0; ; i++) {
            std::this_thread::sleep_for(std::chrono::milliseconds(500));
            std::unique_lock<std::mutex> lock(mtx);
            std::cout << "đang sản xuất " << i << std::endl;
            produced_nums.push(i);
            notified = true;
            cv.notify_all();
        }
    };
    auto consumer = [&]() {
        while (true) {
            std::unique_lock<std::mutex> lock(mtx);
            while (!notified) {  // tránh đánh thức giả
                cv.wait(lock);
            }

            // tạm thời mở khóa để cho phép nhà sản xuất sản xuất thêm
            // thay vì để người tiêu dùng giữ khóa cho đến khi tiêu thụ xong.
            lock.unlock();
            // người tiêu dùng chậm hơn
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));
            lock.lock();
            if (!produced_nums.empty()) {
                std::cout << "đang tiêu thụ " << produced_nums.front() << std::endl;
                produced_nums.pop();
            }
            notified = false;
        }
    };

    std::thread p(producer);
    std::thread cs[2];
    for (int i = 0; i < 2; ++i) {
        cs[i] = std::thread(consumer);
    }
    p.join();
    for (int i = 0; i < 2; ++i) {
        cs[i].join();
    }
    return 0;
}
```

Đáng chú ý là mặc dù chúng ta có thể sử dụng `notify_one()` trong nhà sản xuất, nhưng không nên sử dụng nó ở đây. Bởi vì trong trường hợp có nhiều người tiêu dùng, việc triển khai người tiêu dùng của chúng ta đơn giản là từ bỏ việc giữ khóa, điều này khiến các người tiêu dùng khác có thể cạnh tranh để giành khóa này, nhằm tận dụng tốt hơn sự đồng thời giữa nhiều người tiêu dùng. Nói như vậy, nhưng thực tế vì tính độc quyền của `std::mutex`, chúng ta không thể mong đợi nhiều người tiêu dùng có thể tiêu thụ nội dung trong hàng đợi tiêu dùng song song, và chúng ta vẫn cần một phương pháp chi tiết hơn.

## 7.5 Hoạt Động Nguyên Tử và Mô Hình Bộ Nhớ

Những độc giả cẩn thận có thể nhận thấy rằng ví dụ về mô hình nhà sản xuất-người tiêu dùng trong phần trước có thể gặp phải các tối ưu hóa của trình biên dịch gây ra lỗi chương trình.
Ví dụ, trình biên dịch có thể tối ưu hóa biến `notified`, chẳng hạn như giá trị của một thanh ghi.
Kết quả là, luồng người tiêu dùng có thể không bao giờ quan sát được sự thay đổi của giá trị này. Đây là một câu hỏi hay. Để giải thích vấn đề này, chúng ta cần thảo luận thêm về khái niệm mô hình bộ nhớ được giới thiệu từ C++11. Hãy cùng xem một câu hỏi. Đầu ra của đoạn mã sau là gì?

```cpp
#include <thread>
#include <iostream>

int main() {
    int a = 0;
    volatile int flag = 0;

    std::thread t1([&]() {
        while (flag != 1);

        int b = a;
        std::cout << "b = " << b << std::endl;
    });

    std::thread t2([&]() {
        a = 5;
        flag = 1;
    });

    t1.join();
    t2.join();
    return 0;
}
```

Trực quan mà nói, có vẻ như `a = 5;` trong `t2` luôn thực thi trước `flag = 1;` và `while (flag != 1)` trong `t1`. Có vẻ như có một đảm bảo rằng dòng `std::cout << "b = " << b << std::endl;` sẽ không được thực thi trước khi dấu hiệu được thay đổi. Về mặt logic, có vẻ như giá trị của `b` nên bằng 5.
Nhưng tình huống thực tế phức tạp hơn nhiều, hoặc mã này tự nó là hành vi không xác định vì, đối với `a` và `flag`, chúng được đọc và ghi trong hai luồng song song.
Đã có sự cạnh tranh. Ngoài ra, ngay cả khi chúng ta bỏ qua việc cạnh tranh đọc và ghi, vẫn có thể xảy ra việc thực thi không theo thứ tự của CPU và tác động của trình biên dịch lên việc sắp xếp lại các lệnh.
Khiến `a = 5` xảy ra sau `flag = 1`. Do đó, `b` có thể xuất ra 0.

### Hoạt Động Nguyên Tử

`std::mutex` có thể giải quyết vấn đề đọc và ghi đồng thời, nhưng mutex là một chức năng cấp hệ điều hành.
Điều này là do việc triển khai một mutex thường bao gồm hai nguyên tắc cơ bản:

1. Cung cấp chuyển đổi trạng thái tự động giữa các luồng, tức là trạng thái "khóa"
2. Đảm bảo rằng bộ nhớ của biến bị thao tác được cách ly khỏi vùng quan trọng trong quá trình hoạt động của mutex

Đây là một tập hợp các điều kiện đồng bộ hóa rất mạnh mẽ, nói cách khác khi nó cuối cùng được biên dịch thành một lệnh CPU, nó sẽ hoạt động như nhiều lệnh (chúng ta sẽ xem cách triển khai một mutex đơn giản sau).
Điều này có vẻ quá khắt khe đối với một biến chỉ yêu cầu các thao tác nguyên tử (không có trạng thái trung gian).

Nghiên cứu về các điều kiện đồng bộ hóa đã có một lịch sử rất lâu đời, và chúng ta sẽ không đi vào chi tiết ở đây. Độc giả nên hiểu rằng dưới kiến trúc CPU hiện đại, các thao tác nguyên tử ở cấp độ lệnh CPU được cung cấp.
Do đó, mẫu `std::atomic` được giới thiệu trong C++11 cho chủ đề đọc và ghi biến chia sẻ đa luồng, cho phép chúng ta khởi tạo các kiểu nguyên tử,
và giảm thiểu một thao tác đọc hoặc ghi nguyên tử từ một tập hợp các lệnh xuống một lệnh CPU duy nhất. Ví dụ:

```cpp
std::atomic<int> counter;
```

Và cung cấp các hàm thành viên số học cơ bản cho các kiểu nguyên tử của số nguyên hoặc số thực, chẳng hạn như,
bao gồm `fetch_add`, `fetch_sub`, v.v., và phiên bản tương ứng của `+`, `-` được cung cấp bởi overload.
Ví dụ, ví dụ sau:

```cpp
#include <atomic>
#include <thread>
#include <iostream>

std::atomic<int> count = {0};

int main() {
    std::thread t1([](){
        count.fetch_add(1);
    });
    std::thread t2([](){
        count++;        // tương tự như fetch_add
        count += 1;     // tương tự như fetch_add
    });
    t1.join();
    t2.join();
    std::cout << count << std::endl;
    return 0;
}
```

Tất nhiên, không phải tất cả các kiểu đều cung cấp các thao tác nguyên tử vì tính khả thi của các thao tác nguyên tử phụ thuộc vào kiến trúc của CPU và liệu cấu trúc kiểu được khởi tạo có đáp ứng các yêu cầu căn chỉnh bộ nhớ của kiến trúc hay không, vì vậy chúng ta luôn có thể sử dụng `std::atomic<T>::is_lock_free` để kiểm tra xem kiểu nguyên tử có cần hỗ trợ các thao tác nguyên tử hay không, ví dụ:

```cpp
#include <atomic>
#include <iostream>

struct A {
    float x;
    int y;
    long long z;
};

int main() {
    std::atomic<A> a;
    std::cout << std::boolalpha << a.is_lock_free() << std::endl;
    return 0;
}
```

### Mô Hình Nhất Quán

Nhiều luồng thực thi song song, khi được thảo luận ở một mức độ vĩ mô, có thể được coi là một hệ thống phân tán.
Trong một hệ thống phân tán, bất kỳ giao tiếp hoặc thậm chí là hoạt động cục bộ nào cũng mất một khoảng thời gian nhất định, và thậm chí có thể xảy ra giao tiếp không đáng tin cậy.

Nếu chúng ta buộc hoạt động của một biến `v` giữa nhiều luồng phải là nguyên tử, tức là bất kỳ luồng nào sau khi thực hiện thao tác trên `v`
Các luồng khác có thể **đồng bộ hóa** để nhận biết sự thay đổi của `v`, đối với biến `v`, điều này xuất hiện như một chương trình thực thi tuần tự, nó không có bất kỳ lợi ích hiệu suất nào do việc giới thiệu đa luồng. Có cách nào để tăng tốc điều này một cách hợp lý không? Câu trả lời là làm suy yếu các điều kiện đồng bộ hóa giữa các tiến trình trong các thao tác nguyên tử.

Về nguyên tắc, mỗi luồng có thể tương ứng với một nút cụm, và giao tiếp giữa các luồng gần như tương đương với giao tiếp giữa các nút cụm.
Làm suy yếu các điều kiện đồng bộ hóa giữa các tiến trình, thường chúng ta sẽ xem xét bốn mô hình nhất quán khác nhau:

1. Nhất quán tuyến tính: Còn được gọi là nhất quán mạnh hoặc nhất quán nguyên tử. Nó yêu cầu bất kỳ thao tác đọc nào cũng có thể đọc được lần ghi gần nhất của một dữ liệu nhất định, và thứ tự thao tác của tất cả các luồng phải nhất quán với thứ tự dưới đồng hồ toàn cục.

   ```
           x.store(1)      x.load()
   T1 ---------+----------------+------>


   T2 -------------------+------------->
                   x.store(2)
   ```

   Trong trường hợp này, luồng `T1`, `T2` thực hiện hai lần thao tác nguyên tử lên `x`, và `x.store(1)` xảy ra trước `x.store(2)`. `x.store(2)` xảy ra trước `x.load()`. Đáng chú ý là yêu cầu nhất quán tuyến tính đối với đồng hồ toàn cục rất khó đạt được, đó là lý do tại sao người ta tiếp tục nghiên cứu các thuật toán nhất quán khác dưới điều kiện nhất quán yếu hơn.

2. Nhất quán tuần tự: Cũng yêu cầu bất kỳ thao tác đọc nào cũng có thể đọc được dữ liệu cuối cùng được ghi bởi dữ liệu đó, nhưng không yêu cầu phải nhất quán với thứ tự của đồng hồ toàn cục.

   ```
           x.store(1)  x.store(3)   x.load()
   T1 ---------+-----------+----------+----->


   T2 ---------------+---------------------->
                 x.store(2)

   hoặc

           x.store(1)  x.store(3)   x.load()
   T1 ---------+-----------+----------+----->


   T2 ------+------------------------------->
         x.store(2)
   ```

   Dưới yêu cầu nhất quán tuần tự, `x.load()` phải đọc dữ liệu cuối cùng được ghi, vì vậy `x.store(2)` và `x.store(1)` không có bất kỳ đảm bảo nào, miễn là `x.store(2)` của `T2` xảy ra trước `x.store(3)`.

3. Nhất quán nhân quả: Yêu cầu của nó được giảm bớt hơn nữa, chỉ đảm bảo thứ tự của các thao tác nhân quả, và không yêu cầu thứ tự của các thao tác không nhân quả.

   ```
         a = 1      b = 2
   T1 ----+-----------+---------------------------->


   T2 ------+--------------------+--------+-------->
         x.store(3)         c = a + b    y.load()

   hoặc

         a = 1      b = 2
   T1 ----+-----------+---------------------------->


   T2 ------+--------------------+--------+-------->
         x.store(3)          y.load()   c = a + b

   hoặc

        b = 2       a = 1
   T1 ----+-----------+---------------------------->


   T2 ------+--------------------+--------+-------->
         y.load()            c = a + b  x.store(3)
   ```

   Ba ví dụ trên đều nhất quán nhân quả vì, trong toàn bộ quá trình, chỉ có `c` phụ thuộc vào `a` và `b`, và `x` và `y` không liên quan trong ví dụ này. (Nhưng trong tình huống thực tế, chúng ta cần thông tin chi tiết hơn để xác định rằng `x` không liên quan đến `y`)


   4. Nhất Quán Cuối Cùng: Đây là yêu cầu nhất quán yếu nhất. Nó chỉ đảm bảo rằng một thao tác sẽ được quan sát tại một thời điểm nào đó trong tương lai, nhưng không yêu cầu thời gian quan sát cụ thể. Vì vậy, chúng ta thậm chí có thể tăng cường điều kiện này một chút, ví dụ, để chỉ định rằng thời gian quan sát cho một thao tác luôn bị giới hạn. Tất nhiên, điều này không còn nằm trong phạm vi thảo luận của chúng ta.

   ```
       x.store(3)  x.store(4)
   T1 ----+-----------+-------------------------------------------->


   T2 ---------+------------+--------------------+--------+-------->
            x.read()      x.read()           x.read()   x.read()
   ```

   Trong trường hợp trên, nếu chúng ta giả định rằng giá trị ban đầu của x là 0, thì bốn lần ``x.read()` trong `T2` có thể nhưng không giới hạn ở các kết quả sau:

   ```
   3 4 4 4 // Thao tác ghi của x được quan sát nhanh chóng
   0 3 3 4 // Có độ trễ trong thời gian quan sát thao tác ghi của x
   0 0 0 4 // Lần đọc cuối cùng đọc giá trị cuối cùng của x,
           // nhưng các thay đổi trước đó không được quan sát.
   0 0 0 0 // Thao tác ghi của x không được quan sát trong khoảng thời gian hiện tại,
           // nhưng tình huống x là 4 có thể được quan sát
           // tại một thời điểm nào đó trong tương lai.
   ```

   ### Thứ Tự Bộ Nhớ

Để đạt được hiệu suất tối ưu và đảm bảo tính nhất quán với các yêu cầu khác nhau, C++11 định nghĩa sáu thứ tự bộ nhớ khác nhau cho các thao tác nguyên tử. Tùy chọn `std::memory_order` biểu thị bốn mô hình đồng bộ hóa giữa nhiều luồng:

1. Mô hình thư giãn: Trong mô hình này, các thao tác nguyên tử trong một luồng được thực hiện tuần tự và không cho phép sắp xếp lại lệnh, nhưng thứ tự của các thao tác nguyên tử giữa các luồng khác nhau là tùy ý. Kiểu này được chỉ định bởi `std::memory_order_relaxed`. Hãy xem một ví dụ:

   ```cpp
   std::atomic<int> counter = {0};
   std::vector<std::thread> vt;
   for (int i = 0; i < 100; ++i) {
       vt.emplace_back([&](){
           counter.fetch_add(1, std::memory_order_relaxed);
       });
   }

   for (auto& t : vt) {
       t.join();
   }
   std::cout << "current counter:" << counter << std::endl;
   ```

2. Mô hình phát hành/tiêu thụ: Trong mô hình này, chúng ta bắt đầu giới hạn thứ tự các thao tác giữa các tiến trình. Nếu một luồng cần sửa đổi một giá trị, nhưng một luồng khác sẽ phụ thuộc vào thao tác đó của giá trị, tức là, luồng sau phụ thuộc vào luồng trước. Cụ thể, luồng A đã hoàn thành ba lần ghi vào `x`, và luồng `B` chỉ phụ thuộc vào thao tác ghi thứ ba của `x`, không quan tâm đến hai thao tác ghi đầu tiên của `x`, thì khi `A` thực hiện `x.release()` (tức là sử dụng `std::memory_order_release`), tùy chọn `std::memory_order_consume` đảm bảo rằng `B` quan sát thấy ba lần ghi vào `x` của `A` khi gọi `x.load()`. Hãy xem một ví dụ:


   ```cpp
   // khởi tạo là nullptr để ngăn người tiêu dùng tải một con trỏ lủng lẳng
   std::atomic<int*> ptr(nullptr);
   int v;
   std::thread producer([&]() {
       int* p = new int(42);
       v = 1024;
       ptr.store(p, std::memory_order_release);
   });
   std::thread consumer([&]() {
       int* p;
       while(!(p = ptr.load(std::memory_order_consume)));

       std::cout << "p: " << *p << std::endl;
       std::cout << "v: " << v << std::endl;
   });
   producer.join();
   consumer.join();
   ```

3. Mô hình Phát hành/Đạt được: Dưới mô hình này, chúng ta có thể thắt chặt hơn thứ tự của các thao tác nguyên tử giữa các luồng khác nhau, bằng cách chỉ định thời điểm giữa phát hành `std::memory_order_release` và đạt được `std::memory_order_acquire`. **Tất cả** các thao tác ghi trước thao tác phát hành sẽ hiển thị cho bất kỳ luồng nào khác, tức là xảy ra trước.

   Như bạn có thể thấy, `std::memory_order_release` đảm bảo rằng một thao tác ghi trước khi phát hành không xảy ra sau thao tác phát hành, đây là một **rào cản ngược**, và `std::memory_order_acquire` đảm bảo rằng một thao tác đọc hoặc ghi sau khi đạt được không xảy ra trước thao tác đạt được, đây là một **rào cản tiến**.
   Đối với tùy chọn `std::memory_order_acq_rel`, nó kết hợp các đặc điểm của cả hai rào cản và xác định một rào cản bộ nhớ duy nhất, sao cho các thao tác đọc và ghi của luồng hiện tại sẽ không bị sắp xếp lại qua rào cản.

   Hãy xem một ví dụ:

   ```cpp
   std::vector<int> v;
   std::atomic<int> flag = {0};
   std::thread release([&]() {
       v.push_back(42);
       flag.store(1, std::memory_order_release);
   });
   std::thread acqrel([&]() {
       int expected = 1; // phải trước compare_exchange_strong
       while(!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) 
           expected = 1; // phải sau compare_exchange_strong
       // flag đã thay đổi thành 2
   });
   std::thread acquire([&]() {
       while(flag.load(std::memory_order_acquire) < 2);

       std::cout << v.at(0) << std::endl; // phải là 42
   });
   release.join();
   acqrel.join();
   acquire.join();
   ```

   Trong trường hợp này, chúng ta đã sử dụng `compare_exchange_strong`, đây là nguyên thủy So sánh và Hoán đổi, có một phiên bản yếu hơn là `compare_exchange_weak`, cho phép trả về thất bại ngay cả khi hoán đổi thành công. Lý do là do một thất bại giả trên một số nền tảng, cụ thể là khi CPU thực hiện chuyển đổi ngữ cảnh, một luồng khác tải cùng địa chỉ để tạo ra sự không nhất quán. Ngoài ra, hiệu suất của `compare_exchange_strong` có thể hơi kém hơn so với `compare_exchange_weak`. Tuy nhiên, trong hầu hết các trường hợp, `compare_exchange_weak` không được khuyến khích do sự phức tạp trong việc sử dụng nó.

 4. Mô hình Nhất Quán Tuần Tự: Dưới mô hình này, các thao tác nguyên tử đảm bảo tính nhất quán tuần tự, điều này có thể gây ra mất hiệu suất. Nó có thể được chỉ định rõ ràng bằng `std::memory_order_seq_cst`. Hãy xem một ví dụ cuối cùng:

   ```cpp
   std::atomic<int> counter = {0};
   std::vector<std::thread> vt;
   for (int i = 0; i < 100; ++i) {
       vt.emplace_back([&](){
           counter.fetch_add(1, std::memory_order_seq_cst);
       });
   }

   for (auto& t : vt) {
       t.join();
   }
   std::cout << "current counter:" << counter << std::endl;
   ```

   Ví dụ này về cơ bản giống với ví dụ mô hình lỏng lẻo đầu tiên. Chỉ cần thay đổi thứ tự bộ nhớ của thao tác nguyên tử thành `memory_order_seq_cst`. Độc giả quan tâm có thể tự viết chương trình để đo lường sự khác biệt về hiệu suất do hai thứ tự bộ nhớ khác nhau này gây ra.

## Kết Luận

Ngôn ngữ C++11 cung cấp hỗ trợ cho lập trình đồng thời. Phần này giới thiệu ngắn gọn về `std::thread`/`std::mutex`/`std::future`, những công cụ quan trọng không thể tránh khỏi trong lập trình đồng thời.
Ngoài ra, chúng ta cũng đã giới thiệu "mô hình bộ nhớ" như một trong những tính năng quan trọng nhất của C++11.
Chúng cung cấp nền tảng quan trọng cho tính toán hiệu suất cao tiêu chuẩn hóa cho C++.
## Bài Tập

1. Viết một thread pool đơn giản cung cấp các tính năng sau:

   ```cpp
   ThreadPool p(4); // chỉ định bốn luồng làm việc

   // đưa một tác vụ vào hàng đợi và trả về một std::future
   auto f = pool.enqueue([](int life) {
       return meaning;
   }, 42);

   // lấy kết quả từ future
   std::cout << f.get() << std::endl;
   ```

2. Sử dụng `std::atomic<bool>` để triển khai một mutex.

[Table of Content](./toc.md) | [Chương Trước](./06-regex.md) | [Chương Tiếp Theo: Hệ Thống Tập Tin](./08-filesystem.md)

## Đọc Thêm

- [C++ Concurrency in Action](https://www.amazon.com/dp/1617294691/ref=cm_sw_em_r_mt_dp_U_siEmDbRMMF960)
- [Tài liệu về Thread](https://en.cppreference.com/w/cpp/thread)
- Herlihy, M. P., & Wing, J. M. (1990). Linearizability: a correctness condition for concurrent objects. ACM Transactions on Programming Languages and Systems, 12(3), 463–492. https://doi.org/10.1145/78969.78972

## Giấy Phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy Phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy Phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).
