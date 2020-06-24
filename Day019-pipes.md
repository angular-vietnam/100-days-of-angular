# Day 19: Transform data with Angular Pipes

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
2. Viêt một pipe, cũng nhận input và return output. Điểm lợi thế của Pipe là dễ reuse. Vì thông thường sẽ có khá nhiều page cần hiển thị date time, việc dùng Pipe sẽ đem lại nhiều ưu điểm hơn là function.

## Dùng pipe như thế nào?

Angular có cung cấp sẵn một số pipes thường dùng trong package `@angular/common`. Tuy nhiên số lượng pipe có sẵn đó cũng không thể nào đáp ứng được hết các nhu cầu trong các ứng dụng khác nhau, nên chúng ta cũng hoàn toàn có thể viết các custom pipe theo nhu cầu thực tế.

Như đã nói, pipe sẽ nhận vào input và output ra một giá trị mình mong muốn.

Mình có một biến tên là `date` ở trong component.

```ts
export class PipeExampleComponent implements OnInit {
  now = "2020-06-24T09:00:00.000Z";
}
```

Và đây là cách mình hiển thị với built in pipe trong Angular

```html
{{ now | date}} //Jun 24, 2020
{{ now | date:'medium'}} //Jun 24, 2020, 4:00:00 PM
```

Chú ý phần interpolation giữa hai dấu ngoặc nhọn `{{ }}`, ngoài việc truyền vào variable bạn muốn hiển thì thì có thêm dấu xổ dọc `|`. Đó là pipe operator, sau đó là tên của pipe bạn đã định nghĩa. Tất cả pipe đều hoạt động theo cách này.

## Các pipe có sẵn đi kèm với Angular (built-in pipe)

## Code sample

https://stackblitz.com/edit/angular-100-days-of-code-day-19-pipes

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day19`
