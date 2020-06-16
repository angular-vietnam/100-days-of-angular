# Day 4: ANGULAR STRUCTURE DIRECTIVE - NGIF

Theo như một lẽ tự nhiên trong lập trình, có những lúc chúng ta cần phụ thuộc vào điều kiện gì đó để đưa ra quyết định tương ứng. Giả sử chúng ta đang xây dựng ứng dụng xem video trực tuyển, lúc này có những bộ phim PG-13 yêu cầu người xem phải từ 13 tuổi trở lên mới xem được. Làm thế nào để hiển thị cho người dùng biết được họ có đủ điều kiện để xem video đó hay không? Lúc này chúng ta có thể dùng cấu trúc IF-ELSE mà Angular cung cấp để đáp ứng yêu cầu đó.
Trong Angular để thêm, xóa, thay đổi structure (structure HTML chẳng hạn) ở trên view của component chúng ta sẽ dùng Structure Directive.

## CẤU TRÚC IF-ELSE

Để hiển thị một phần view (template) theo một điều kiện, chúng ta sẽ gắn thêm một property đặc biệt vào một tag, với cú pháp có chứa dấu `* (asterisk)` như sau `*ngIf="expression"`:

```typescript
@Component({
  selector: "app-hello",
  template: `
    <h2>Hello there!</h2>
    <h3>Your name: {{ user.name }}</h3>
    <p>Your name: {{ user.age }}</p>
    <div *ngIf="user.age >= 13">
      Bạn có thể xem nội dung PG-13
    </div>
  `,
})
export class HelloComponent {
  user = {
    name: "Tiep Phan",
    age: 30,
  };
}
```

Chỉ cần có thế là chúng ta có thể hiển thị view tùy thuộc vào dữ liệu mà expression trả về. Truthy thì hiển thị, Falsy thì không hiển thị.
Với những directive được cung cấp sẵn (built-in) bởi Angular, giờ đây HTML template của component sẽ rất flexible.
Vậy nếu chúng ta muốn dùng IF-ELSE thì thế nào. Có thể các bạn sẽ nghĩ ngay đến phủ định mệnh đề IF là được ELSE thôi. Điều này hoàn toàn được.

```html
<div *ngIf="user.age >= 13">
  Bạn có thể xem nội dung PG-13
</div>
<div *ngIf="user.age < 13">
  Bạn không thể xem nội dung PG-13
</div>
```

Hoặc chúng ta có cách hay ho khác, đó là dùng đến ng-template. Tag ng-template là một tag được cung cấp bởi Angular, nó sẽ lưu trữ một Template được định nghĩa bên trong cặp thẻ mở/đóng của nó. Những gì được định nghĩa bên trong đó sẽ không được hiển thị ra view, nhưng chúng ta có thể sử dụng Template đó để render bằng code. Đoạn code phía trên có thể được chuyển đổi tương đương:

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

## NG-TEMPLATE

Với cú pháp sử dụng dấu `*` ở trên, có thể các bạn sẽ thấy nó khác lạ, nhưng thực tế, nó được gọi là Syntactic sugar (giúp nhìn code dễ hiểu, dễ đọc hơn chẳng hạn) được chuyển đổi sang dạng property binding như sau:

```html
<ng-template [ngIf]="user.age >= 13" [ngIfElse]="noPG13">
  <div>
    Bạn có thể xem nội dung PG-13
  </div>
</ng-template>
```

## SUMMARY

Trong ngày thứ 4, chúng ta cần hiểu cách dùng cấu trúc ngIf-else, ngoài các cách sử dụng ở trên Angular còn cung cấp cách dùng ngIf-then-else nữa, các bạn có thể tìm hiểu thêm tại link tham khảo phía dưới.
Link document các bạn cần tìm hiểu trong Day 4

- https://angular.io/guide/structural-directives
- https://angular.io/api/common/NgIf

Mục tiêu Day 5 sẽ là về cấu trúc lặp `ngForOf`

## Author

[Tiep Phan](https://github.com/tieppt)

## HASHTAG

`#100DaysOfCodeAngular` `#100DaysOfCode #AngularVietNam100DoC_Day4`
