# Day 23: RxJS Combination Operators

Tiếp tục cuộc hành trình tìm hiểu về các `operators` của **RxJS** nhé. Lần này, chúng ta sẽ tìm hiểu về 1 loại `operators` rất quan trọng khi làm việc với **Angular** vì những `operators` này sẽ cho phép các bạn kết hợp nhiều `Observable` lại với nhau. Những `operators` này gọi là **Combination Operators**.

```ts
const observer = {
  next: (val) => console.log(val),
  error: (err) => console.log(err),
  complete: () => console.log('complete'),
};
```

### forkJoin()

`forkJoin(...sources: any[]): Observable<any>`

Nếu bạn nào đã dùng quen `Promise` qua rồi thì `forkJoin()` sẽ là 1 `operator` cực kỳ quen thuộc vì xét về công dụng thì `forkJoin()` tương đương với `Promise.all()`.

`forkJoin()` nhận vào tham số là 1 list các `Observables` theo dạng `Array` hoặc `Dictionary (Object)` (children). Khi các children `Observables` **complete** hết thì `forkJoin()` sẽ emit giá trị của các children `Observables` theo dạng `Array` hoặc `Dictionary` (tuỳ vào tham số truyền vào) rồi sau đó sẽ **complete**.

![RxJS forkJoin](assets/rxjs-forkJoin.png)

```ts
forkJoin([of(1), of('hello'), of({ foo: 'bar' })]).subscribe(observer);
// output: [1, 'hello', {foo: 'bar'}]
// output: 'complete'

forkJoin({ one: of(1), hello: of('hello'), foo: of({ foo: 'bar' }) }).subscribe(
  observer
);
/**
 * output:
 * {
 *   one: 1,
 *   hello: 'hello',
 *   foo: { foo: 'bar' }
 * }
 * output: 'complete'
 */
```

#### Lưu ý:

- `forkJoin()` chỉ emit khi các children `Observables` complete. Nếu như 1 trong số các children `Observables` không complete, `forkJoin()` sẽ không bao giờ emit.
- `forkJoin()` sẽ throw error khi 1 trong các children `Observables` throw error, và giá trị của các children `Observables` đã complete khác sẽ bị _nuốt_ mất nếu như các bạn không xử lý error hợp lý.

#### Use-case:

`forkJoin()` sử dụng rất nhiều trong ứng dụng **Angular**, đặc biệt là khi bạn cần request cùng lúc một loạt các `Dropdown/Select`.

```ts
forkJoin([
  this.apiService.getAccountDropdown(),
  this.apiService.getDepartmentDropdown(),
  this.apiService.getStoreDropdown(),
]).subscribe(observer);
// output: [accountList, departmentList, storeList]
// output: 'complete'
```

`forkJoin()` khi dùng với children `Observables` là 1 `Array`, thì `forkJoin()` có thể nhận vào 1 tham số thứ 2 gọi là `projectFunction`. `projectFunction` này sẽ được gọi với các tham số là giá trị của children `Observables` và kết quả của `projectFunction` sẽ là kết quả emit của `forkJoin()`. `projectFunction` chỉ được thực thi nếu như `forkJoin()` **SẼ** emit (nghĩa là tất cả children `Observables` đều complete)

```ts
forkJoin(
  [
    this.apiService.getAccountDropdown(),
    this.apiService.getDepartmentDropdown(),
    this.apiService.getStoreDropdown(),
  ],
  (accountList, departmentList, storeList) => {
    return {
      accounts: accountList,
      departments: departmentList,
      stores: storeList,
    };
  }
).subscribe(observer);
// output: { accounts: [...], departments: [...], stores: [...] }
// output: 'complete'
```

### combineLatest()

`combineLatest<O extends ObservableInput<any>, R>(...observables: (SchedulerLike | O | ((...values: ObservedValueOf<O>[]) => R))[]): Observable<R>`

`combineLatest()` giống với `forkJoin()` là cũng sẽ nhận vào tham số là 1 `Array<Observable>`. Khác với `forkJoin()` là `combineLatest()` không nhận vào `Dictionary (Object)` và `combineLatest()` sẽ emit khi **TẤT CẢ** các children `Observables` emit ít nhất một lần, nghĩa là các children `Observables` không cần phải complete mà chỉ cần emit ít nhất 1 giá trị thì `combineLatest()` sẽ emit giá trị là `Array` gồm tất cả các giá trị được children `Observables` emit, theo thứ tự.

> Thay vì truyền vào `Array<Observable>` cho `combineLatest()` như sau: `combineLatest([obs1, obs2])`, bạn cũng có thể truyền vào mà ko cần `[]` như: `combineLatest(obs1, obs2)`. Cả 2 cách đều cho ra kết quả như nhau, tuy nhiên, **RxJS** khuyên dùng cách 1 hơn vì nó nhất quán với `forkJoin()` hơn và cũng dễ dự đoán được kết quả hơn, vì kết quả của `combineLatest()` là 1 `Array`. Vì vậy, mình chỉ đề cập đến cách dùng `combineLatest([obs1, obs2])`

![RxJS combineLatest](assets/rxjs-combineLatest.png)

```ts
combineLatest([
  interval(2000).pipe(map((x) => `First: ${x}`)), // {1}
  interval(1000).pipe(map((x) => `Second: ${x}`)), // {2}
  interval(3000).pipe(map((x) => `Third: ${x}`)), // {3}
]).subscribe(observer);

// output:
// sau 3s, vì interval(3000) có khoảng thời gian dài nhất:
// [First 0, Second 2, Third 0] -- vì sao? vì tại 3s, thì {2} đã emit đc 3 lần rồi (3s, mỗi giây nhảy từ 0 -> 1 -> 2)

// sau 1s kế tiếp: (giây thứ 4)
// [First 1, Second 2, Third 0] -- vì sao? vì lúc này là giây thứ 4, {1} đã emit đc 2 lần (4s, mỗi 2 giây nhảy từ  0 -> 1)
// [First 1, Second 3, Third 0] -- vì sao? vì lúc này là giây thứ 4, {2} đã emit đc lần thứ 4 (0 -> 1 -> 2 -> 3)

// sau 1s kế tiếp: (giây thứ 5)
// [First 1, Second 4, Third 0] -- {2} emit lần thứ 5

// sau 1s kế tiếp: (giây thứ 6)
// [First 2, Second 4, Third 0] -- {1} emit lần thứ 3
// [First 2, Second 5, Third 0] -- {2} emit lần thứ 6
// [First 2, Second 5, Third 1] -- {3} emit lần thứ 2
```

#### Lưu ý:

- Qua ví dụ, các bạn cũng có thể thấy là `combineLatest()`, sau lần emit đầu tiên của các children `Observables`, thì sẽ emit giá trị mới nhất của child `Observable` đang emit + giá trị gần nhất của các children `Observables` đã emit.
- Cũng qua ví dụ, các bạn có thể thấy là `Observable` thứ 2 **{2}** (`interval(1000)`) bị _nuốt_ mất 2 giá trị đầu tiên là `0` và `1` vì **{2}** đã emit với tần suất nhanh hơn là `Observable` có khoảng thời gian lâu nhất **{3}**. Đây là điều các bạn cần lưu ý để có thể tránh hiện tượng **racing condition**.
- `combineLatest()` sẽ complete khi tất cả children `Observables` complete.
- `combineLatest()` sẽ không bao giờ complete nếu như 1 trong số các children `Observables` không bao giờ complete.
- `combineLatest()` sẽ throw error nếu như 1 trong số các children `Observables` throw error và giá trị của các children `Observables` đã emit khác sẽ bị _nuốt_ (behavior này giống với `forkJoin()`)

#### Use-case

Dùng rất nhiều trong việc combine state khi dùng `Service` trong **Angular**. Vì tính chất **long-lived** không complete sau 1 lần emit, `combineLatest()` là sự lựa chọn tốt cho việc combine các state trong `Service` và kết hợp với `AsyncPipe` để dùng trong template.

```ts
this.vm$ = combineLatest([
  this.paginationService.currentPage$,
  this.paginationService.currentSize$,
  this.paginationService.totalCount$,
  this.paginationService.currentOffset$,
]).pipe(
  map((currentPage, currentSize, totalCount, currentOffset) => {
    return {
      currentPage,
      currentSize,
      totalCount,
      currentOffset,
    };
  })
);

onSizeChanged(newSize: number) {
  this.paginationService.updateSize(newSize);
}

onPageChanged(newPage: number) {
  this.paginationService.updatePage(newPage);
}
```

```html
<ng-container *ngIf="vm$ | async as vm">
  <app-show-total
    [offset]="vm.currentOffset"
    [total]="vm.totalCount"
    [size]="vm.currentSize"
  ></app-show-total>
  <!-- Display: 1 - 20 of 100 -->
  <app-paginator
    [current]="vm.currentPage"
    [total]="vm.totalCount"
    [size]="vm.currentSize"
    (sizeChanged)="onSizeChanged($event)"
    (pageChanged)="onPageChanged($event)"
  ></app-paginator>
</ng-container>
```

Như trên là 1 ví dụ khá hoàn chỉnh về việc xử lý `PaginationComponent` trong **Angular** sử dụng `combineLatest()` và `AsyncPipe`. Khi `updateSize()` và `updatePage()` được thực thi thì `currentPage$` và `currentSize$` sẽ emit giá trị mới, dẫn đến `combineLatest()` (`vm$`) sẽ emit giá trị mới và template sẽ được update (`vm$ | async`).

Cũng giống như `forkJoin()`, khi dùng tham số là 1 `Array<Observable>` thì `combineLatest()` có thẻ nhận vào thêm 1 tham số là `projectFunction`. `projectFunction` này sẽ được gọi với các tham số là giá trị của children `Observables` và kết quả của `projectFunction` sẽ là kết quả emit của `combineLatest()`. Đây cũng là lí do mình chỉ đề cập đến cách dùng `combineLatest([obs1, obs2])` vì tính nhất quán với `forkJoin()`, và khả năng dùng `projectFunction`. Sau đây là ví dụ về `vm$` ở trên viết lại với `projectFunction`

```ts
this.vm$ = combineLatest(
  [
    this.paginationService.currentPage$,
    this.paginationService.currentSize$,
    this.paginationService.totalCount$,
    this.paginationService.currentOffset$,
  ],
  (currentPage, currentSize, totalCount, currentOffset) => {
    return {
      currentPage,
      currentSize,
      totalCount,
      currentOffset,
    };
  }
);
```

### zip()

`zip<O extends ObservableInput<any>, R>(...observables: (O | ((...values: ObservedValueOf<O>[]) => R))[]): Observable<ObservedValueOf<O>[] | R>`

`zip()` nhận vào tham số là `...(Observables | Function)` để tượng trưng cho các child `Observable` được truyền vào lần lượt. `zip()` là 1 operator khá hay ho vì `zip()` sẽ gom tất cả các giá trị được emit bởi children `Observable` theo cặp.

Nghĩa là sao? Ví dụ: `obs1 emit 1 - 2 - 3`, `obs2 emit 2 - 3- 4`, `obs3 emit 3 - 4 - 5`. Nếu như bạn dùng `combineLatest()` thì giá trị mà các bạn nhận được là:

```ts
combineLatest(of(1, 2, 3), of(4, 5, 6), of(7, 8, 9)).subscribe(observer);
// [1, 4, 7], // cả 3 emit
// [2, 4, 7], // obs1 emit 2, combineLatest emit giá trị 2 của obs1 và 2 giá trị cũ của obs2 và obs3
// ...        // sau 1 loạt emit
// [3, 6, 9]
```

Nhưng khi dùng `zip` với ví dụ trên:

```ts
zip(of(1, 2, 3), of(4, 5, 6), of(7, 8, 9)).subscribe(observer);
// [1, 4, 7]; // 3 số đầu tiên ở từng observable
// [2, 5, 8]; // 3 số tiếp theo
// [3, 6, 9]; // 3 số cuối cùng
```

- `zip()` sẽ complete khi 1 trong các children `Observables` complete. Nghĩa là các bạn chỉ luôn luôn nhận được giá trị đã được gộp lại của `zip`

```ts
zip(of(1, 2, 3, 99), of(4, 5, 6), of(7, 8, 9)).subscribe(observer);
// [1, 4, 7]; // 3 số đầu tiên ở từng observable
// [2, 5, 8]; // 3 số tiếp theo
// [3, 6, 9]; // 3 số cuối cùng
// 99 của observable đầu tiên sẽ bị bỏ qua vì observable thứ 2 và thứ 3 đã complete mất rồi.
```

- `zip()` sẽ throw error nếu 1 trong các children `Observables` throw error.
- Nếu tham số cuối cùng của `zip()` là 1 `Function` thì `zip()` sẽ coi tham số này là `projectFunction`. Cách thức hoạt động hoàn toàn giống với `projectFunction` của `combineLatest()` và `forkJoin()`.

#### Use-case

`zip()` cực kỳ hữu hiệu nếu như các bạn rơi vào các trường hợp sau:

- Giá trị cuối cùng mà bạn cần được cung cấp bởi nhiều `Observables` khác nhau. Ví dụ:

```ts
const age$ = of<number>(29, 28, 30);
const name$ = of<string>('Chau', 'Trung', 'Tiep');
const isAdmin$ = of<boolean>(true, false, true);

zip(age$, name$, isAdmin$).pipe(
  map(([age, name, isAdmin]) => ({ age, name, isAdmin }))
);
// output:
// { age: 29, name: 'Chau', isAdmin: true }
// { age: 28, name: 'Trung', isAdmin: false }
// { age: 30, name: 'Tiep', isAdmin: true }

// dùng với projectFunction
zip(age$, name$, isAdmin$, (age, name, isAdmin) => ({
  age,
  name,
  isAdmin,
})).subscribe(observer);
// output:
// { age: 29, name: 'Chau', isAdmin: true }
// { age: 28, name: 'Trung', isAdmin: false }
// { age: 30, name: 'Tiep', isAdmin: true }
```

- Kết hợp giá trị của 2 `Observables` khác nhau ở 2 thời điểm khác nhau. Ví dụ: các bạn muốn biết toạ độ chuột của người dùng từ lúc họ `mousedown` cho đến lúc họ `mouseup`, hoặc có thể lấy khoảng thời gian họ rê chuột (dùng `new Date()` thay vì `getCoords()` như ví dụ bên dưới)

```ts
const log = (event, val) => `${event}: ${JSON.stringify(val)}`;
const getCoords = pipe(
  map((e: MouseEvent) => ({ x: e.clientX, y: e.clientY }))
);
const documentEvent = (eventName) =>
  fromEvent(document, eventName).pipe(getCoords);

zip(documentEvent('mousedown'), documentEvent('mouseup')).subscribe((e) =>
  console.log(`${log('start', e[0])} ${log('end', e[1])}`)
);
// output:
// start: {"x":291,"y":136} end: {"x":143,"y":168}
// start: {"x":33,"y":284} end: {"x":503,"y":74}
```

### concat()

`concat<O extends ObservableInput<any>, R>(...observables: (SchedulerLike | O)[]): Observable<ObservedValueOf<O> | R>`

`concat()` nhận vào tham số `...Observable` để tượng trưng cho các child `Observable` được truyền vào lần lượt thay vì truyền vào 1 `Array<Observable>`

`concat()` sẽ subscribe vào các children `Observables` theo thứ tự được truyền vào và sẽ emit khi `Observable` đang được subscribe complete (gọi là {1}).

- Nếu **{1}** emit và complete, `concat()` sẽ emit giá trị mà **{1}** emit rồi sẽ subscribe vào `Observable` kế tiếp.
- Nếu **{1}** error, `concat()` sẽ error ngay lặp tức và chuỗi `Observable` phía sau sẽ bị bỏ qua.
- Nếu **{1}** complete mà không emit, `concat()` sẽ bỏ qua và subscribe vào `Observable` kế tiếp
- Nết **{1}** emit và không complete, `concat()` sẽ emit giá trị mà **{1}** emit **NHƯNG** sẽ không subscribe vào `Observable` kế tiếp vì **{1}** không complete.

`concat()` sẽ hoạt động tương tự cho **LẦN LƯỢT** từng children `Observables` cho đến khi không còn `Observable` nào thì `concat()` sẽ complete.

![RxJS concat](assets/rxjs-concat.png)

```ts
concat(of(4, 5, 6).pipe(delay(1000)), of(1, 2, 3)).subscribe(observer);
// output:
// sau 1s:
// 4-5-6-1-2-3
// output: 'complete'
```

Các bạn có thể thấy là `concat()` sẽ chờ cho `of(4, 5, 6).pipe(delay(1000))` emit và complete thì `concat()` mới emit `4-5-6` rồi subscribe vào `of(1, 2, 3)`.

Các bạn cũng có thể truyền 1 `Observable` nhiều lần vào cho `concat()`.

```ts
const fiveSecondTimer = interval(1000).pipe(take(5));

concat(fiveSecondTimer, fiveSecondTimer, fiveSecondTimer).subscribe(observer);
// output: 0,1,2,3,4 - 0,1,2,3,4 - 0,1,2,3,4
// output: 'complete'

// dùng repeat()
concat(fiveSecondTimer.pipe(repeat(3))).subscribe(observer);
// output: 0,1,2,3,4 - 0,1,2,3,4 - 0,1,2,3,4
// output: 'complete'
```

### merge()

`merge<T, R>(...observables: any[]): Observable<R>`

`merge()` nhận vào tham số `...(Observable | number)` để tượng trưng cho các child `Observable` được truyền vào lần lượt thay vì truyền vào 1 `Array<Observable>`. Khác với `concat()`, `merge()` không quan tâm đến thứ tự mà các children `Observables` emit cho nên `merge()` không bị giới hạn bởi việc các `Observable` con phải complete thì `Observable` kế tiếp mới được subscribe. Tham số cuối cùng của `merge()` nếu là 1 `number`, thì `merge()` sẽ coi tham số đó là số `concurrent`. `concurrent` xác định số lượng children `Observables` mà `merge()` sẽ subscribe song song (concurrently). Default thì `merge()` sẽ subscribe vào **TẤT CẢ** children `Observables` song song.

`merge()` sẽ subscribe vào tất cả (phụ thuộc vào tham số `concurrent` nếu được truyền vào) các children `Observables` và sẽ:

- emit giá trị mà bất cứ `Observable` nào emit.
- throw error nếu 1 trong children `Observables` throw error
- complete khi và chỉ khi tất cả children `Observables` complete.

![RxJS merge](assets/rxjs-merge.png)

```ts
merge(of(4, 5, 6).pipe(delay(1000)), of(1, 2, 3)).subscribe(observer);
// output:
// 1,2,3
// sau 1s: 4,5,6
// output: 'complete'
```

Các bạn thấy sự khác biệt với `concat()` chưa? Ở đây, `merge()` emit luôn `1,2,3` rồi mới tới `4,5,6` và không hề quan tâm đến thứ tự mà các children `Observables` này được truyền vào. Mình thêm 1 ví dụ khác để các bạn thấy rõ hơn cách hoạt động của `merge()`

```ts
merge(
  interval(2000).pipe(mapTo('emit every 2 seconds'), take(3)),
  interval(1000).pipe(mapTo('emit every 1 second'), take(3))
).subscribe(observer);

// output:
// (sau 1s):
// emit every 1 second
// (sau 2s):
// emit every 2 seconds
// emit every 1 second
// (sau 3s):
// emit every 1 second - cái này complete, vì đã emit 3 lần rồi (take(3))
// (sau 4s):
// emit every 2 seconds
// (sau 6s):
// emit every 2 seconds - cái này complete (take(3))
// output: 'complete'
```

Ví dụ trên sẽ cho các bạn thấy `merge()` sẽ emit khi mà child `Observable` emit thôi và có thể emit xen kẽ nhau. Sau đây sẽ là ví dụ truyền vào tham số `concurrent`

```ts
merge(
  interval(1000).pipe(mapTo('first'), take(5)), // will take 5 seconds to complete
  interval(2000).pipe(mapTo('second'), take(3)), // will take 6 seconds to complete
  interval(3000).pipe(mapTo('third'), take(2)), // will take 6 seconds to complete
  2
).subscribe(observer);

// output:
// (sau 1s):
// first
// (sau 2s):
// first
// second
// (sau 3s):
// first
// (sau 4s):
// first
// second
// (sau 5s):
// first (first complete (take(5)), third sẽ bắt đầu được subscribe)
// (sau 6s):
// second (second complete (take(3)))
// (sau 8s):
// third (vì third bắt đầu đc subscribe ở giây thứ 5)
// (sau 11s):
// third (third complete vì take(2))
```

Các bạn sẽ thấy khi truyền vào tham số `concurrent` là 2, `merge` sẽ chỉ subscribe vào `first` và `second` song song mà thôi. Cho đến khi `first` complete, thì `third` mới bắt đầu đc subscribe. Điều này cũng sẽ cho các bạn thấy được rằng thật ra `concat()` chính là `merge()` với `concurrent` là 1.

#### Use-case:

Trong **Angular**, `merge()` có thể được sử dụng khi các bạn có 1 `FormGroup` và các bạn muốn lắng nghe vào từng `FormControl.valueChanges` để thực hiện 1 nghiệp vụ nào đó. Lúc này, các bạn không hề quan tâm thứ tự việc `FormControl` nào sẽ thay đổi, các bạn chỉ cần quan tâm là nếu `FormControl` đó thay đổi thì sẽ xử lý hợp lý.

```ts
const formControlValueChanges = Object.keys(this.formGroup.value).map((key) =>
  this.formGroup.get(key).valueChanges.pipe(map((value) => ({ key, value })))
); // ví dụ từ {firstName: 'Chau', lastName: 'Tran'} -> [Observable<{key, value}>, Observable<{key, value}>]
merge(...formControlValueChanges).subscribe(({key, value}) => {
  if (key === 'firstName') {...};
  if (key === 'lastName') {...};
});
```

### race()

`race<T>(...observables: any[]): Observable<T>`

`race()` là một operator khá hay ho và khá hữu ích trong 1 số trường hợp nhất định. `race()` có tham số giống như `merge()` và `concat()` nên mình sẽ không lặp lại nữa.

- `race()` sẽ emit giá trị của `Observable` nào emit đầu tiên (nhanh nhất) sau đó lặp lại cho đến khi 1 trong các children `Observables` complete.
- `race()` sẽ error ngay lập tức nếu `Observable` _nhanh nhất_ lại throw error thay vì emit giá trị.

```ts
race(
  interval(1000).pipe(mapTo('fast')),
  interval(2000).pipe(mapTo('medium')),
  interval(3000).pipe(mapTo('slow'))
).subscribe(observer);
// output: fast - 1s -> fast - 1s -> fast - 1s -> fast...
```

#### Use-case:

Ở một ứng dụng bất kỳ, các bạn lâu lâu sẽ phải hiển thị 1 Banner nào đó dựa vào hành động của người dùng. Ví dụ: Người dùng vừa submit 1 form, bạn hiển thị 1 Banner ([ng-ant-zorro Alert](https://ng.ant.design/components/alert/en)) báo người dùng là họ submit thành công, hoặc họ có gặp lỗi. Nghiệp vụ lúc này muốn Banner này sẽ tắt đi khi 1 trong 3 điều kiện sau được thoả:

- Sau khi hiển thị được 10s
- Người dùng click đóng Banner
- Người dùng navigate sang một page khác.

Lúc này, operator thích hợp nhất chính là `race()` vì các bạn chỉ muốn đóng Banner khi 1 trong 3 điều kiện trên được thoả mà thôi.

```ts
race(
  timer(10000), // timer 10 second
  this.userClick$, // user click event
  this.componentDestroy$ // navigate -> ngOnDestroy
)
  .pipe(takeUntil(this.componentDestroy$)) // chúng ta cũng sẽ ko muốn lắng nghe vào race nữa nếu như componentDestroy$
  .subscribe(() => this.closeBanner());
```

Tất cả các operators trên đây đều là `static function`. Các operators sau sẽ là các `pipeable operator`, nghĩa là các operator sau đây đều được dùng với `pipe()` và sẽ được bao bên ngoài 1 `Observable` gọi là **Outer Observable**.

### withLatestFrom()

`withLatestFrom<T, R>(...args: any[]): OperatorFunction<T, R>`

`withLatestFrom()` nhận vào tham số là 1 `Observable`. `withLatestFrom()` sẽ gộp giá trị emit của **Outer Observable** với giá trị gần nhất của tham số `Observable` thành 1 `Array` rồi emit `Array` này.

![RxJS withLatestFrom](assets/rxjs-withLatestFrom.png)

```ts
fromEvent(document, 'click')
  .pipe(withLatestFrom(interval(1000)))
  .subscribe(observer);
// output:
// - click trước 1s --- chờ đến 1s --> [MouseEvent, 0]
// - click sau 1s -> [MouseEvent, 0];
// - click lúc 5.5s -> [MouseEvent, 4]; // sau 5s thì giá trị gần nhất của interval(1000) là 4 (0 - 1 - 2 - 3 -4)
```

`withLatestFrom()` cũng nhận vào tham số thứ 2 optional là `projectFunction`. Cách thức hoạt động như những `projectFunction` được đề cập trong bài viết này.

#### Use-case

Vì tính chất chỉ emit khi **Outer Observable** emit nên `withLatestFrom()` sẽ phù hợp với những nghiệp vụ mà các bạn cần lắng nghe 1 `Observable` (đây là **Outer Observable**) và cần thêm giá trị gần nhất của 1 `Observable` khác. Nếu dùng `combineLatest()` thì mỗi lần `Observable` khác kia emit, thì `combineLatest()` cũng emit và đây là điều chúng ta không muốn.

```ts
this.apiService.getSomething().pipe(withLatestFrom(this.currentLoggedInUser$));
// các bạn gọi một API và các bạn muốn dùng kết quả của API này + với thông tin của người dùng đang đăng nhập để thực hiện nghiệp vụ ké tiếp
```

### startWith()

`startWith<T, D>(...array: (SchedulerLike | T)[]): OperatorFunction<T, T | D>`

`startWith()` là 1 operator rất dễ hiểu. `startWith()` nhận vào 1 list các tham số. `startWith()` sẽ làm cho cả `Observable` emit giá trị của `startWith()` trước rồi mới emit đến giá trị của **Outer Observable**. `startWith()` sẽ emit giá trị ngay lặp tức mà không phụ thuộc vào việc **Outer Observable** có emit hay là chưa.

![RxJS startWith](assets/rxjs-startWith.png)

```ts
of('world').pipe(starWith('Hello')).subscribe(observer);
// output:
// 'Hello'
// 'word'
// 'complete'
```

#### Use-case

`startWith()` có thể được dùng trong **Angular** để cung cấp giá trị ban đầu cho các API call. Ví dụ:

```ts
this.books$ = this.apiService.getBooks().pipe(startWith([]));
```

```html
<ng-container *ngIf="books$ | async as books">
  <!-- vì books$ đã có startWith([]) nên giá trị của books sẽ là truthy ngay từ đầu, nên *ngIf này sẽ truthy và render bên trong ng-container ngay -->
</ng-container>
```

### endWith()

`endWith<T>(...array: (SchedulerLike | T)[]): MonoTypeOperatorFunction<T>`

`endWith()` cũng nhận vào 1 list các tham số như `startWith()` nhưng cách hoạt động thì ngược lại với `startWith()`. Một điểm khác biệt lớn là `endWith()` chỉ emit giá trị của `endWith()` khi **Outer Observable** complete mà thôi.

![RxJS endWith](assets/rxjs-endWith.png)

```ts
of('hi', 'how are you?', 'sorry, I have to go now')
  .pipe(endWith('goodbye!'))
  .subscribe(observer);
// output:
// 'hi'
// 'how are you?'
// 'sorry, I have to go now'
// 'goodbye!'
```

### pairwise()

`pairwise<T>(): OperatorFunction<T, [T, T]>`

`pairwise()` là 1 operator rất thú vị và rất kén nghiệp vụ. `pairwise()` sẽ gộp giá trị emit gần nhất và giá trị đang được emit của **Outer Observable** thành 1 `Array` (1 cặp giá trị) và emit `Array` này.

![RxJS pairwise](assets/rxjs-pairwise.png)

```ts
from([1, 2, 3, 4, 5])
  .pipe(
    pairwise(),
    map((prev, cur) => prev + cur)
  )
  .subscribe(observer);
// output:
// 3 (1 + 2)
// 5 (2 + 3)
// 7 (3 + 4)
// 9 (4 + 5)
```

## Summary

Lại 1 ngày quá nhiều kiến thức 💪. Một lần nữa, các operators và use-case mình liệt kê ở đây là những gì mình có thể nghĩ tới được, hoặc đã từng áp dụng. Các bạn có nghiệp vụ nào dùng qua những operators này thì post lên chia sẻ nhé. Hẹn gặp lại các bạn trong ngày mai.

Mục tiêu ngày 24 sẽ là **RxJS Error handling/conditional Operators**

## References

- [RxJS Overview](https://rxjs.dev/guide/overview)
- [LearnRxJS](https://www.learnrxjs.io/)

## Author

[Chau Tran](https://github.com/nartc)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day23`
