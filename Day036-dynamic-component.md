# Day 36: Dynamic Component trong Angular

## Introduction

Qua các bài trước, các bạn cũng biết cách tạo những components cha con cũng như cách tương tác giữa chúng rồi.
Vậy có 1 trường hợp thế này. Ta có component A là cha của component B như sau:

![ParentComponent](assets/day-36-dynamic-component-01.png)

Trong nhiều trường hợp, chúng ta muốn thay đổi trong lúc runtime, ở vị trí đó không phải chỉ fix cứng 1 component B như vậy. Có lúc sẽ là component B, có lúc sẽ là component C tùy logic của ứng dụng.
Hay ở tình huống khác, chúng ta muốn người dùng phải làm gì đó ở component A thì mới load component B lên. Nếu code bình thường, component B luôn được fix cứng trong template là con của A.

Vậy việc load động 1 component khác trong lúc runtime được thực hiện như thế nào?
Điều đó dẫn ta đến bài hôm nay, **Dynamic Component** sẽ là câu trả lời phù hợp để làm việc này.

## Coding Practice

### Step 1: Khởi tạo project

```sh
ng new dynamic-component-demo
```

### Step 2: Tạo các components

```sh
ng g c example-container
```

```sh
ng g c dynamic-content-one
```

```sh
ng g c dynamic-content-two
```

Sau đó chúng ta add example-container vào template của app.component.html như sau:

```html
<app-example-container></app-example-container>
```

### Step 3: Code container components

Chúng ta khởi tạo template example-component với 2 nút và 1 **ViewChild** như sau.

```html
<button (click)="addDynamicCompOne()" class="btn">
  Add Dynamic Component 1
</button>
<button (click)="addDynamicCompTwo()" class="btn">
  Add Dynamic Component 2
</button>

<div #dynamicComponent></div>
```

```typescript
import {
  Component,
  OnInit,
  ViewChild,
  ViewContainerRef,
  ComponentFactoryResolver,
} from "@angular/core";
import { DynamicContentOneComponent } from "../dynamic-content-one/dynamic-content-one.component";
import { DynamicContentTwoComponent } from "../dynamic-content-two/dynamic-content-two.component";

@Component({
  selector: "app-example-container",
  templateUrl: "./example-container.component.html",
  styleUrls: ["./example-container.component.scss"],
})
export class ExampleContainerComponent implements OnInit {
  @ViewChild("dynamicComponent", { read: ViewContainerRef, static: true })
  containerRef: ViewContainerRef;

  constructor(private cfr: ComponentFactoryResolver) {}

  ngOnInit() {}

  addDynamicCompOne() {
    const componentFactory = this.cfr.resolveComponentFactory(
      DynamicContentOneComponent
    );
    const componentRef = this.containerRef.createComponent(componentFactory);
  }

  addDynamicCompTwo() {
    const componentFactory = this.cfr.resolveComponentFactory(
      DynamicContentTwoComponent
    );
    const componentRef = this.containerRef.createComponent(componentFactory);
  }
}
```

Flow chính:

1. Tạo 1 ViewChild trong template. Ở đây là thẻ div **#dynamicComponent**. Đây sẽ là nơi chúng ta load những components vào ở runtime.
2. Connect **#dynamicComponent** thông qua @ViewChild. Chúng ta sẽ có 1 [ViewContainerRef](###ViewContainerRef)
3. Inject [CompanyFactoryResolver](###ComponentFactoryResolver) của Angular vào component ExampleContainerComponent.
4. Dùng Resolver connect với component nào chúng ta muốn load dynamic.
   => Kết quả sẽ trả về 1 [Component Factory](###ComponentFactory)

```typescript
const componentFactory = this.cfr.resolveComponentFactory(
  DynamicContentOneComponent
);
```

Dùng **ViewContainerRef** với Component Factory chúng ta vừa tạo ở trên để load **Dynamic Component**.

```typescript
const componentRef = this.containerRef.createComponent(componentFactory);
```

### Step 4: Add các dynamic components vào entryComponents

Để code trên hoạt động được, các bạn cần add 2 components DynamicContentOne và DynamicContentTwo vào **entryComponents** như sau. Nếu không sẽ xảy ra lỗi "No component factory found ... "

```typescript

@NgModule({
  declarations: [
    AppComponent,
    ExampleContainerComponent,
    DynamicContentOneComponent,
    DynamicContentTwoComponent,
  ],
  imports: [BrowserModule],
  providers: [],
  bootstrap: [AppComponent],
  entryComponents: [DynamicContentOneComponent, DynamicContentTwoComponent],
})
```

Ngoài ra từ khi có Angular Ivy, chúng ta có thể bỏ đi step này.

### Step 5: Clear các dynamic components

Tiếp đến chúng ta thực hiện 1 tính năng là clear các components đã load dynamic.
Chúng ta làm điều này như sau:

```html
<button (click)="clearDynamicComp()" class="btn">Clear</button>
```

```typescript
clearDynamicComp() {
    this.containerRef.clear();
  }
```

### Step 6: Tương tác với các dynamic components

Chúng ta tương tác giữa containers và các dynamic components cũng tương tự như cách tương tác giữa component cha với con. Cụ thể như sau:

Ở component con, chúng ta tạo 1 @Input như sau:

```typescript
@Input()
  data: string;
```

```html
<h1>DYNAMIC CONTENT 1</h1>
<p>++++++{{data}}+++++++++</p>
```

Ở component cha, chúng ta sẽ truyền data thông qua componentRef (Đây là kết quả trả về sau khi chúng ta dùng **ViewContainerRef** load dynamic component)

```typescript
addDynamicCompOne() {
    const componentFactory = this.cfr.resolveComponentFactory(
      DynamicContentOneComponent
    );
    const componentRef = this.containerRef.createComponent(componentFactory);
    componentRef.instance.data = "INPUT DATA 1";
  }
```

### Step 7: Update with Angular Ivy Lazy Load

Hiện tại code sử dụng entryComponents đã cũ và với **Angular Ivy**, chúng ta hoàn toàn không cần sử dụng nữa. Ngoài ra chúng ta có thể sử dụng **Angular Ivy** để **lazy load** các components dynamic.
Code sẽ sửa như sau:

### Step 7.1: Xóa entryComponents setting in app.module.ts

Các bạn hãy vào file app.module.ts xóa đi config đã set ở step 4.

### Step 7.2: Update code ở container component

- Remove 2 cái import components ở đầu file.
- Sửa 2 hàm addDynamicComp

Code sẽ như sau:

```typescript
  async addDynamicCompOne() {
    const { DynamicContentOneComponent } = await import('../dynamic-content-one/dynamic-content-one.component');
    const componentFactory = this.cfr.resolveComponentFactory(
      DynamicContentOneComponent
    );
    const componentRef = this.containerRef.createComponent(componentFactory);
    componentRef.instance.data = "INPUT DATA 1";
  }

  async addDynamicCompTwo() {
    const { DynamicContentTwoComponent } = await import('../dynamic-content-two/dynamic-content-two.component');
    const componentFactory = this.cfr.resolveComponentFactory(
      DynamicContentTwoComponent
    );
    const componentRef = this.containerRef.createComponent(componentFactory);
    componentRef.instance.data = "INPUT DATA 2";
  }
```
Vậy là đã xong, các bạn đã thực hiện thành công việc lazy load các dynamic components mà không phải add trực tiếp vào như ở những step đầu.
**Lưu ý**: Đối với những bạn nào dùng Angular phiên bản cũ thì nhớ update angular để sử dụng tính năng Angular Ivy. 

## Concepts

### ViewContainerRef

Nó là một cái container từ đó có thể tạo ra Host View (component khi được khởi tạo sẽ tạo ra view tương ứng), và Embedded View (được tạo từ TemplateRef). Với các view được tạo đó sẽ có nơi để gắn vào (container).

Container có thể chứa các container khác (ng-container chẳng hạn) tạo nên cấu trúc cây. Hay hiểu đơn giản thì nó giống như 1 DOM Element, khi đó có thể add thêm các view khác (Component, Template) vào đó.
![TiepPhan](https://www.tiepphan.com/angular-trong-5-phut-dynamic-component-rendering/)

### ComponentFactory

Đây là 1 class dùng để tạo ra các components dynamic. Là kết quả trả về của **ComponentFactoryResolver.resolveComponentFactory()**.

### ComponentFactoryResolver

Đây là 1 class nhận vào các component để load dynamic và tạo ra 1 component factory của component đó. ViewContainerRef sẽ dùng **ComponentFactory** đó để load dynamic các components.

## Exercies

### 1. Replace component, not Add

Hiện tại project đang là add các dynamic components vào cùng 1 ViewChild 1 cách liên tục. Thay vì vậy, các bạn hãy tạo tính năng thay thế. Ví dụ bấm nút A thì hiện component A, bấm nút B thì hiện component B thay thế cho component A

### 2. Interact with more view childs

Hiện tại project chỉ load các dynamic components vào cùng 1 ViewChild.
Các bạn thử tương tác tạo ViewChilds hơn và load các dynamic components vào đó.
Cũng như thử emit event từ ViewChild và nhận, xử lý sự kiện đó ở component cha.

## Summary

Day 36 chúng ta đã học được những concepts liên quan đến Dynamic Component. Đây là 1 tính năng quan trọng có tính ứng dụng cao. Các bạn có thể thực hành nhiều hơn thông qua các bài tập mình đưa cũng như các nguồn tài liệu mình để dưới đây.

Mục tiêu của ngày 37 sẽ là

## Code sample

- https://github.com/januaryofmine/Dynamic-Component-Demo

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/guide/dynamic-component-loader
- https://www.tiepphan.com/angular-trong-5-phut-dynamic-component-rendering/
- https://stackblitz.com/edit/angular-dynamic-components-example
- https://www.youtube.com/watch?v=dZD7pw6rmRA

## Author

[Khanh Tiet](https://github.com/januaryofmine)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day36`

[day34]: Day034-template-driven-forms-2.md
[day35]: Day035-reactive-forms.md
