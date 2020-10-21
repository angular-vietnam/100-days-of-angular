# Day 10: Template Variable và ViewChild/ViewChildren

Nếu bạn cần trỏ tới một phần tử (HTMLElement/component/directive) ở trong template và thao tác trực tiếp lên nó thì sao. Có cách nào để chúng ta tạo ra một `variable` ở trong template và sử dụng nó không? Câu hỏi trên sẽ được trả lời trong ngày thứ 10 này.

## Parent interacts with child via local variable
Giả sử chúng ta có `AppComponent` có nhúng một phần template như sau:

```html
<app-toggle></app-toggle>
```

Nhưng thay vì click vào `ToggleComponent` để thay đổi trạng thái của nó thì chúng ta sẽ dùng một button ở parent (AppComponent) như template dưới đây:

```html
<button (click)="doSomething">Toggle</button>

<br>

<app-toggle></app-toggle>
```

Làm thế nào để gọi được method `toggle()` của `ToggleComponent`?

Lúc này bạn sẽ có thể sử dụng đến Template variable như một giải pháp hữu hiệu.

```html
<button (click)="toggleComp.toggle()">Toggle</button>

<br>

<app-toggle #toggleComp></app-toggle>
```

Cú pháp của Template variable chính là sử dụng `#varName` và bạn có thể tạo nhiều variable trong template.

Như trước đây làm với ví dụ về `NgIf-Else` chúng ta cũng đã sử dụng Template variable để lấy về một instance của `ng-template` và truyền vào cho `NgIfElse` như sau:

```html
<div *ngIf="user.age >= 13; else noPG13">
  Bạn có thể xem nội dung PG-13
</div>
<ng-template #noPG13>
  <div>
    Bạn không thể xem nội dung PG-13
  </div>
</ng-template>
```

## Template variable sẽ là instance của class nào?

Thông thường, một element trong template của một component sẽ có thể là một HTMLElement, hoặc một Component. Nhưng có những trường hợp sẽ có nhiều Directives cùng được đặt vào một element, vậy Template variable sẽ cho chúng ta một object thuộc kiểu nào?

Mặc định Template variable mà không có expression `#varName` sẽ cố gắng lấy về một object, trong đó đối với HTMLElement thì object đó chính là HTMLElement, còn với Component thì chính là instance của Component đó.

Trong một số trường hợp bạn cần lấy chính xác một instance của một directive/component nào đó thì bạn sẽ cần sử dụng cú pháp `#varName="exportAsOfDirectiveOrComponent"`, ví dụ như khi làm việc với `FormsModule` và `ngModel`, bạn có thể nhìn thấy cú pháp sau:

```html
<form #nameForm="ngForm">
	<input
    type="text" class="form-control"
    required
    [(ngModel)]="model.name" name="name"
    #name="ngModel">
  <button>Submit</button>
</form>
```

Ở template trên chúng ta đã tạo ra 2 Template variable là:
- `nameForm`: mong muốn lấy instance của directive có `exportAs` là `ngForm`
- `name`: mong muốn lấy instance của directive có `exportAs` là `ngModel`

Nếu như chúng ta không request chính xác type chúng ta mong muốn là gì thì các Template variable trên chỉ lấy về HTMLElement thông thường.

## Parent class with ViewChild

Vậy nếu bạn muốn gọi đến Template variable ở trong class của một component thì sao? Giải pháp ở đây là gì, vì nếu chỉ ở ngoài template thì có vẻ nó không được mạnh mẽ cho lắm, vì code logic của chúng ta nên ở trong component (và service - sau này sẽ học) thay vì ở ngoài template.

Lúc này chúng ta có thể query một Template variable ở trong Component như sau:

```html
<button (click)="toggleInside()">Toggle inside class</button>
<br>
<br>

<app-toggle #toggleComp></app-toggle>
```

```ts
export class AppComponent  {
  @ViewChild('toggleComp') toggleComp: ToggleComponent;
  toggleInside() {
    this.toggleComp.toggle();
  }
}
```

Nếu bạn sử dụng ViewChild cho một HTMLElement thì chúng ta sẽ nhận được một `ElementRef` thay vì một HTMLElement như sử dụng ở trong template.

```html
<div #chartContainer></div>
```

```ts
export class AppComponent  {
  @ViewChild('chartContainer') container: ElementRef<HTMLDivElement>;
}
```

Lưu ý: Ngoài những thiết lập mặc định ở trên cho ViewChild, chúng ta còn có thể truyền vào config cho nó với các thông số chi tiết trong link sau:

[https://angular.io/api/core/ViewChild](https://angular.io/api/core/ViewChild)

Cú pháp để sử dụng sẽ như sau:

```ts
// View queries are set before the ngAfterViewInit callback is called.
ViewChild(selector: string | Function | Type<any>, opts?: {
  read?: any;
  static?: boolean;
})
```

Trong đó các `selector` có thể là:
- Any class with the @Component or @Directive decorator
- A template reference variable as a string (e.g. query <my-component #cmp></my-component> with @ViewChild('cmp'))
- Any provider defined in the child component tree of the current component (e.g. @ViewChild(SomeService) someService: SomeService)
- Any provider defined through a string token (e.g. @ViewChild('someToken') someTokenVal: any)
- A TemplateRef (e.g. query <ng-template></ng-template> with @ViewChild(TemplateRef) template;)

`opts.read` có thể là bất cứ một token nào - có thể là một directive, một component, một service, etc. Nếu match với token nào thì sẽ trả về. Ví dụ:

```html
<form #nameForm="ngForm">
	<input
    type="text" class="form-control"
    required
    [(ngModel)]="model.name" name="name"
    #name="ngModel">
  <button>Submit</button>
</form>
```

```ts
export class NameFormComponent implements OnInit {
  model = {
    name: 'Tiep Phan'
  };

  @ViewChild('nameForm', {
    read: ElementRef,
    static: true
  }) form: ElementRef<HTMLFormElement>;
  constructor() { }

  ngOnInit() {
    console.log(this.form)
  }

}
```

Ở trong trường hợp trên, nếu chúng ta không khai báo `read` thì sẽ lấy về NgForm instance, nhưng do khai báo là một ElementRef nên nó sẽ query khác với variable ở ngoài template.

`opts.static` nếu `selector` không nằm trong if/else hay một structure directive nào thì chúng ta có thể gọi nó là `static: true`, tức là nó không thay đổi trong suốt thời gian sống của component. Lúc này Angular (v9 trở lên) sẽ chạy phần `resolve query result` (tiến trình) trước khi chạy *Change Detection* nên chúng ta có thể truy cập nó ở trong `ngOnInit` như ở trên, nếu `static: false` (giá trị mặc định) thì tiến trình trên sẽ chạy sau khi chạy *Change Detection* nên bạn không thể dùng nó ở `ngOnInit` mà phải chạy ở `ngAfterViewInit`.

## Parent class with ViewChildren

Khi bạn muốn query một danh sách các element thì bạn có thể dùng ViewChildren.

ViewChildren sẽ trả về một QueryList trước khi `ngAfterViewInit` được chạy. Nó sẽ chứa một số property/method để chúng ta có thể listen vào một số event (Observable).

```html
<app-toggle></app-toggle>
<br>
<app-toggle></app-toggle>
```
```ts
@ViewChildren(ToggleComponent) toggleList: QueryList<ToggleComponent>;


ngAfterViewInit() {
  console.log(this.toggleList);
}
```

## SUMMARY

Như vậy trong Day 10, chúng ta cần tìm hiểu về Template variable và cách sử dụng ViewChild/ViewChildren ở trong component class. Ngoài ra, các bạn cần lưu ý các options có thể thêm vào cho ViewChild/ViewChildren theo các link dưới đây.

Cũng trong Day 10 chúng ta học thêm một component lifecycle khác là `ngAfterViewInit` để có thể thao tác được với ViewChild/ViewChildren.

- https://angular.io/api/core/ViewChild
- https://angular.io/api/forms/NgModel
- https://angular.io/api/core/ViewChildren
- https://www.tiepphan.com/thu-nghiem-voi-angular-template-variable-trong-angular/
- https://www.tiepphan.com/angular-trong-5-phut-dynamic-component-rendering/

## Youtube Video

https://youtu.be/Wd_644YBQUM

## Code sample

https://stackblitz.com/edit/angular-ivy-100-days-of-code-day-10?file=src/app/app.component.ts

Mục tiêu của Day 11 là TypeScript Basic: Data Type & Class.

## Author

[Tiep Phan](https://github.com/tieppt)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day10`
