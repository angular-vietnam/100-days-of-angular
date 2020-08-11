# Day 40: Custom Structural Directive trong Angular

## Introduction

Ở bài trước, chúng ta đã biết cách tạo và sử dụng 1 **Custom Attribute Directive**. Ở bài này, chúng ta sẽ tiếp tục tiếp hiểu cách tạo và sử dụng 1 **Custom Structural Directive**.

Đây là project của bài hôm nay

![Custom structural directive](assets/day040.gif)

## Coding Practice

### Step 1: Khởi tạo project

```sh
ng new custom-structural-directive-demo
```

### Step 2: Tạo các components và directives

```sh
ng g c example-container
```

```sh
ng g d custom-loop
```

Sau đó chúng ta add example-container vào template của app.component.html như sau:

```html
<app-example-container></app-example-container>
```

### Step 3: Code directive logic and apply to component

Đầu tiên chúng ta sẽ tạo 1 ngFor cơ bản như sau.

Các bạn hãy add FormsModule vào app.module. (Vì chúng ta sẽ sử dụng ngModel cho project này).

Các bạn add đoạn code này vào file Custom Loop directive vừa được generate ở trên.

```typescript
import { Directive, Input, ViewContainerRef, TemplateRef } from "@angular/core";

@Directive({
  selector: "[ngCustomLoop]",
})
export class CustomLoopDirective {
  @Input("ngCustomLoopOf") itemList: Array<any>;

  constructor(
    private containerRef: ViewContainerRef,
    private template: TemplateRef<any>
  ) {}

  ngOnChanges() {
    for (const item of itemList) {
      this.containerRef.createEmbeddedView(this.template, {
        $implicit: item,
        index: this.itemList.indexOf(item),
      });
    }
  }
}
```

Các bạn add đoạn code này vào component Example Container.

```html
<input type="text" [(ngModel)]="text" />
<p *ngCustomLoop="let city of cityList; let i = index">
  {{ i + 1 }}. {{ city }}
</p>
```

```typescript
export class ExampleContainerComponent implements OnInit {
  text = "";
  cityList = ["Ha Noi", "Ho Chi Minh", "Da Nang", "Nha Trang", "Vinh Long"];

  constructor() {}

  ngOnInit() {}
}
```

Giờ các bạn chạy project thử và xem kết quả. Các bạn đã thấy được

Core ideas need to explain

1. Context Object / \$implicit
2. this.containerRef.clear();
3. Template Directives Micro Syntax
4. @Input alias / Add more input
5. TemplateRef/ createEmbeddedView

### Step 4: Add more input for directive

### Step 5: Implement Filter features

Vậy là đã xong, các bạn đã thực hiện thành công việc tạo và sử dụng 1 **Custom Structural Directive** trong Angular.

## Concepts

### TemplateRef

## Exercise

### 1.

### 2.

## Summary

Day 40 chúng ta đã học được những concepts và cách làm liên quan đến cách **Custom structural directive**.

Mục tiêu của ngày 41 sẽ là những lệnh quen thuộc và các flag thường gặp trong **Angular CLI**.

## Code sample

- https://github.com/januaryofmine/angular-custom-structural-directive

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/guide/structural-directives#writing-your-own-structural-directives
- https://medium.com/@kay.odenthal_25114/creating-a-custom-structural-directive-with-angular-7-3e85bcf88bdf
- https://netbasal.com/understanding-angular-structural-directives-659acd0f67e
- https://blog.cloudboost.io/creating-structural-directives-in-angular-ff17211c7b28

## Author

[Khanh Tiet](https://github.com/januaryofmine)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day40`

[day38]: Day038-dynamic-component.md
[day39]: Day039-custom-attribute-directive.md
