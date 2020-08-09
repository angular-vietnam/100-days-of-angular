# Day 39: Custom Attribute Directive trong Angular

## Introduction

Qua các bài trước, các bạn cũng biết những directives trong Angular như ngFor, ngIf, ngStyle.
Trước khi bước vào bài mới hôm nay, mình xin tổng quát lại về các directives như sau:

Có 3 loại directives trong Angular:

**Components**: Đây là directive phổ biến nhất, cũng thường gặp nhất trong Angular. Directive này giúp chúng ta đóng gói (encapsulated), và tái sử dụng (reusable). Đây là directive với template, view.

**Structural directives**: Đây là directive dùng để thay đổi cấu trúc của views bằng cách thêm hoặc ẩn, xóa bớt phần tử trong DOM. Ví dụ: ngFor, ngIf.

**Attribute directives**: Đây là directive dùng để thay đổi diện mạo (style, appearance) hoặc hành vi (behavior) của các phần tử DOM hay components. Ví dụ: ngStyle, ngClass

Và bài hôm nay của chúng ta sẽ là tạo 1 **Custom Attribute Directive** để chúng ta có thể tái sử dụng nhiều nơi trong ứng dụng.

## Coding Practice

### Step 1: Khởi tạo project

```sh
ng new custom-attribute-directive-demo
```

### Step 2: Tạo các components và directives

```sh
ng g c example-component
```

```sh
ng generate directive custom-directive-demo
```

Một lưu ý nhỏ ở đây như sau, để sử dụng được directive này chúng ta cần khai báo nó vào module giống như cách chúng ta khai báo component. Câu lệnh generate trên đã auto khai báo directive trên declarations của AppModule cho chúng ta.

Sau đó chúng ta add example-component vào template của app.component.html như sau:

```html
<app-example-container></app-example-container>
```

### Step 3: Code directive logic

Chúng ta sẽ inject **ElementRef** vào constructor của directive như sau:

```typescript
constructor(private el: ElementRef) {
    this.el.nativeElement.innerText = "DEMO TEXT";
    this.el.nativeElement.style.color = "blue";
}
```

Ở đây chúng ta dùng **ElementRef** để tương tác và thay đổi các properties của DOM element. Cụ thể là **innerText** hay **style.color**.

### Step 4: Apply directive

Chúng ta apply attribute directive bằng cách add nó vào như mọi attribute bình thường khác. Tên của attribute này sẽ nằm ở decorator @Directive. (Chúng ta hoàn toàn có thể sửa lại tên chúng ta muốn. Ở đây mình đã sửa từ appCustomDirectiveDemo -> appCustomDirective)

```typescript
@Directive({
  selector: "[appCustomDirective]",
})
```

Chúng ta vào file **example-component.component.html** sửa lại như sau để apply custom directive chúng ta vừa tạo vào component này.

```html
<p appCustomDirective></p>
```

Vậy là chúng ta đã hoàn thành cách tạo và sử dụng 1 custom attribute directive đơn giản.

### Step 5: Using Render2

### Step 6: Interact with event of Host Element

### Step 7: Pass values into the directive

## Concepts

### ElementRef

### Renderer

### HostListener

## Exercies

### 1. Replace component, not Add

### 2. Interact with more view childs

## Summary

Day 39 chúng ta đã học được những concepts và cách làm liên quan đến cách **Custom attribute directive**.

Mục tiêu của ngày 40 sẽ là **Custom structural directive**.

## Code sample

- https://github.com/januaryofmine/Dynamic-Component-Demo

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/guide/attribute-directives#build-a-simple-attribute-directive
- https://medium.com/@nishu0505/custom-directives-in-angular-5a843f76bb96
- https://angular.io/api/core/ElementRef

## Author

[Khanh Tiet](https://github.com/januaryofmine)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day39`

[day34]: Day034-template-driven-forms-2.md
[day38]: Day038-dynamic-component.md
