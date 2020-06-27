# Day 20: RxJS Creation Operators

Ở ngày 19, chúng ta đã tìm hiểu qua về `Observable` và chúng ta đã biết cách tạo 1 `Observable` bằng tay như sau:

```typescript
const observable = new Observable(function subscribe(observer) {
  const id = setTimeout(() => {
    observer.next('Hello Rxjs');
    observer.complete();
  }, 1000);
  return function unsubscribe() {
    clearTimeout(id);
  };
});
```

Chắc sẽ có một số bạn tự hỏi "Vậy chẳng lẽ lúc nào cũng phải nhớ cú pháp này để tạo `Observable` sao? Nào là `next`, rồi `complete`, rồi cả `unsubscribe`". Để trả lời câu hỏi này thì ở ngày hôm nay, chúng ta sẽ tìm về các `Operators` dùng để khởi tạo `Observable` mà **RxJS** cung cấp nhé.

> - Operators là các pure functions cho phép lập trình functional với Observable. (day 19)

## Intro

Đây sẽ là `observer` để dùng chung ở trong bài viết này. Nếu như ví dụ nào có `observer` riêng thì mình sẽ viết riêng nhé.

```typescript
const observer = {
  next: (val) => console.log(val),
  error: (err) => console.log(err),
  complete: () => console.log('complete'),
};
```

## Common Creation Operators

#### `of()`

`of()` là operator dùng để tạo 1 `Observable` từ bất cứ giá trị gì: primitives, Array, Object, Function v.v... `of()` sẽ nhận vào các giá trị và sẽ `complete` ngay sau khi tất cả các giá trị truyền vào được `emit`.

1. Primitive value

```typescript
// output: 'hello'
// complete: 'complete'
of('hello').subscribe(observer);
```

2. Object/Array

```typescript
// output: [1, 2, 3]
// complete: 'complete'
of([1, 2, 3]).subscribe(observer);
```

3. Dãy giá trị (sequence of values)

```typescript
// output: 1, 2, 3, 'hello', 'world', {foo: 'bar'}, [4, 5, 6]
// complete: 'complete'
of(1, 2, 3, 'hello', 'world', { foo: 'bar' }, [4, 5, 6]).subscribe(observer);
```

#### `from()`

`from()` cũng gần giống với `of()`, cũng được sử dụng để tạo `Observable` từ 1 giá trị. Tuy nhiên, điểm khác biệt đối với `of()` là `from()` chỉ nhận
vào giá trị là một `Iterable` hoặc là một `Promise`.

> Iterable là những giá trị có thể iterate qua được, ví dụ: array, map, set, hoặc string. Khi bạn loop qua 1 string, thì các bạn sẽ nhận đc 1 dãy các ký tự trong string.

1. Array

```typescript
// output: 1, 2, 3
// complete: 'complete'
from([1, 2, 3]).subscribe(observer);
```

Khi nhận vào 1 `Array`, `from()` sẽ emit các giá trị trong array theo sequence. Điều này tương đương với `of(1, 2, 3)`.

2. String

```typescript
// output: 'h', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd'
// complete: 'complete'
from('hello world').subscribe(observer);
```

3. Map/Set

```typescript
const map = new Map();
map.set(1, 'hello');
map.set(2, 'bye');

// output: [1, 'hello'], [2, 'bye']
// complete: 'complete'
from(map).subscribe(observer);

const set = new Set();
set.add(1);
set.add(2);

// output: 1, 2
// complete: 'complete'
from(set).subscribe(observer);
```

4. Promise

```typescript
// output: 'hello world'
// complete: 'complete'
from(Promise.resolve('hello world')).subscribe(observer);
```

Các bạn có thể thấy là `from()` sẽ unwrap và return được giá trị `resolved` của `Promise`. Đây là cách chuyển đổi `Promise` thành `Observable`.

#### `fromEvent()`

`fromEvent()` được dùng để chuyển đổi 1 sự kiện (Event) sang `Observable`. Ví dụ chúng ta có `DOM Event` như mouse click hoặc input.

```typescript
const btn = document.querySelector('#btn');
const input = document.querySelector('#input');

// output (example): MouseEvent {...}
// complete: không có gì log.
fromEvent(btn, 'click').subscribe(observer);

// output (example): KeyboardEvent {...}
// complete: không có gì log.
fromEvent(input, 'keydown').subscribe(observer);
```

Các bạn chú ý là `fromEvent()` sẽ tạo 1 `Observable` không tự `complete`. Việc này hoàn toàn hợp lý vì chúng ta muốn lắng nghe các sự kiện như `click` và `keydown` cho đến
khi nào chính chúng ta không cần lắng nghe nữa thì thôi. `fromEvent()` không thể quyết định được lúc nào chúng ta không cần những sự kiện này nữa. Điều này cũng đồng nghĩa
với việc các bạn phải chủ động `unsubscribe` các `Observable` tạo từ `fromEvent()` này nếu không muốn bị **tràn bộ nhớ** (memory leak).

#### `fromEventPattern()`

`fromEventPattern()` là 1 dạng _nâng cao_ của `fromEvent()`. Nói về concept thì `fromEventPattern()` cũng giống với `fromEvent()` là tạo `Observable` từ sự kiện. Tuy nhiên, `fromEventPattern()` rất khác với `fromEvent()` về cách dùng, cũng như loại sự kiện để xử lý. Để hiểu rõ hơn, chúng ta cùng tham khảo ví dụ sau:

```typescript
// fromEvent() từ ví dụ trên
// output: MouseEvent {...}
fromEvent(btn, 'click').subscribe(observer);

// fromEventPattern
// output: MouseEvent {...}
fromEventPattern(
  (handler) => {
    btn.addEventListener('click', handler);
  },
  (handler) => {
    btn.removeEventListener('click', handler);
  }
).subscribe(observer);
```

Một ví dụ khác nếu chúng ta cần biết được ví trị con trỏ trên element được click

```typescript
// output: 10 10
fromEvent(btn, 'click')
  .pipe(map((ev: MouseEvent) => ev.offsetX + ' ' + ev.offsetY))
  .subscribe(observer);

// fromEventPattern
// Ở ví dụ này, chúng ta sẽ tách `addHandler` và `removeHandler` ra thành function riêng nhé

function addHandler(handler) {
  btn.addEventListener('click', handler);
}

function removeHandler(handler) {
  btn.removeEventListener('click', handler);
}

// output: 10 10
fromEventPattern(
  addHandler,
  removeHandler,
  (ev: MouseEvent) => ev.offsetX + ' ' + ev.offsetY
).subscribe(observer);
```

Như các ví dụ trên, `fromEventPattern()` nhận vào 3 giá trị: `addHandler`, `removeHandler`, và `projectFunction` với `projectFunction` là `optional`. `fromEventPattern()` không khác
gì `fromEvent()` nhưng `fromEventPattern()` cung cấp cho các bạn 1 API để có thể chuyển đổi các sự kiện từ API gốc của sự kiện. Ví dụ như trên thì chúng ta sử dụng trực tiếp API như `addEventListener` và `removeEventListener` từ DOM để chuyển đổi sang `Observable` chứ không giống như `fromEvent()`. Với kiến thức này, các bạn hoàn toàn có thể dùng `fromEventPattern()`
để chuyển đổi các sự kiện có API phức tạp hơn thành `Observable`, ví dụ như **SignalR Hub**

```typescript
// _getHub() là hàm trả về Hub.
const hub = this._getHub(url);
return fromEventPattern(
  (handler) => {
    // mở websocket
    hub.connection.on(methodName, handler);

    if (hub.refCount === 0) {
      hub.connection.start();
    }

    hub.refCount++;
  },
  (handler) => {
    hub.refCount--;
    // đóng websocket khi unsubscribe
    hub.connection.off(methodName, handler);
    if (hub.refCount === 0) {
      hub.connection.stop();
      delete this._hubs[url];
    }
  }
);
```

#### `interval()`

`interval()` là hàm để tạo `Observable` mà sẽ emit giá trị số nguyên từ số 0 theo 1 chu kỳ nhất định. Hàm này giống với `setInterval`.

```typescript
// output: 0, 1, 2, 3, 4, ...
interval(1000) // emit giá trị sau mỗi giây
  .subscribe(observer);
```

Giống như `fromEvent()`, `interval()` không tự động `complete` cho nên các bạn phải xử lý việc `unsubscribe`.

#### `timer()`

`timer()` có 2 cách sử dụng:

- Tạo `Observable` mà sẽ emit giá trị sau khi delay 1 khoảng thời gian nhất định. Cách dùng này sẽ tự `complete` nhé.
- Tạo `Observable` mà sẽ emit giá trị sau khi delay 1 khoảng thời gian và sẽ emit giá trị sau mỗi chu kỳ sau đó. Cách dùng này tương tự với `interval()` nhưng `timer()` hỗ trợ delay trước khi emit. Vì cách dùng này giống với `interval()` nên sẽ không tự `complete`.

```typescript
// output: sau 1 giây -> 0
// complete: 'complete'
timer(1000).subscribe(observer);

// output: sau 1 giây -> 0, 1, 2, 3, 4, 5 ...
timer(1000, 1000).subscribe(observer);
```

#### `throwError()`

`throwError()` sẽ dùng để tạo `Observable` mà thay vì emit giá trị, `Observable` này sẽ throw 1 error ngay lập tức sau khi `subscribe`.

```typescript
// error: 'an error'
throwError('an error').subscribe(observer);
```

`throwError()` thường dùng trong việc xử lý lỗi của 1 `Observable`, sau khi xử lý lỗi, chúng ta muốn throw tiếp error cho `ErrorHandler` tiếp theo, chúng ta sẽ dùng `throwError`. Khi làm
việc với `Observable`, có 1 số `operators` yêu cầu các bạn phải cung cấp 1 `Observable` (ví dụ như `switchMap`, `catchError`) thì việc `throwError` trả về 1 `Observable` là rất thích hợp.

#### `defer()`

Cuối cùng, chúng ta sẽ tìm hiểu 1 operator rất hay ho đó là `defer()`. `defer()` nhận vào 1 `ObservableFactory` và sẽ trả về `Observable` này. Điểm đặc biệt của `defer()` là ở việc `defer()` sẽ dùng `ObservableFactory` này để tạo 1 `Observable` mới cho mỗi `Subscriber`. Chúng ta cùng tham khảo ví dụ sau:

```typescript
// of()
const now$ = of(Math.random());
// output: 0.4146530439875191
now$.subscribe(observer);
// output: 0.4146530439875191
now$.subscribe(observer);
// output: 0.4146530439875191
now$.subscribe(observer);
```

Các bạn có thể thấy `of()` sẽ trả về cùng giá trị `Math.random()` cho cả 3. Bây giờ chúng ta thử đổi sang `defer()` nhé.

```typescript
const now$ = defer(() => of(Math.random()));
// output: 0.27312186273281935
now$.subscribe(observer);
// output: 0.7180321390218474
now$.subscribe(observer);
// output: 0.9626312890837065
now$.subscribe(observer);
```

Với `defer()`, chúng ta đã có 3 giá trị khác nhau cho mỗi lần subscribe. Điều này giúp ích ở điểm nào? Ví dụ trường hợp chúng ta cần `retry` 1 `Observable` nào đó mà cần so sánh với 1 giá trị random để quyết định xem có chạy tiếp hay không, thì `defer()` (kết hợp với `retry`) là 1 giải pháp cực kỳ hiệu quả.

## Summary

Ở ngày 20 này, chúng ta đã tìm hiểu qua kha khá các `operators` dùng để tạo `Observable`, với tên gọi chính thức là `Creation Operators`. Đây là những operators khá phổ biến, tuy nhiên, các bạn chỉ cần nắm kĩ: `from()`, `of()`, `interval()`, `timer()`, và `defer()` là được. `fromEvent()` và `fromEventPattern()` rất ít khi sử dụng trong ứng dụng Angular. Ngoài những `operators` mình liệt kê trên, **RxJS** còn cung cấp 1 số `Creation Operators` khác như: `ajax()`, `fromFetch()`, `generate()`. Và cũng như lý do trên, trong `Angular`, chúng ta rất ít khi sử dụng những operators này. Ví dụ thay vì `ajax()` và `fromFetch()` chúng ta đã có `HttpClientModule`.

## References

- [RxJS Overview](https://rxjs.dev/guide/overview)
- [LearnRxJS](https://www.learnrxjs.io/)

Mục tiêu của Day 21 là **RxJS Transformation Operators**.

## Author

[Chau Tran](https://github.com/nartc)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day20`
