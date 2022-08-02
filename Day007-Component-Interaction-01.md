# Day 7: COMPONENT INTERACTION - PASS DATA FROM PARENT TO CHILD WITH INPUT BINDING

Trong các ứng dụng thực tế, đôi khi chúng ta sẽ có những component mà không cần truyền gì vào nó vẫn sẽ hoạt động, nhưng có rất nhiều component mà khi thiết kế chúng ta mong muốn có thể tái sử dụng cao hơn, tùy thuộc vào các properties được truyền vào mà sẽ hiển thị, xử lý tương ứng. Giả sử khách hàng yêu cầu chúng ta tạo một ứng dụng, mà trong ứng dụng đó sẽ có chức năng hiển thị progress, ví dụ progress cho upload file, progress cho người dùng biết họ đang xem được bao nhiêu phần trăm của một video, vân vân. Lúc này chúng ta có thể viết một component progress bar, mà chỉ cần các nơi sử dụng nó sẽ truyền vào các config tương ứng thì nó sẽ hiển thị theo yêu cầu, thay vì mỗi nơi lại viết một component, khó có thể tái sử dụng được.

Làm sao để có thể tạo một component mà nó có thể nhận vào các properties.
Đầu tiên, chúng ta sẽ đi tạo component trước, nếu bạn vẫn chưa biết cách tạo thì có thể Google hoặc xem lại Day 1.
Chúng ta sẽ dùng lệnh để generate 1 component tên là progress-bar

```
ng g c progress-bar
```

Đó là cách viết tắt, hoặc có thể tường minh hơn là:

```
ng generate component progress-bar
```

Thế là xong, chúng ta đã có một component để bắt đầu.

## @INPUT DECORATOR

Để thêm một property cho phép thiết lập progress hiện tại của thanh progress-bar, chúng ta có thể khai báo một property cho class và thêm một móc móc (decorator) như sau:

```typescript
export class ProgressBarComponent implements OnInit {
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```

Đơn giản thế là chúng ta đã có thể báo cho Angular biết rằng, chúng ta cần component này nhận vào một property tên là progress, và giá trị mặc định của nó bằng 0.
Bây giờ, chúng ta có thể thêm một số property khác cho component như property về màu sắc chẳng hạn.

```typescript
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```

Như vậy, component của chúng ta sẽ có thể nhận vào 3 properties, với property progress sẽ có thể chứa giá trị default bằng 0.
Lưu ý rằng, @Input được gọi là một property decorator, nó sẽ gắn thêm meta data cho property ngay phía sau nó.
Nếu bạn không khai báo decorator @Input thì sẽ không thể nhận giá trị truyền vào từ component khác, vì Angular sẽ không biết cách để binding, và do đó property của bạn chỉ là một property bình thường của class.

Khi đã có component và khai báo input rồi thì làm sao để sử dụng. Các bạn cần nhớ lại property binding ở trong những buổi đầu tiên, lúc này chỉ cần dùng cú pháp vuông vuông để binding cho property các bạn muốn là được rồi.

```html
<app-progress-bar
  [progress]="15"
  [backgroundColor]="'#9e9e9e'"
  [progressColor]="'#2e8b57'"
>
</app-progress-bar>
```

Đây là component progress-bar của chúng ta.

```typescript
import { Component, OnInit, Input } from '@angular/core';

@Component({
  selector: 'app-progress-bar',
  template: `
    <div
      class="progress-bar-container"
      [style.backgroundColor]="backgroundColor"
    >
      <div
        class="progress"
        [style]="{
          backgroundColor: progressColor,
          width: progress + '%'
        }"
      ></div>
    </div>
  `,
  styles: [
    `
      .progress-bar-container,
      .progress {
        height: 20px;
      }

      .progress-bar-container {
        width: 100%;
      }
    `,
  ],
})
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;
  constructor() {}
  ngOnInit() {}
}
```

## NGONINIT VS CONSTRUCTOR

Khi làm việc các bạn có thể thấy rằng Angular giới thiệu một số method gọi là life-cycle, chúng làm nhiệm vụ gì.
Theo như lẽ thông thường, con người từ khi sinh ra đến khi chết đi sẽ đều trải qua một số sự kiện trọng đại trong đời, ví du: sinh ra, đầy tháng, sinh nhật hàng năm, kết hôn, chết đi. Tương tự như thế trong ứng dụng Angular sẽ có vòng đời cho các component. Angular được xây dựng xung quanh Component và Directive, ở một thời điểm có thể có 1 component được khởi tạo, một thời điểm khác chúng lại được xóa đi khỏi view, và có một số “sự kiện” khác cũng được xảy ra, do đó Angular cung cấp cho chúng ta một số method mà chúng ta chỉ việc khai báo cho component/directive, còn lại Angular sẽ tự call khi có những sự kiện tương ứng xảy ra.
Ở trong code phía trên, chúng ta được giới thiệu constructor và ngOnInit, vậy chúng giống và khác nhau gì?

Constructor là hàm tạo của một class, nó là một function đặc biệt mà khi bạn khởi tạo một instance của class thì nó sẽ được tự động chạy, và chỉ chạy duy nhất một lần.
ngOninit là một life-cycle method, nó sẽ được Angular tự động gọi khi component được khởi tạo, sau khi constructor chạy và sau khi các input đã được binding.
Do đó nếu bạn binding cho một property ở template của component cha, thì ở constructor của component con bạn sẽ chưa nhận được giá trị mà bạn đã binding, nhưng ở ngOnInit thì bạn sẽ có thể nhận được.

Trong thực tế, Angular khuyến cáo hạn chế code ở constructor, constructor làm càng ít nhiệm vụ càng tôt, hãy để ngOnInit lo tiếp phần việc còn lại.

## THAY ĐỔI GIÁ TRỊ CỦA INPUT.

Giả sử trong trường hợp nào đó mà chúng ta không biết người dùng binding những dữ liệu có hợp lệ không (điều này hoàn toàn có thể xảy ra như trường hợp người dùng truyền data kiểu any vào chẳng hạn), và chúng ta muốn validate Input thì có cách nào không?

Lúc này bạn sẽ dễ dàng validate lần đầu tiên ở trong ngOnInit được. Nhưng như thế ở các lần sau ngOnInit không chạy lại thì cũng không phải là giải pháp toàn diện. Đây chính là lúc mà bạn có thể sử dụng life-cycle tiếp theo là ngOnChanges.

ngOnChanges sẽ chạy lại mỗi khi có một input nào bị thay đổi, nó sẽ được tự động gọi bởi Angular, do đó chúng ta có thể validate property progress như sau:

```typescript
export class ProgressBarComponent implements OnInit, OnChanges {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  @Input() progress = 0;

  constructor() {}

  ngOnChanges(changes: SimpleChanges) {
    if ('progress' in changes) {
      if (typeof changes['progress'].currentValue !== 'number') {
        const progress = Number(changes['progress'].currentValue);
        if (Number.isNaN(progress)) {
          this.progress = 0;
        } else {
          this.progress = progress;
        }
      }
    }
  }

  ngOnInit() {}
}
```

Trong trường hợp bạn không thích dùng ngOnChanges, chúng ta có thể sử dụng getter/setter để làm điều này.

```typescript
export class ProgressBarComponent implements OnInit {
  @Input() backgroundColor: string;
  @Input() progressColor: string;
  private $progress = 0;
  @Input()
  get progress(): number {
    return this.$progress;
  }
  set progress(value: number) {
    if (typeof value !== 'number') {
      const progress = Number(value);
      if (Number.isNaN(progress)) {
        this.$progress = 0;
      } else {
        this.$progress = progress;
      }
    } else {
      this.$progress = value;
    }
  }

  constructor() {}

  ngOnInit() {}
}
```

## SUMMARY

Như vậy trong Day 7, chúng ta sẽ phải tìm hiểu cách để khai báo input binding cho một component, từ đó giúp dễ dàng thiết kế các component có tính tái sử dụng cao.

Ngoài những gì trong report trên, chúng ta có thể tìm hiểu thêm về việc tạo alias cho property và inputs array trong bài sau:

- https://www.tiepphan.com/thu-nghiem-voi-angular-2-truyen-du-lieu-cho-component-voi-input/

Dưới đây là các link document mà các bạn cần tìm hiểu:

- https://angular.io/guide/component-interaction
- https://angular.io/guide/lifecycle-hooks
- https://www.tiepphan.com/thu-nghiem-voi-angular-2-truyen-du-lieu-cho-component-voi-input/

## Youtube Video

[![Day 07](https://img.youtube.com/vi/uTd2W4NQkgs/0.jpg)](https://youtu.be/uTd2W4NQkgs)

## Code sample

https://stackblitz.com/edit/angular-ivy-100-days-of-code-day-7

Mục tiêu của Day 8 là child component sẽ thông báo lại cho parent component một event với @Output decorator.

## Author

[Tiep Phan](https://github.com/tieppt)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day7`
