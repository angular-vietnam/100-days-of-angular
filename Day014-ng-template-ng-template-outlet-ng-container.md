# Day 14: ng-template, ngTemplateOutlet và ng-container trong Angular

## ng-template

Trong [bài viết thứ tư][day4] của series, anh @tiepphan đã nói có nói đến một trường hợp dùng `ng-template`. Đó là khi dùng `*ngIf` với điều kiện else, chúng ta có thể truyền vào một template reference đc định nghĩa thông qua cú pháp `#templateReferenceName` để render lên UI.

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

Thông qua ví dụ trên, chắc các bạn cũng đã nhận ra được đôi điều:

- Khi code HTML của bạn dc bao quanh bởi `ng-template`, phần HTML đó sẽ <u>không dc render lên UI ngay lập tức</u>. Mà chỉ dc render trong một số trường hợp, ví dụ như khi `*ngIf else tmpl` hoặc thông qua `ngTemplateOutlet` mà chúng ta sẽ đề cập đến ở phần sau của bài viết.
- Tên gọi của ng-template cũng phần nào nói lên đc ý nghĩa của nó. Template hiểu nôm na là mẫu, dạng. Dịch ra tiếng Việt hơi khó, tuy nhiên khi kết hợp nhiều template với nhau thì chúng ta có thể có một UI đầy đủ.


Từ những điểm trên, có thể định nghĩa `ng-template` là một thành phần của Angular để render HTML code. Và phần HTML code nằm trong `ng-template` không bao giờ được hiển thị trực tiếp ở nơi nó được định nghĩa

### Khi nào thì nên dùng ng-template?

Một số trường hợp hay cần dùng đến ng-template theo như kinh nghiệm của mình.

#### 1. Dùng kết hợp với các Structure Directive của Angular, ví dụ như `*ngIf`

#### 2. Khi một số UI element trong một component bị lặp lại trong chính component đó, nhưng phần code đó quá nhỏ để tách ra làm một component riêng.

Ví dụ như bạn có một component có chứa biến một biển tên là `counter`. Phần UI của counter này sẽ đc lặp lại ở trong component của bạn vài lần với UI giống nhau. 

Đây là cách bình thường chúng ta hay làm. Copy and paste code.

```html
<div class="card">
    <div class="card-header">
        You have selected <span class="badge badge-primary">{{ counter }}</span> items.
    </div>
    <div class="card-body">
        There are <span class="badge badge-primary">{{ counter }}</span> items was selected.
    </div>
    <div class="card-footer">
        You have selected <span class="badge badge-primary">{{ counter }}</span> items.
    </div>
</div>
```

Và đây là cách chúng ta có thể viết lại bằng cách dùng `ng-template` và `ngTemplateOutlet`.

```html
<div class="card">
    <div class="card-header">
        You have selected <ng-container [ngTemplateOutlet]="counterTmpl"></ng-container>.
    </div>
    <div class="card-body">
        There are <ng-container [ngTemplateOutlet]="counterTmpl"></ng-container> was selected.
    </div>
    <div class="card-footer">
        You have selected <ng-container [ngTemplateOutlet]="counterTmpl"></ng-container>.
    </div>
</div>

<ng-template #counterTmpl>
    <span class="badge badge-primary">{{ counter }}</span> items
</ng-template>
```

Khi viết lại code dùng `ng-template`, ưu điểm dễ nhận thấy đó là:

- Nếu cần sửa lại UI cho counter. Thay vì phải sửa ở 3 nơi, bây giờ ta chỉ cần sửa ở một vị trí đó là `ng-template` của counter thôi. Tránh những lỗi typo hay find and replace bị thiếu. 
- Vì phần template này chỉ gói gọn trong đúng một dòng code nên dùng ng-template tiện hơn hẳn là phải tách phần counter ra một component mới.

#### 3. Dùng ng-template để pass vào component khác. Hỗ trợ override template có sẵn trong component.

Ví dụ mình có component `tab-container`, mặc định sẽ render tab với template default là `defaultTabButtonsTmpl`.

```ts
@Component({
  selector: 'tab-container',
  template: `
    <ng-template #defaultTabButtonsTmpl>
      <div class="default-tab-buttons">
        ...
      </div>
    </ng-template>
    <ng-container *ngTemplateOutlet="headerTemplate || defaultTabButtons"></ng-container>
    ... rest of tab container component ...
  `
})
export class TabContainerComponent {
    @Input() headerTemplate: TemplateRef<any>; // Custom template provided by parent
}
```

Tuy nhiên, khi dùng `tab-container` bạn hoàn toàn có thể pass vào template mới để override lại default UI ở parent component.

```ts
@Component({
  selector: 'app-root',
  template: `      
    <ng-template #customTabButtons>
      <div class="custom-class">
        <button class="tab-button" (click)="login()">
          {{loginText}}
        </button>
        <button class="tab-button" (click)="signUp()">
          {{signUpText}}
        </button>
      </div>
    </ng-template>
    <tab-container [headerTemplate]="customTabButtons"></tab-container>      
  `
})
```

## ngTemplateOutlet

Qua ví dụ trên thì có thể thấy ngay `ngTemplateOutlet` là cách dùng để render một template được tạo ra bởi `ng-template` lên UI. Cú pháp như sau

- *ngTemplateOutlet="templateRef"(chú ý dấu sao `*` nhé, không có là không chạy đâu đấy)
- [ngTemplateOutlet]="templateRef"

Tuy nhiên, như component có `@Input()` để truyền data từ bên ngoài vào, thì ng-template cũng có cú pháp tương tự để truyền data. Đó chính là `ngTemplateOutletContext`

Ví dụ như mình muốn reuse lại một button với những tên khác nhau, ngoài ra, một button mình muốn có icon, và một button mình ko muốn có icon. Thì bạn hoàn toàn cũng có thể dùng kết hợp giữa `ng-template`, `ngTemplateOutlet` và `ngTemplateOutletContext` để viết code.

```html
<button class="btn btn-primary">
    Click here
</button>

<button class="btn btn-danger">
    <i class="fa fa-remove"></i>
    Delete
</button>
```

Hoàn toàn có thể được viết lại thành.

```html
<ng-template #buttonTmpl
             let-label="label"
             let-className="className"
             let-icon="icon">
    <button [ngClass]="['btn', className ? className : '']">
        <i *ngIf="icon"
           class="fa {{icon}}"></i>
        {{ label }}
    </button>
</ng-template>

<ng-container [ngTemplateOutlet]="buttonTmpl"
              [ngTemplateOutletContext]="{ label: 'Click here', className: 'btn-primary', icon: null }">
</ng-container>

<ng-container [ngTemplateOutlet]="buttonTmpl"
              [ngTemplateOutletContext]="{ label: 'Remove', className: 'btn-danger', icon: 'fa-remove' }">
</ng-container>
```

Tuy nhiều code hơn, nhưng giờ chúng ta có thể reuse lại button này trong cùng component. Hoặc pass cái template này vào component khác cũng được, ko vấn đề gì cả.

Vài điểm chú ý:

- Khi define ra `ng-template`, bạn có thể config cho template đó nhận vào value bằng cách dùng cú pháp `let-name="name"`.
Trong đó name ở bên trái dấu bằng là variable bạn có thể access được ở trong ng-template, còn name ở bên phải dấu bằng là tên của object property khi pass qua `ngTemplateOutletContext`. Hai cái name này hoàn toàn có thể khác nhau, không có vấn đề gì cả.

- Nếu bạn không đặt tên cho variable ở vế phải của dấu bằng, chỉ viết là `let-name`, thì khi truyền data qua context, property của object truyền vào sẽ là `$implicit`. Ví dụ cụ thể:

```html
<ng-template #buttonTmpl
             let-label
             let-className="className"
             let-icon="icon">
    <button [ngClass]="['btn', className ? className : '']">
        <i *ngIf="icon"
           class="fa {{icon}}"></i>
        {{ label }}
    </button>
</ng-template>

<ng-container [ngTemplateOutlet]="buttonTmpl"
              [ngTemplateOutletContext]="{ $implicit: 'Remove', className: 'btn-danger', icon: 'fa-remove' }">
</ng-container>
```

- Khi dùng variable ở trong `ng-template`, bạn sẽ mất type safe. Ví dụ bạn pass vào một object user theo cú pháp `let-user="user"` với `firstName`, `lastName` và `age`. Thì trong ng-template, bạn muốn làm gì cái object này cũng được, ví dụ như thử dùng `user.fullName`, compiler sẽ không catch được lỗi, và cả angular language service cũng không báo lỗi trên IDE.

## ng-container

ng-container là một custom html tag để khi render trên UI sẽ không có extra tag để tránh ảnh hưởng đến style mình viết. Như ở ví dụ trên, bạn hoàn toàn có thể viết lại thành.

```html
<div
  [ngTemplateOutlet]="buttonTmpl"
  [ngTemplateOutletContext]="{ label: 'Click here', class: 'btn-primary', icon: null }">
</div>
```

Tuy nhiên khi render ra UI, sẽ bị thừa một thẻ div bao ngoài.

```html
<div>
  <button class="btn btn-primary">
      Click here
  </button>
</div>
```

Nếu bạn có style CSS chặt chẽ theo kiểu `parent > child`. Thì khi thừa một thẻ div, chắc chắn UI sẽ bị ảnh hưởng.

## Summary

Phew, lâu quá không giải thích bằng tiếng Việt nên có thể sẽ không được tường minh như mong muốn. Hy vọng các bạn đã hiểu sơ qua về cách khái niệm `ng-template`, `ng-container` và `ngTemplateOutlet` trong bài viết này.

Một số bài viết khác bạn có thể đọc thêm.

- https://alligator.io/angular/reusable-components-ngtemplateoutlet/
- https://angular.io/guide/structural-directives#the-ng-template
- [Angular render recursive view using *ngFor and ng-template](https://trungk18.com/experience/angular-recursive-view-render/)
- https://blog.angular-university.io/angular-ng-template-ng-container-ngtemplateoutlet/

Mục tiêu của Day 15 là **Giới thiệu Dependency Injection trong Angular**.

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day14`

[day4]: Day004-Structure-Directive-If-Else.md