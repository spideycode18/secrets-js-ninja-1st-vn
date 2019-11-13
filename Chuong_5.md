# Chương 5: Từng bước tiếp cận closure
**Những vấn đề được đề cập trong chương này**
- *Closure* là gì và nó hoạt động như thế nào
- Sử dụng *closure* để đơn giản hóa mã được phát triển
- Cải thiện hiệu năng bằng *closure*
- Giải quyết các vấn đề thường gặp về *scope* với *closure*.
## 5.1. Closure hoạt động như thế nào
Nói một cách ngắn gọn, *closure* là *scope* được tạo ra khi một hàm được khai báo cho phép hàm có thể truy cập và thao tác với các biến bên ngoài để hàm đó. Nói cách khác, *closure* cho phép một hàm truy cập tất cả các biến, cũng như các hàm khác, nằm trong *scope* khi chính hàm đó được khai báo.

Điều đó có vẻ khá trực quan cho đến khi bạn nhớ rằng một hàm được khai báo có thể được gọi bất cứ lúc nào sau đó, ngay cả sau khi *scope* mà nó được khai báo biến mất.
      
Khái niệm này có lẽ được giải thích tốt nhất thông qua mã, vì vậy chúng ta hãy bắt đầu một bước nhỏ với đoạn mã sau:

**Mã 5.1: closure đơn giản:**  
<pre>
<script type="text/javascript">
  var outerValue = 'ninja';     // <b>(1)</b> định nghĩa một giá trị trong *global scope*
  
  function outerFunction() {
    assert(outerValue == "ninja","I can see the ninja.");     // <b>(2)</b> khai báo một hàm trong *global scope*
  }
  
  outerFunction();     // <b>(3)</b> thực thi hàm
</script>
</pre>
Trong đoạn mã ví dụ, chúng ta khai báo một biến **(1)** và một hàm **(2)** trong cùng một *scope* - trong trường họp này là *global scope*. Sau đó chúng ta thực thi hàm **(3)**.

Có thể thấy ở hình 5.1, hàm có thể truy cập vào biến `outerValue`. Bạn có thể đã viết mã như thế hàng trăm lần mà không nhận ra rằng bạn vừa tạo một *closure*.

Không ấn tượng ư? Tôi đoán không ngạc nhiên lắm. Bởi vì cả `outerValue` và `outerFunction` đều được khai báo ở *global scope*, *scope* (mà thực tế là *closure*) không bao giờ biến mất (miến sao trang web được tải xong). Không ngạc nhiên khi hàm có thể truy câp vào biến bởi gì nó vẫn còn trong *scope* và khả thi. Kể cả *closure* tồn tại, lợi ích của nó vẫn không rõ ràng.

Hãy làm nó thú vị hơn một chút với đoạn mã tiếp theo:

**Mã 5.2: một ví dụ không quá đơn giản của `closure`**
```
<script type="text/javascript">
  var outerValue = 'ninja';
  var later;      
  function outerFunction() {
    var innerValue = 'samurai';
      function innerFunction() {
        assert(outerValue,"I can see the ninja.");
        assert(innerValue,"I can see the samurai.");
      }
    later = innerFunction;
  }
  outerFunction();
  later();
</script>
```
Hãy phân tích kỹ hơn đoạn mã `innerFunction()` và xem chúng ta có thể dự đoán điều gì xảy ra không.

`assert` đầu tiên chắc chắn thành công: `outerValue` thuộc *global scope* và nó rõ ràng với mọi thứ. Nhưng `assert` thứ hai thì thế nào?

Chúng ta thực thi `innderFunction` sau khi `outerFunction` được thực thi thông qua thủ thuật sao chép một tham chiếu đến hàm sang một tham chiếu cục bộ `later`. Khi `innderFunction` được thực thi, *scope* bên trong `outerFunction` biến mất và không còn thấy được tại thời điểm chúng ta gọi hàm thông qua `later`.

Vì thế chúng ta đã kì vọng `assert` sẽ thất bại, giống như `innderValue` chắc chắn sẽ là `undefined`. Đúng như thế không?

Nhưng khi chúng ta chạy kiểm thử, chúng ta thấy được hình 5.2.

Làm sao có thể như thế được? Phép màu nào cho phép biến `innerValue` còn tồn tại khi chúng ta thực thi `innerFunction`, sau khi *scope* nơi mà nó được tạo đã biến mất. Câu trả lời, tất nhiên, là `closure`.

Khi chúng ta khai báo `innerFunction()` bên trong `outerFunction` thì không chỉ hàm được khai báo mà một *closure* cũng được tạo, nó bao gồm không chỉ bao gồm khai báo hàm mà còn tất cả các biến trong `scope` tại thời điểm khai báo.

Khi `innerFunction()` thực thi, kể cả khi nó được thực thi sau khi `scope` nơi nó được khai báo biến mất, nó vẫn có thể truy cập vào `scope` gốc, nơi nó được khai báo thông qua `closure`, như hình 5.3.

Đó là tất cả về `closure`. Chúng tạo ra một 'bong bóng an toàn' cho hàm và biến trong `scope` tại thời điểm khai báo của hàm, do đó hàm sẽ có tất cả những thứ cần thiết đểm thực thi.

'Bong bóng' này, bao gồm hàm và biến của nó, luôn tồn tại với hàm đó.

Hãy phát triển ví dụ đó với một vài bổ sung để quan sát thêm một vài nguyên tắc cốt lõi của `closure`. Hãy xem đoạn mã sau, trong đó các bổ sung sẽ được tô đậm

**Mã 5.3: Những `closure` khác có thể thấy**
<pre>
<script type="text/javascript">
  var outerValue = 'ninja';
  var later;
  function outerFunction() {
    var innerValue = 'samurai';
    function innerFunction(<b>paramValue</b>) {     // <b>(1)</b> Thêm tham số vào hàm
      assert(outerValue,"Inner can see the ninja.");
      assert(innerValue,"Inner can see the samurai.");
      <b>assert(paramValue,"Inner can see the wakizashi.");</b>     // <b>(2)</b> Kiểm thử nếu chúng ta có thể thấy tham số
      <b>assert(tooLate,"Inner can see the ronin.");</b>     // Kiểm thử nếu <i>closure</i> bao gồm cả biến được khai báo sau                                                                        // khi hàm được hai báo.
    }
    later = innerFunction;
  }
  assert(!tooLate,"Outer can't see the ronin.");     // <b>(3)</b> Tìm kiếm giá trị trước khi được khai báo trong cùng <i>scope</i>
  <b>var tooLate = 'ronin';</b>     // <b>(4)</b> Khai báo một biến sau khi khai báo <code>innerFunction</code>
  outerFunction();
  later('<b>wakizashi</b>');     // <b>(5)</b> Gọi <code>innerFunction</code> để chạy các kiểm thử bên trong nó.
</script>
</pre>
