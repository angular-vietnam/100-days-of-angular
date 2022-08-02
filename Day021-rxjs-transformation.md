# Day 21: RxJS Transformation Operators

Trong Day 20 chúng ta đã biết về một số **Creation Operators**, chúng là những operators có thể call như call một function thông thường. Day 21 này, chúng ta sẽ bắt đầu đi vào tìm hiểu **Pipeable Operators**, thay vì được call độc lập thì nó sẽ được call ở trong `pipe()` method của một Observable instance.

## Pipeable Operators

Một Pipeable Operator là một function nó nhận đầu vào là một Observable và returns một Observable khác. Chúng là pure operation: Observable truyền vào sẽ không bị thay đổi gì.

Cú pháp:

```ts
observableInstance.pipe(operator1(), operator2());
```

Với cú pháp trên thì `observableInstance` có _pipe_ bao nhiêu operator đi nữa thì nó vẫn không đổi, và cuối cùng chúng ta sẽ nhận lại một Observable nên để có thể sử dụng thì chúng ta cần gán lại, hoặc thực hiện subscribe ngay sau khi _pipe_:

```ts
const returnObservable = observableInstance.pipe(operator1(), operator2());
```

Nếu bạn dùng với RxJS version < 5.5 thì có thể các bạn sẽ thấy cú pháp sử dụng khác là prototype method chain, nhưng nếu bạn dùng từ version 5.5 trở lên thì nên dùng pipe operators, dựa theo một số giải thích ở đây: [pipeable operators](https://rxjs.dev/guide/v6/pipeable-operators)

Pipeable Operators có thể chia thành nhiều category khác nhau, trong ngày hôm nay chúng ta sẽ tìm hiểu về **Transformation Operators**.

## Transformation Operators

Chắc hẳn các bạn đã quá quen với làm việc cùng Array trong JS, chúng ta có thể lặp qua từng phần tử trong mảng, sau đó apply một function lên mỗi phần tử, kết quả trả về sẽ được đưa vào một mảng mới có kích thước giống như mảng ban đầu như sau:

```ts
const users = [
  {
    id: 'ddfe3653-1569-4f2f-b57f-bf9bae542662',
    username: 'tiepphan',
    firstname: 'tiep',
    lastname: 'phan',
  },
  {
    id: '34784716-019b-4868-86cd-02287e49c2d3',
    username: 'nartc',
    firstname: 'chau',
    lastname: 'tran',
  },
];

const usersVm = users.map((user) => {
  return {
    ...user,
    fullname: `${user.firstname} ${user.lastname}`,
  };
});
```

Kết quả có được sẽ có dạng như sau:

```ts
usersVm = [
  {
    id: 'ddfe3653-1569-4f2f-b57f-bf9bae542662',
    username: 'tiepphan',
    firstname: 'tiep',
    lastname: 'phan',
    fullname: 'tiep phan',
  },
  {
    id: '34784716-019b-4868-86cd-02287e49c2d3',
    username: 'nartc',
    firstname: 'chau',
    lastname: 'tran',
    fullname: 'chau tran',
  },
];
```

Như vậy qua một lần biến đổi, chúng ta sẽ có được dữ liệu như ý muốn.

Vậy với Observable thì sao. Giả sử chúng ta đang có một hệ thống tracking xem những ai đăng nhập vào hệ thống. Do đó ở một số thời điểm sẽ có một/một vài người đăng nhập, và mỗi lần như thế hệ thống sẽ gửi cho chúng ta một event để biết. Bây giờ chúng ta cũng làm nhiệm vụ tương tự như `map` ở trên thì sao.

```ts
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

interface User {
  id: string;
  username: string;
  firstname: string;
  lastname: string;
}

const source = new Observable<User>((observer) => {
  const users = [
    {
      id: 'ddfe3653-1569-4f2f-b57f-bf9bae542662',
      username: 'tiepphan',
      firstname: 'tiep',
      lastname: 'phan',
    },
    {
      id: '34784716-019b-4868-86cd-02287e49c2d3',
      username: 'nartc',
      firstname: 'chau',
      lastname: 'tran',
    },
  ];

  setTimeout(() => {
    observer.next(users[0]);
  }, 1000);
  setTimeout(() => {
    observer.next(users[1]);
    observer.complete();
  }, 3000);
});

const observer = {
  next: (value) => console.log(value),
  error: (err) => console.error(err),
  complete: () => console.log('completed'),
};
source.subscribe(observer);
```

Khi chạy chương trình bạn sẽ thấy rằng, sau 1 giây thì sẽ emit ra user đầu tiên, và sau đó 2 giây thì sẽ emit ra user thứ hai kèm theo complete signal.

### map

`map<T, R>(project: (value: T, index: number) => R, thisArg?: any): OperatorFunction<T, R>`

Giả sử bạn cần hiển thị thông tin `fullname` của user trong `next` thì bạn sẽ có thể dùng cách nào.

Cách đơn giản nhất là bạn sẽ vào hàm next để thực hiện tính toán. Nhưng chúng ta có thể **transform** stream data trước khi nó đi đến với điểm cuối.

Đây chính là lúc bạn có thể sử dụng đến Operator như map của RxJS.

```ts
import { map } from 'rxjs/operators';

source
  .pipe(
    map((user) => {
      return {
        ...user,
        fullname: `${user.firstname} ${user.lastname}`,
      };
    })
  )
  .subscribe(observer);
```

Hoặc giả sử yêu cầu của chúng ta giờ đây thay đổi, chỉ cần trả về id của user mỗi khi được emit.

```ts
source.pipe(map((user) => user.id)).subscribe(observer);
```

Cách dùng map này _khá giống_ cách dùng map của array ở trên phải không???

![RxJS map](assets/rxjs-map.png)

### pluck

`pluck<T, R>(...properties: string[]): OperatorFunction<T, R>`

Đối với yêu cầu map ra một property trong một object như vừa rồi, bạn có thể sử dụng một cách khác là `pluck`:

```ts
import { pluck } from 'rxjs/operators';

source.pipe(pluck('id')).subscribe(observer);
```

![RxJS pluck](assets/rxjs-pluck.png)

### mapTo

`mapTo<T, R>(value: R): OperatorFunction<T, R>`

Sẽ thế nào nếu bạn muốn bất cứ khi nào stream emit một giá trị thì bạn luôn trả về một giá trị fixed không?

Giả sử bạn đang làm chức năng để lắng nghe mouse hover. Như bạn cũng có thể biết chúng ta sẽ cần kết hợp giữa `mouseover` và `mouseleave` event chẳng hạn.

Khi `mouseover` chúng ta luôn trả về `true`, và khi `mouseleave` chúng ta luôn trả về `false`.

Trong đoạn code dưới đây các bạn tạm thời hiểu rằng merge sẽ gộp 2 streams lại thành một, chúng ta sẽ học về combine streams những ngày sau.

```ts
const element = document.querySelector('#hover');

const mouseover$ = fromEvent(element, 'mouseover');
const mouseleave$ = fromEvent(element, 'mouseleave');

const hover$ = merge(
  mouseover$.pipe(mapTo(true)),
  mouseleave$.pipe(mapTo(false))
);

hover$.subscribe(observer);
```

Giờ đây chúng ta đã có một stream `hover$` để biết được khi nào chúng ta in/out ở một element.

![RxJS mapTo](assets/rxjs-mapTo.png)

### scan

`scan<T, R>(accumulator: (acc: R, value: T, index: number) => R, seed?: T | R): OperatorFunction<T, R>`

Bây giờ mỗi lần stream emit một value, bạn muốn apply một function lên value đó nhưng có sử dụng kèm theo kết quả lưu trữ trước đó (accumulator). Các bạn có thể liên tưởng ngay đến hàm `reduce` của Array.

Ví dụ: Count số lần người dùng đã click vào một button (giống như bài đầu tiên về RxJS).

```ts
const button = document.querySelector('#add');

const click$ = fromEvent(button, 'click');

click$.pipe(scan((acc, curr) => acc + 1, 0)).subscribe(observer);
```

Count số bài đăng của những người dùng đăng nhập theo thời gian:

```ts
const users$ = new Observable<User>((observer) => {
  const users = [
    {
      id: 'ddfe3653-1569-4f2f-b57f-bf9bae542662',
      username: 'tiepphan',
      firstname: 'tiep',
      lastname: 'phan',
      postCount: 5,
    },
    {
      id: '34784716-019b-4868-86cd-02287e49c2d3',
      username: 'nartc',
      firstname: 'chau',
      lastname: 'tran',
      postCount: 22,
    },
  ];

  setTimeout(() => {
    observer.next(users[0]);
  }, 1000);
  setTimeout(() => {
    observer.next(users[1]);
    observer.complete();
  }, 3000);
});

users$.pipe(scan((acc, curr) => acc + curr.postCount, 0)).subscribe(observer);
```

![RxJS scan](assets/rxjs-scan.png)

### reduce

`reduce<T, R>(accumulator: (acc: T | R, value: T, index?: number) => T | R, seed?: T | R): OperatorFunction<T, T | R>`

Operator này khá giống `scan` là nó sẽ reduce value overtime, nhưng nó sẽ đợi đến khi source complete rồi thì nó mới emit một giá trị cuối cùng và gửi đi `complete`.

```ts
users$.pipe(reduce((acc, curr) => acc + curr.postCount, 0)).subscribe(observer);
```

![RxJS reduce](assets/rxjs-reduce.png)

### toArray

`toArray<T>(): OperatorFunction<T, T[]>`

Giả sử bạn cần collect toàn bộ các value emit bởi stream rồi lưu trữ thành một array, sau đó đợi đến khi stream complete thì emit một array và complete. Lúc này bạn hoàn toàn có thể sử dụng `reduce`:

```ts
users$.pipe(reduce((acc, curr) => [...acc, curr], [])).subscribe(observer);
```

Nhưng có một cách viết khác ngắn gọn hơn đó là dùng `toArray`.

```ts
users$.pipe(toArray()).subscribe(observer);
```

### buffer

`buffer<T>(closingNotifier: Observable<any>): OperatorFunction<T, T[]>`

Lưu trữ giá trị được emit ra và đợi đến khi closingNotifier emit thì emit những giá trị đó thành 1 array.

```ts
const interval$ = interval(1000);

const click$ = fromEvent(document, 'click');

const buffer$ = interval$.pipe(buffer(click$));

const subscribe = buffer$.subscribe((val) =>
  console.log('Buffered Values: ', val)
);

// output có dạng
'Buffered Values: '[(0, 1)];
'Buffered Values: '[(2, 3, 4, 5, 6)];
```

![RxJS buffer](assets/rxjs-buffer.png)

### bufferTime

`bufferTime<T>(bufferTimeSpan: number): OperatorFunction<T, T[]>`

Tương tự như `buffer`, nhưng emit values mỗi khoảng thời gian `bufferTimeSpan` ms.

```ts
const source = interval(500);

const bufferTime = source.pipe(
  bufferTime(2000)
);

const bufferTimeSub = bufferTime.subscribe(
  val => console.log('Buffered with Time:', val)
);
// output
"Buffered with Time:"
[0, 1]
"Buffered with Time:"
[2, 3]
"Buffered with Time:"
[4, 5]
...
```

![RxJS bufferTime](assets/rxjs-bufferTime.png)

## Summary

Như vậy trong Day 21 chúng ta đã tìm hiểu cơ bản về một số **Transformation Operators** hay dùng trong RxJS, các bạn có thể thực hành thêm thông qua các ví dụ từ trang [rxjs.dev](https://rxjs.dev) để hiểu thêm.

## References

- [RxJS Overview](https://rxjs.dev/guide/overview)
- [LearnRxJS](https://www.learnrxjs.io/)

Mục tiêu của Day 22 là **RxJS Filtering Operators**.

## Youtube Video

[![Day 21](https://img.youtube.com/vi/AG97A7_NCLE/0.jpg)](https://youtu.be/AG97A7_NCLE)

## Author

[Tiep Phan](https://github.com/tieppt)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day21`
