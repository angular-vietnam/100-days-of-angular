# Day 18: Transform data with Angular Pipes

Các ứng dụng thông thường đều bao gồm các tác vụ khá đơn giản:

1. Lấy dữ liệu từ server. Đơn giản là gọi API call lên server, phức tạp thì listen tới một websocket để nhận được dữ liệu theo thời gian thực.
2. Transform the data, ví dụ như bạn nhận được `firstName` và `lastName` từ server. Nhưng trên UI mình phải show `{firstName} ${lastName}`.
3. Và hiển thị dữ liệu lên UI cho người dùng.

Pipes sẽ lo phần thứ 2, transform data trước khi show cho người dùng.

## Pipes là gì?

Hiểu đơn giản, pipe là một function nhận **input** mà chúng ta truyền vào, và **output** ra giá trị mình mong muốn.

Ví dụ giữa client và server khi trao đổi thông tin liên quan đến thời gian, thường dùng ISO format `"2020-06-24T09:00:00.000Z"`, tương đương với ngày 26 tháng 6, 4h chiều (giờ Việt Nam).

Tuy nhiên khi hiển thị, mình ko thể hiển trị trực tiếp ISO string cho người dùng vì chắc chắn là ko phải ai cũng là developer để hiểu được đó là gì. Vậy nên chúng ta cần transform ISO string ở trên dưới dạng mà người dùng có thể hiểu được, ví dụ `Jun 24, 2020, 4:00:00 PM`.

Để làm được việc này bạn có khá nhiều lựa chọn, nhưng thường thì có hai lựa chọn trong Angular:

1. Viết một function, nhận date input và return output.
2. Viêt một pipe, cũng nhận input và return output. 

Điểm lợi thế của Pipe là dễ tái sử dụng. Vì thông thường sẽ có khá nhiều page cần hiển thị date time, việc dùng Pipe sẽ đem lại nhiều ưu điểm hơn là function.

## Dùng pipe như thế nào?

Angular có cung cấp sẵn một số pipes thường dùng trong package `@angular/common`. Tuy nhiên số lượng pipe có sẵn đó cũng không thể nào đáp ứng được hết các nhu cầu trong các ứng dụng khác nhau, nên chúng ta cũng hoàn toàn có thể viết các custom pipe theo nhu cầu thực tế.

Như đã nói, pipe sẽ nhận vào input và output ra một giá trị mình mong muốn.

Mình có một biến tên là `date` ở trong component.

```ts
export class PipeExampleComponent implements OnInit {
  now = "2020-06-24T09:00:00.000Z";
}
```

Và đây là cách mình hiển thị với built in pipe [Date][date] trong Angular

```html
{{ now | date }} //Jun 24, 2020 
{{ now | date:'medium'}} //Jun 24, 2020, 5:00:00 PM
```

Chú ý phần giữa hai dấu ngoặc nhọn `{{ }}`, ngoài việc truyền vào variable bạn muốn hiển thì thì có thêm dấu xổ dọc `|`. Đó là pipe operator, sau đó là tên của pipe bạn đã định nghĩa. Tất cả pipe đều hoạt động theo cách này.

```html
{{ interpolated_value | pipe_name }}
```

### Pipe and parameters

Pipe cho phép truyền thêm các parameters, ví dụ `date` ở trên mình có thể truyền thêm format `medium` phân tách nhau bằng dấu hai chấm `:`. Đó cũng là cú pháp để pass parameter cho pipe.

```
{{ interpolated_value | pipe_name:parameter1:parameter2:...:parameterN }}
```

Bạn có thể truyền vào số lượng parameter không giới hạn.

### Chaining pipe

Pipes cho phép chúng ta dùng nhiều pipe để transform một value, cú pháp có dạng như sau.

```html
{{ interpolated_value | pipe_name_1 | pipe_name_2 |... | pipe_name_n }}
```

Sau khi có output `pipe_name_1`, thì output này sẽ được xử lý qua `pipe_name_2` rồi tương tự đến `pipe_name_n` để ra output cuối cùng hiển thị lên UI. Vẫn ví dụ với date ở trên, mình sẽ thêm pipe `uppercase` để chuyển hết text thành chữ hoa. Sau khi biến `now` được xử lý bằng `date` pipe sẽ có value `Jun 24, 2020, 5:00:00 PM`, sau đó `uppercase` sẽ transform thành `JUN 24, 2020, 5:00:00 PM`.

Thứ tự thực hiện sẽ là từ trái qua phải. Sau khi `pipe_name_1` chạy xong có value thì `pipe_name_2` sẽ nhận vào output từ `pipe_name_1`.

> Mình đang ở Sing và máy tính được config múi giờ +8 nên mình nhìn thấy value là 5 PM. Còn các bạn đa phần sẽ nhìn thấy giá trị là 4 PM vì cấu hình máy tính ở giờ +7

```
{{ now | date:'medium' | uppercase}} // JUN 24, 2020, 5:00:00 PM
```

## Các pipe có sẵn đi kèm với Angular (built-in pipe)

Angular cung cấp sẵn khá nhiều pipes để có thể sử dụng được khi import `CommonModule` từ package `@angular/common`. Dưới đây là một số pipe mình hay dùng:

| Pipe                                                           | Mô tả                                                                      |
| -------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [`DatePipe`](https://angular.io/api/common/DatePipe)           | Formats a date.                                                            |
| [`UpperCasePipe`](https://angular.io/api/common/UpperCasePipe) | Convert text sang chữ hoa.                                                 |
| [`LowerCasePipe`](https://angular.io/api/common/LowerCasePipe) | Convert text sang chữ thường.                                              |
| [`CurrencyPipe`](https://angular.io/api/common/CurrencyPipe)   | Hiển thị giá trị tiền tệ.                                                  |
| [`DecimalPipe`](/https://angular.io/api/common/DecimalPipe)    | Hiển thị số thập phân                                                      |
| [`PercentPipe`](https://angular.io/api/common/PercentPipe)     | Hiển thị phần trăm %                                                       |
| [`JsonPipe`](https://angular.io/api/common/JsonPipe)           | Hiển thị json                                                              |
| [`AsyncPipe`](https://angular.io/api/common/AsyncPipe)         | Hiển thị value của observable và tự động unsubscribe khi view được destroy |

Toàn bộ các pipe có sẵn trong `CommonModule`, bạn có thể xem thêm [ở đây][pipes]

## Viết custom pipe

Ví dụ này lấy từ chính application đang chạy của mình. Use case đơn giản là bọn mình cần code rất nhiều form CRUD. Vì thường form dành cho add item và form dành cho edit item sẽ được reuse cùng code HTML. Nếu bạn click Add item, form add sẽ được hiển thị và title của form lúc đó sẽ là Add Item, tương tự cho edit.

Khi mở form Edit, trong router có truyền itemId lên URL nên bọn mình biết được đây là form Edit. Còn form Add thì sẽ không có itemId.

Thông thường bọn mình có thể viết đi viết lại một cái logic trong từng component.

```html
{{ itemId ? "Edit" : "Add" }}
```

Cho đến một ngày một dev mắc một lỗi typo ngớ ngẩn là thay vì `Add`, bạn ấy type thành `Adđ` (Vì có bật Unikey :)))

Thế là mình quyết định viết một pipe đơn giản là nhận vào một string, nếu string này có value, show Edit, còn không thì show Add. Tránh được lỗi typo như ở trên về sau. 

Để viết một pipe dành riêng cho nhu cầu của từng dự án, cần follow hai bước sau.

### 1. Trước tiên chúng ta cần tạo một class có implement interface [`PipeTransform`][pipeTransform]. 

Interface này chỉ bao gồm một method duy nhất tên là `transform`.

Đây là interface `PipeTransform` cần implement

```ts
interface PipeTransform {
  transform(value: any, ...args: any[]): any
}
```

Đây là ví dụ một class sau khi đã implement `PipeTransform`

```ts
export class AppTitlePipe implements PipeTransform {
    transform(resourceId: string): string {
        return resourceId ? "Edit" : "Add";
    }
}
```

Đại khái là method transform ở đây rất đơn giản. Nếu resourceId là truthy thì return `Edit`, nếu ko thì return lại `Add`.

### Thêm Pipe decorator cho class đã implement PipeTransform

Giống như component có decorator `@Component`. Pipe cũng có decorator `@Pipe`. 

```
@Pipe({
    name: 'appTitle'
})
export class AppTitlePipe implements PipeTransform {
    transform(resourceId: string): string {
        return resourceId ? "Edit" : "Add";
    }
}
```

Khi thêm Pipe decorator thì có một property là required, đó chính là tên của pipe. Mình đặt là `appTitle`.

Nhớ là phải đặt `AppTitlePipe` trong mảng declarations ở module tương ứng mà bạn muốn sử dụng. Nếu không Angular sẽ báo lỗi.

Xong rồi đây, giờ mình có thể dùng `appTitle` như bình thường.

```html
<h2 class="ibox-title">{{ userId | appTitle }} User</h2>
```

> Chú ý cách đặt tên cho pipe và class:

- Class name follow `UpperCamelCase`, tức là viết hoa các chữ cái đầu của từng từ
- `name` của pipe sẽ follow theo `camelCase`, tức là chữ cái đầu của từ đầu tiên viết thường. Các chữ cái đầu của các từ tiếp theo viết hoa.
- Không được dùng dấu gạch ngang `-` cho `name`

Chi tiết có trong [Angular Style Guide][styleguide]

### Custom pipe parameters

Vẫn là ví dụ trên, nhưng giờ có yêu cầu ở một vài page là khi mở form Add, sẽ không hiện title là Add nữa, mà đổi lại thành Set. Ví dụ `Set Item`. Còn form Edit thì đổi lại thành Change.

Mình hoàn toàn có thể truyền vào hai parameters tương ứng với hai text này. Và nếu mặc định không truyền mình sẽ set lại Add và Edit tương ứng.

Code của mình có thể được viết lại như sau:

```ts
  transform(
    resourceId: string,
    addText: string = "Add",
    editText: string = "Edit"
  ): string {
    return resourceId ? editText : addText;
  }
```

Và dùng trên UI

```html
{{ userId | appTitle:"Set":"Change"}}
```

Method `transform` sẽ nhận vào nhiều argument. Trong đó:

- Argument đầu tiên chính là value của variable khi mình dùng pipe. Ví dụ `{{ userId | appTitle }}` thì `transform(resourceId: string)`. resourceId chính là value của userId được truyền vào.
- Khi truyền các parameter khác bằng dấu hai chấm `:` thì argument tương ứng trong method `transform` sẽ là từ argument thứ 2 trở đi. Ví dụ như `{{ userId | appTitle:"Set":"Change"}}` thì "Set" sẽ là value của `addText: string` và "Change" sẽ tương ứng với `editText: string`

## Detecting changes with data binding in pipes


## Code sample

https://stackblitz.com/edit/angular-100-days-of-code-day-18-pipes

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day18`

[date]: https://angular.io/api/common/DatePipe
[pipes]: https://angular.io/api/common/CommonModule#pipes
[pipeTransform]: https://angular.io/api/core/PipeTransform
[styleguide]: https://angular.io/guide/styleguide#pipe-names