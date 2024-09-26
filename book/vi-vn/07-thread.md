---
title: "Chương 07: Lập trình song song và đồng thời"
type: book-vi-vn
order: 7
---

# Chương 07: Lập trình song song và đồng thời

[TOC]

## 7.1 Cơ bản về lập trình song song

`std::thread` được sử dụng để tạo ra một thể hiện của luồng,
nó là nền tảng cho tất cả các chương trình đồng thời.
Để sử dụng `std::thread`, bạn cần include header file `<thread>`.
Lớp `std::thread` cung cấp một số phương thức cơ bản để thao tác với luồng,
ví dụ như `get_id()` để lấy ID của luồng, `join()` để chờ luồng kết thúc, v.v.
Ví dụ:

```cpp
#include <iostream>
#include <thread>

int main() {
  std::thread t([]() { std::cout << "hello world." << std::endl; });
  t.join();
  return 0;
}
```

## 7.2 Mutex và đoạn mã tới hạn

Chúng ta đã được học về các khái niệm cơ bản của lập trình đồng thời
trong hệ điều hành và cơ sở dữ liệu. Mutex là một trong những thành phần cốt lõi
của lập trình đồng thời.
C++11 giới thiệu một số lớp liên quan đến mutex trong header file `<mutex>`.

`std::mutex` là lớp mutex cơ bản nhất trong C++11.
Để tạo ra một mutex, bạn có thể khởi tạo một đối tượng `std::mutex`.
Bạn có thể khóa mutex bằng phương thức `lock()` và mở khóa bằng phương thức `unlock()`.
Tuy nhiên, trong thực tế, bạn nên tránh gọi trực tiếp các phương thức này.
Bởi vì nếu bạn gọi trực tiếp, bạn cần phải nhớ gọi `unlock()` khi kết thúc
đoạn mã tới hạn, bao gồm cả khi có ngoại lệ xảy ra.
Để đơn giản hóa việc sử dụng mutex, C++11 cung cấp lớp `std::lock_guard`
dựa trên kỹ thuật RAII.

RAII giúp đảm bảo an toàn cho mã nguồn khi có ngoại lệ xảy ra,
đồng thời giúp mã nguồn trở nên đơn giản hơn.

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
  static std::mutex mtx;
  std::lock_guard<std::mutex> lock(mtx);

  // Thực hiện công việc cần đồng bộ
  v = change_v;

  // Mutex mtx sẽ được tự động mở khóa khi lock ra khỏi phạm vi
}

int main() {
  std::thread t1(critical_section, 2), t2(critical_section, 3);
  t1.join();
  t2.join();

  std::cout << v << std::endl;
  return 0;
}
```

Do C++ đảm bảo rằng tất cả các đối tượng trên stack sẽ bị hủy khi ra khỏi phạm vi,
nên đoạn mã trên rất an toàn. Cho dù hàm `critical_section()` trả về bình thường
hay có ngoại lệ xảy ra, mutex `mtx` cũng sẽ được tự động mở khóa khi đối tượng
`lock` bị hủy.

> Lưu ý: Hành vi của chương trình khi một ngoại lệ bị ném ra nhưng không bị bắt
> là không xác định.

`std::unique_lock` linh hoạt hơn `std::lock_guard`.
Một đối tượng `std::unique_lock` quản lý việc khóa và mở khóa một mutex,
và chỉ có duy nhất một `std::unique_lock` có thể sở hữu một mutex
tại một thời điểm. Do đó, trong lập trình đồng thời, bạn nên sử dụng `std::unique_lock`.

Khác với `std::lock_guard`, `std::unique_lock` cho phép bạn gọi `lock`
và `unlock` một cách tường minh tại bất kỳ vị trí nào trong mã nguồn.
Điều này cho phép bạn thu hẹp phạm vi của đoạn mã tới hạn,
từ đó tăng hiệu quả của chương trình đồng thời.

Nếu bạn sử dụng hàm `std::condition_variable::wait`,
bạn bắt buộc phải sử dụng `std::unique_lock` làm tham số.

Ví dụ:

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int v = 1;

void critical_section(int change_v) {
  static std::mutex mtx;
  std::unique_lock<std::mutex> lock(mtx);
  // Thực hiện công việc cần đồng bộ
  v = change_v;
  std::cout << v << std::endl;
  // Mở khóa mutex
  lock.unlock();

  // Trong khoảng thời gian này, các luồng khác có thể truy cập v

  // Thực hiện tiếp công việc cần đồng bộ
  // Khóa mutex
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

`std::future` là một lớp đại diện cho một giá trị trong tương lai,
nó cung cấp cơ chế để truy cập kết quả của các tác vụ bất đồng bộ.
Để hiểu rõ hơn về `std::future`, chúng ta cần tìm hiểu cách xử lý kết quả
của các tác vụ bất đồng bộ trước C++11.

Giả sử luồng chính A muốn tạo ra một luồng mới B để thực hiện một tác vụ
và trả về kết quả.
Trong khi luồng B đang thực hiện tác vụ, luồng A có thể bận xử lý các công việc khác
và không thể nhận kết quả ngay lập tức.
Do đó, chúng ta cần một cơ chế để lấy kết quả của luồng B
tại một thời điểm nào đó trong tương lai.

Trước khi `std::future` được giới thiệu trong C++11, cách làm thường thấy là:
Tạo một luồng A, khởi chạy tác vụ B trong luồng A,
gửi một tín hiệu (event) khi tác vụ hoàn thành và lưu kết quả vào một biến toàn cục.
Luồng chính A tiếp tục xử lý các công việc khác.
Khi cần kết quả của tác vụ B, luồng A sẽ chờ tín hiệu từ luồng B
và lấy kết quả từ biến toàn cục.

`std::future` trong C++11 đơn giản hóa quá trình này.
Nó có thể được sử dụng để lấy kết quả của các tác vụ bất đồng bộ
tại một thời điểm nào đó trong tương lai.
Bạn có thể hình dung `std::future` như một cơ chế đồng bộ hóa luồng,
tương tự như barrier.

Dưới đây là một ví dụ sử dụng `std::future` kết hợp với `std::packaged_task`
để lấy kết quả của một tác vụ bất đồng bộ.
`std::packaged_task` được sử dụng để đóng gói một tác vụ bất kỳ,
ví dụ như hàm hoặc biểu thức lambda, để có thể thực thi nó một cách bất đồng bộ.

```cpp
#include <future>
#include <iostream>
#include <thread>

int main() {
  // Đóng gói một biểu thức lambda trả về 7 vào std::packaged_task
  std::packaged_task<int()> task([]() { return 7; });
  // Lấy std::future từ task
  std::future<int> result = task.get_future();
  // Chạy task trong một luồng riêng biệt
  std::thread(std::move(task)).detach();
  std::cout << "Đang chờ...";
  result.wait(); // Chờ cho đến khi có kết quả
  // In kết quả
  std::cout << "Xong!" << std::endl
            << "Kết quả: " << result.get() << std::endl;
  return 0;
}
```

Sau khi đóng gói tác vụ, bạn có thể sử dụng `get_future()` để lấy một
đối tượng `std::future` và sử dụng nó để đồng bộ hóa luồng.

## 7.4 Biến điều kiện

Biến điều kiện (`std::condition_variable`) được sinh ra để giải quyết
vấn đề deadlock và được sử dụng khi mutex không đủ.
Ví dụ, một luồng có thể cần phải chờ một điều kiện nào đó trở thành đúng
để tiếp tục thực thi.
Nếu luồng này sử dụng vòng lặp `while` để chờ đợi,
nó sẽ chiếm giữ mutex, khiến các luồng khác không thể truy cập
vào đoạn mã tới hạn và dẫn đến deadlock.
Vì vậy, `std::condition_variable` được tạo ra để đánh thức các luồng đang chờ
và tránh deadlock.
Phương thức `notify_one()` của `std::condition_variable` được dùng để
đánh thức một luồng đang chờ;
phương thức `notify_all()` được dùng để đánh thức tất cả các luồng đang chờ.

Dưới đây là ví dụ về mô hình nhà sản xuất - người tiêu dùng
sử dụng `std::condition_variable`:

```cpp
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>

int main() {
  std::queue<int> produced_nums;
  std::mutex mtx;
  std::condition_variable cv;
  bool notified = false; // Cờ hiệu báo hiệu có dữ liệu mới

  auto producer = [&]() {
    for (int i = 0;; i++) {
      std::this_thread::sleep_for(std::chrono::milliseconds(900));
      std::unique_lock<std::mutex> lock(mtx);
      std::cout << "Đang sản xuất " << i << std::endl;
      produced_nums.push(i);
      notified = true;
      cv.notify_all(); // Thông báo cho tất cả các luồng đang chờ
    }
  };

  auto consumer = [&]() {
    while (true) {
      std::unique_lock<std::mutex> lock(mtx);
      while (!notified) { // Chờ cho đến khi có dữ liệu mới
        cv.wait(lock);
      }
      // Tạm thời mở khóa mutex để cho phép nhà sản xuất tạo ra thêm dữ liệu
      lock.unlock();
      // Giả sử người tiêu dùng xử lý chậm hơn nhà sản xuất
      std::this_thread::sleep_for(std::chrono::milliseconds(1000));
      // Khóa mutex
      lock.lock();
      if (!produced_nums.empty()) {
        std::cout << "Đang tiêu thụ " << produced_nums.front() << std::endl;
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

Đáng chú ý là mặc dù chúng ta có thể sử dụng `notify_one()` trong nhà sản xuất,
nhưng không nên dùng nó trong ví dụ này.
Bởi vì khi có nhiều người tiêu dùng, việc sử dụng `notify_all()` sẽ giúp
tất cả người tiêu dùng đều có cơ hội được đánh thức và xử lý dữ liệu.
Tuy nhiên, do mutex `mtx` là độc quyền, nên chỉ có một người tiêu dùng
có thể truy cập vào hàng đợi `produced_nums` tại một thời điểm.
Để giải quyết vấn đề này, chúng ta cần sử dụng các kỹ thuật đồng bộ hóa
ở mức độ chi tiết hơn.

## 7.5 Thao tác nguyên tử và mô hình bộ nhớ

Độc giả tinh ý có thể nhận thấy rằng ví dụ về mô hình nhà sản xuất -
người tiêu dùng ở phần trước có thể gặp lỗi do tối ưu hóa của trình biên dịch.
Ví dụ, trình biên dịch có thể tối ưu hóa biến `notified` bằng cách lưu
giá trị của nó vào một thanh ghi.
Điều này có thể dẫn đến việc luồng người tiêu dùng không bao giờ nhìn thấy
sự thay đổi của `notified` và rơi vào trạng thái chờ đợi vô tận.
Để giải thích vấn đề này, chúng ta cần tìm hiểu thêm về mô hình bộ nhớ
trong C++11.

Trước tiên, hãy xem xét đoạn mã sau và đoán xem đầu ra của nó là gì:

```cpp
#include <iostream>
#include <thread>

int main() {
  int a = 0;
  volatile int flag = 0;

  std::thread t1([&]() {
    while (flag != 1)
      ;

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

Thoạt nhìn, có vẻ như `a = 5;` trong luồng `t2` luôn được thực thi trước
`flag = 1;` và vòng lặp `while (flag != 1)` trong luồng `t1` sẽ kết thúc
ngay sau khi `flag` được gán giá trị 1.
Do đó, chúng ta có thể nghĩ rằng giá trị của `b` sẽ luôn là 5.
Tuy nhiên, thực tế phức tạp hơn nhiều.
Đoạn mã trên có hành vi không xác định, bởi vì cả hai luồng `t1` và `t2`
đều truy cập và thay đổi biến `a` và `flag` dẫn đến data race.
Ngoài ra, ngay cả khi chúng ta bỏ qua vấn đề data race,
việc thực thi không theo thứ tự của CPU và việc sắp xếp lại lệnh của trình biên dịch
cũng có thể khiến `a = 5` được thực thi sau `flag = 1`,
và `b` có thể có giá trị là 0.

### Thao tác nguyên tử

`std::mutex` có thể giải quyết vấn đề data race, nhưng mutex là một cơ chế
ở cấp độ hệ điều hành.
Mutex thường được triển khai dựa trên hai nguyên tắc:

1. Cung cấp cơ chế chuyển đổi trạng thái giữa các luồng,
   ví dụ: trạng thái "đã khóa" và "chưa khóa".
2. Đảm bảo rằng chỉ có một luồng có thể truy cập vào biến được bảo vệ
   bởi mutex tại một thời điểm.

Mutex là một cơ chế đồng bộ hóa mạnh,
nó thường được biên dịch thành nhiều lệnh CPU.
Đối với các biến chỉ yêu cầu thao tác nguyên tử (không có trạng thái trung gian),
việc sử dụng mutex có thể gây ra lãng phí hiệu năng.

Nghiên cứu về các điều kiện đồng bộ hóa đã có từ lâu.
Trong kiến trúc CPU hiện đại, các thao tác nguyên tử được hỗ trợ ở cấp độ phần cứng.
C++11 giới thiệu template class `std::atomic` để hỗ trợ các thao tác nguyên tử
trên các biến được chia sẻ giữa nhiều luồng.
Sử dụng `std::atomic`, chúng ta có thể khởi tạo các biến nguyên tử
và giảm thiểu số lượng lệnh CPU cần thiết để thực hiện
các thao tác đọc và ghi nguyên tử.
Ví dụ:

```cpp
std::atomic<int> counter;
```

`std::atomic` cung cấp một số phương thức để thực hiện các thao tác nguyên tử
trên các kiểu số nguyên và số thực,
ví dụ như `fetch_add`, `fetch_sub`, v.v.
Các toán tử `+`, `-` cũng được overload để thực hiện các thao tác nguyên tử.
Ví dụ:

```cpp
#include <atomic>
#include <iostream>
#include <thread>

std::atomic<int> count = {0};

int main() {
  std::thread t1([]() { count.fetch_add(1); });
  std::thread t2([]() {
    count++;        // Tương đương với count.fetch_add(1)
    count += 1;     // Tương đương với count.fetch_add(1)
  });
  t1.join();
  t2.join();
  std::cout << count << std::endl;
  return 0;
}
```

Không phải tất cả các kiểu dữ liệu đều hỗ trợ thao tác nguyên tử.
Khả năng hỗ trợ thao tác nguyên tử phụ thuộc vào kiến trúc CPU
và cách thức dữ liệu được lưu trữ trong bộ nhớ.
Bạn có thể sử dụng phương thức `std::atomic<T>::is_lock_free()` để kiểm tra
xem một kiểu dữ liệu `T` có hỗ trợ thao tác nguyên tử hay không.
Ví dụ:

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

### Mô hình nhất quán

Ở mức độ vĩ mô, một chương trình đa luồng có thể được coi là một hệ thống phân tán.
Trong hệ thống phân tán, mọi hoạt động giao tiếp, thậm chí cả các hoạt động cục bộ,
đều mất một khoảng thời gian nhất định để hoàn thành, và có thể xảy ra lỗi
trong quá trình giao tiếp.

Nếu chúng ta yêu cầu mọi thao tác trên một biến `v` phải được thực hiện
một cách nguyên tử và được đồng bộ hóa ngay lập tức giữa tất cả các luồng,
thì chương trình sẽ không đạt được hiệu quả mong muốn từ việc sử dụng đa luồng.
Vậy làm cách nào để cải thiện hiệu năng của chương trình đa luồng?
Câu trả lời là nới lỏng các điều kiện đồng bộ hóa giữa các luồng
trong các thao tác nguyên tử.

Về nguyên tắc, mỗi luồng có thể được coi là một nút trong hệ thống phân tán,
và giao tiếp giữa các luồng tương đương với giao tiếp giữa các nút.
Khi nới lỏng các điều kiện đồng bộ hóa, chúng ta thường xem xét
bốn mô hình nhất quán sau:

1. **Nhất quán tuyến tính (linearizability):**
   còn được gọi là nhất quán mạnh (strong consistency)
   hoặc nhất quán nguyên tử (atomic consistency).
   Mô hình này yêu cầu mọi thao tác đọc phải trả về giá trị mới nhất
   của biến, và thứ tự thực hiện các thao tác của tất cả các luồng phải nhất quán
   với thứ tự theo một đồng hồ toàn cục.

   ```
           x.store(1)      x.load()
   T1 ---------+----------------+------>


   T2 -------------------+------------->
                   x.store(2)
   ```

   Trong ví dụ trên, luồng `T1` và `T2` thực hiện hai thao tác ghi nguyên tử
   lên biến `x`. `x.store(1)` được thực hiện trước `x.store(2)`,
   và `x.store(2)` được thực hiện trước `x.load()`.
   Nhất quán tuyến tính yêu cầu phải có một đồng hồ toàn cục để đảm bảo
   thứ tự thực hiện các thao tác,
   điều này rất khó thực hiện trong thực tế.
   Chính vì vậy, các mô hình nhất quán khác đã được nghiên cứu
   dựa trên việc nới lỏng các điều kiện của nhất quán tuyến tính.

2. **Nhất quán tuần tự (sequential consistency):**
   cũng yêu cầu mọi thao tác đọc phải trả về giá trị mới nhất của biến,
   nhưng không yêu cầu thứ tự thực hiện các thao tác phải nhất quán
   với một đồng hồ toàn cục.

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

   Trong mô hình nhất quán tuần tự, `x.load()` phải trả về giá trị
   của lần ghi cuối cùng.
   Thứ tự thực hiện của `x.store(1)` và `x.store(2)` không được đảm bảo,
   miễn là `x.store(2)` trong luồng `T2` được thực hiện trước `x.store(3)`.

3. **Nhất quán nhân quả (causal consistency):**
   yêu cầu thứ tự thực hiện của các thao tác có quan hệ nhân quả
   phải được đảm bảo,
   nhưng không yêu cầu thứ tự thực hiện của các thao tác không có
   quan hệ nhân quả.

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

   Cả ba ví dụ trên đều tuân theo nhất quán nhân quả, bởi vì
   chỉ có thao tác `c = a + b` phụ thuộc vào kết quả của các thao tác
   `a = 1` và `b = 2`.
   Các thao tác `x.store(3)` và `y.load()` không có quan hệ nhân quả
   với các thao tác khác.
   (Tuy nhiên, trong thực tế, chúng ta cần thêm thông tin để xác định
   chắc chắn rằng `x.store(3)` và `y.load()` không có quan hệ nhân quả).

4. **Nhất quán cuối cùng (eventual consistency):**
   là mô hình nhất quán yếu nhất.
   Nó chỉ đảm bảo rằng một thao tác sẽ được nhìn thấy bởi tất cả
   các luồng tại một thời điểm nào đó trong tương lai,
   nhưng không yêu cầu thời điểm cụ thể.
   Chúng ta có thể bổ sung thêm điều kiện cho mô hình này,
   ví dụ như yêu cầu thời điểm nhìn thấy kết quả của một thao tác
   phải nằm trong một khoảng thời gian giới hạn.
   Tuy nhiên, chúng ta sẽ không đi sâu vào chi tiết trong chương này.

   ```
       x.store(3)  x.store(4)
   T1 ----+-----------+-------------------------------------------->


   T2 ---------+------------+--------------------+--------+-------->
            x.read()      x.read()           x.read()   x.read()
   ```

   Trong ví dụ trên, nếu giá trị ban đầu của `x` là 0,
   thì bốn lần đọc `x.read()` trong luồng `T2` có thể trả về
   các giá trị sau (không giới hạn):

   ```
   3 4 4 4 // Các thao tác ghi được nhìn thấy ngay lập tức
   0 3 3 4 // Có độ trễ khi quan sát các thao tác ghi
   0 0 0 4 // Lần đọc cuối cùng trả về giá trị cuối cùng của x,
           // nhưng các thay đổi trước đó không được nhìn thấy
   0 0 0 0 // Các thao tác ghi chưa được nhìn thấy trong khoảng thời gian này,
           // nhưng giá trị cuối cùng của x (là 4) sẽ được nhìn thấy
           // tại một thời điểm nào đó trong tương lai.
   ```

### Thứ tự bộ nhớ (memory order)

Để đạt được hiệu năng tối ưu và hỗ trợ các mô hình nhất quán khác nhau,
C++11 định nghĩa sáu thứ tự bộ nhớ khác nhau cho các thao tác nguyên tử.
Enum `std::memory_order` được sử dụng để chỉ định thứ tự bộ nhớ
cho các thao tác nguyên tử, nó đại diện cho bốn mô hình đồng bộ hóa sau:

1. **Mô hình thư giãn (relaxed ordering):**
   trong mô hình này, các thao tác nguyên tử trong cùng một luồng
   được thực hiện tuần tự, nhưng thứ tự thực hiện của các thao tác nguyên tử
   giữa các luồng khác nhau là không xác định.
   Thứ tự bộ nhớ này được biểu diễn bởi hằng số `std::memory_order_relaxed`.
   Ví dụ:

   ```cpp
   std::atomic<int> counter = {0};
   std::vector<std::thread> vt;
   for (int i = 0; i < 100; ++i) {
     vt.emplace_back([&]() {
       counter.fetch_add(1, std::memory_order_relaxed);
     });
   }

   for (auto &t : vt) {
     t.join();
   }
   std::cout << "Giá trị hiện tại của counter: " << counter << std::endl;
   ```

2. **Mô hình phát hành/tiêu thụ (release-consume ordering):**
   trong mô hình này, chúng ta bắt đầu giới hạn thứ tự thực hiện các thao tác
   nguyên tử giữa các luồng khác nhau.
   Nếu một luồng A thực hiện một thao tác ghi lên biến `x`
   và một luồng B phụ thuộc vào kết quả của thao tác ghi đó,
   thì luồng A cần sử dụng `std::memory_order_release` khi ghi vào `x`,
   và luồng B cần sử dụng `std::memory_order_consume` khi đọc `x`.
   Điều này đảm bảo rằng tất cả các thao tác ghi của luồng A
   trước thao tác ghi sử dụng `std::memory_order_release`
   sẽ được nhìn thấy bởi luồng B.
   Ví dụ:

   ```cpp
   // Khởi tạo ptr là nullptr để tránh trường hợp luồng consumer
   // đọc giá trị của ptr khi nó chưa được gán
   std::atomic<int *> ptr(nullptr);
   int v;
   std::thread producer([&]() {
     int *p = new int(42);
     v = 1024;
     ptr.store(p, std::memory_order_release);
   });
   std::thread consumer([&]() {
     int *p;
     while (!(p = ptr.load(std::memory_order_consume)))
       ;

     std::cout << "p: " << *p << std::endl;
     std::cout << "v: " << v << std::endl;
   });
   producer.join();
   consumer.join();
   ```

3. **Mô hình phát hành/đạt được (release-acquire ordering):**
   trong mô hình này, chúng ta thắt chặt hơn nữa điều kiện đồng bộ hóa
   giữa các luồng.
   `std::memory_order_release` hoạt động như một rào cản,
   đảm bảo rằng tất cả các thao tác ghi trước nó
   sẽ được nhìn thấy bởi các luồng khác sử dụng `std::memory_order_acquire`
   để đọc biến.
   Tương tự, `std::memory_order_acquire` cũng hoạt động như một rào cản,
   đảm bảo rằng tất cả các thao tác đọc và ghi sau nó
   sẽ không được thực hiện trước khi nó hoàn thành.
   `std::memory_order_acq_rel` là sự kết hợp của hai thứ tự bộ nhớ trên,
   nó đảm bảo rằng cả thao tác đọc và ghi đều tuân theo các quy tắc
   của `std::memory_order_release` và `std::memory_order_acquire`.

   Ví dụ:

   ```cpp
   std::vector<int> v;
   std::atomic<int> flag = {0};
   std::thread release([&]() {
     v.push_back(42);
     flag.store(1, std::memory_order_release);
   });
   std::thread acqrel([&]() {
     int expected = 1; // Phải được gán giá trị trước khi gọi compare_exchange_strong
     while (!flag.compare_exchange_strong(expected, 2,
                                         std::memory_order_acq_rel))
       expected = 1; // Phải được gán lại giá trị sau khi gọi compare_exchange_strong
     // flag đã được thay đổi thành 2
   });
   std::thread acquire([&]() {
     while (flag.load(std::memory_order_acquire) < 2)
       ;

     std::cout << v.at(0) << std::endl; // Phải là 42
   });
   release.join();
   acqrel.join();
   acquire.join();
   ```

   Trong ví dụ trên, chúng ta sử dụng phương thức `compare_exchange_strong`,
   đây là một thao tác nguyên tử so sánh và hoán đổi (CAS).
   `compare_exchange_strong` có một phiên bản yếu hơn là `compare_exchange_weak`,
   cho phép thao tác CAS thất bại ngay cả khi điều kiện so sánh là đúng.
   Lý do là do lỗi "false failure" có thể xảy ra trên một số nền tảng,
   ví dụ như khi CPU chuyển đổi ngữ cảnh,
   một luồng khác có thể đọc và ghi vào cùng địa chỉ bộ nhớ,
   gây ra hiện tượng inconsistent.
   Ngoài ra, `compare_exchange_strong` có thể chậm hơn `compare_exchange_weak`.
   Tuy nhiên, trong hầu hết các trường hợp, bạn nên sử dụng `compare_exchange_strong`
   do cách sử dụng đơn giản hơn.

4. **Mô hình nhất quán tuần tự (sequentially consistent ordering):**
   trong mô hình này, các thao tác nguyên tử tuân theo mô hình nhất quán
   tuần tự.
   Thứ tự bộ nhớ này được biểu diễn bởi hằng số `std::memory_order_seq_cst`.
   Nó là thứ tự bộ nhớ mặc định cho các thao tác nguyên tử,
   và thường có hiệu năng thấp nhất.

   Ví dụ:

   ```cpp
   std::atomic<int> counter = {0};
   std::vector<std::thread> vt;
   for (int i = 0; i < 100; ++i) {
     vt.emplace_back([&]() {
       counter.fetch_add(1, std::memory_order_seq_cst);
     });
   }

   for (auto &t : vt) {
     t.join();
   }
   std::cout << "Giá trị hiện tại của counter: " << counter << std::endl;
   ```

   Ví dụ trên về cơ bản giống với ví dụ về mô hình thư giãn,
   chỉ khác là chúng ta đã chỉ định thứ tự bộ nhớ là `std::memory_order_seq_cst`.
   Bạn đọc có thể tự viết chương trình để đo lường sự khác biệt về hiệu năng
   giữa hai thứ tự bộ nhớ này.

## Kết luận

C++11 cung cấp hỗ trợ cho lập trình đồng thời ở cấp độ ngôn ngữ.
Chương này đã giới thiệu sơ lược về `std::thread`, `std::mutex`
và `std::future`, những công cụ quan trọng trong lập trình đồng thời.
Ngoài ra, chúng ta cũng đã tìm hiểu về mô hình bộ nhớ, một tính năng quan trọng
của C++11.
Các tính năng này là nền tảng cho việc xây dựng các chương trình C++ hiệu năng cao
và tuân theo các tiêu chuẩn.

## Bài tập

1. Viết một thread pool đơn giản với các chức năng sau:

   ```cpp
   ThreadPool p(4); // Tạo thread pool với 4 luồng worker

   // Thêm một tác vụ vào thread pool, trả về std::future
   auto f = pool.enqueue([](int life) { return meaning; }, 42);

   // Lấy kết quả từ future
   std::cout << f.get() << std::endl;
   ```

2. Sử dụng `std::atomic<bool>` để triển khai một mutex.

[Mục lục](./toc.md) | [Chương trước](./06-regex.md) | [Chương tiếp theo: Hệ thống tệp](./08-filesystem.md)

## Đọc thêm

- [C++ Concurrency in Action](https://www.amazon.com/dp/1617294691/ref=cm_sw_em_r_mt_dp_U_siEmDbRMMF960)
- [Tài liệu về luồng](https://en.cppreference.com/w/cpp/thread)
- Herlihy, M. P., & Wing, J. M. (1990). Linearizability: a correctness condition for concurrent objects. ACM Transactions on Programming Languages and Systems, 12(3), 463–492. https://doi.org/10.1145/78969.78972

## Giấy phép

<a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Giấy phép Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />Tác phẩm này được viết bởi [Ou Changkun](https://changkun.de) và được cấp phép theo <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/">Giấy phép Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International</a>. Mã nguồn của kho lưu trữ này được mở theo [giấy phép MIT](../../LICENSE).

