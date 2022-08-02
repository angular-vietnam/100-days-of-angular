# Day 5: ANGULAR STRUCTURE DIRECTIVE NGFOROF

Sẽ ra sao nếu trong ứng dụng bạn cần phải hiển thị một danh sách các phần tử ra ngoài template? Nếu danh sách đó chúng ta có được là một array thì có cách nào thực hiện vòng lặp ở template không? Nếu chúng ta cần sử dụng for loop như trong code TypeScript thì sao?
Câu trả lời cho những câu hỏi trên chính là NgForOf trong Angular. (Một số bạn có thể gọi nó là `NgFor`).
NgForOf có cách dùng như thế nào, có dùng giống cú pháp của `NgIf` hôm trước không?
Giả sử chúng ta có một danh sách các tác giả của các cuốn sách trong một ứng dụng bán sách/thư viện có cấu trúc như sau:

```typescript
authors = [
  {
    id: 1,
    firstName: 'Flora',
    lastName: 'Twell',
    email: 'ftwell0@phoca.cz',
    gender: 'Female',
    ipAddress: '99.180.237.33',
  },
  {
    id: 2,
    firstName: 'Priscella',
    lastName: 'Signe',
    email: 'psigne1@berkeley.edu',
    gender: 'Female',
    ipAddress: '183.243.228.65',
  },
  // more data
];
```

Dưới đây là ví dụ về cách sử dụng NgForOf:

```html
<div *ngFor="let author of authors">
  {{author.id}} - {{author.firstName}} {{author.lastName}}
</div>
```

Cấu trúc sử dụng này khá giống for (let author of authors) trong TypeScript đúng không.

## MỘT SỐ LOCAL VARIABLE TRONG MỘT NGFOROF TEMPLATE

Khi sử dụng NgForOf, ở mỗi vòng lặp chúng ta sẽ có thể truy xuất đến một số local variable như:

| Term           | Description                                            |
| -------------- | ------------------------------------------------------ |
| \$implicit: T  | Giá trị của phần tử trong danh sách ở lần lặp hiện tại |
| index: number  | index của lần lặp hiện tại                             |
| count: number  | số lượng phần tử trong danh sách                       |
| first: boolean | True nếu đây là phần tử đầu tiên trong danh sách       |
| last: boolean  | True nếu đây là phần tử cuối cùng trong danh sách      |
| even: boolean  | True nếu đây là phần tử ở index chẵn                   |
| odd: boolean   | True nếu đây là phần tử ở index lẻ                     |

Để truy xuất được những biến trên chúng ta chỉ cần gọi chúng như sau:
Đối với \$implicit nó sẽ được gắn cho biến mà chính chúng ta khi khai báo let something of xxx, lúc này something = `\$implicit`.
Đối với các biến khác, chúng ta chỉ cần dùng cú pháp sau:

```html
<div *ngFor="let author of authors; index as idx; count as total">
  ({{idx}})/({{total}}): {{author.id}} - {{author.firstName}}
  {{author.lastName}}
</div>
```

Lúc này biến `idx = index`, và `total = count`. Tương tự đối với các biến còn lại.

## CẤU TRÚC NGFOROF VÀ NG-TEMPLATE

Với cấu trúc dùng dấu sao (\*) như trên chúng ta có thể chuyển đổi tương ứng về dạng ng-template và property binding như sau:

```html
<ng-template
  ngFor
  [ngForOf]="authors"
  let-author
  let-idx="index"
  let-total="count"
>
  <div>
    ({{idx}})/({{total}}): {{author.id}} - {{author.firstName}}
    {{author.lastName}}
  </div>
</ng-template>
```

## SỬ DỤNG NHIỀU STRUCUTRE DIRECTIVE TRÊN CÙNG MỘT PHẦN TỬ

Trong nhiều trường hợp, bạn có thể cần kiểm tra một dữ liệu nào đó trong vòng lặp, nếu bạn đặt NgIf và NgForOf trên cùng một phần tử thì nó sẽ không work. Liệu có cách nào khác không?
Câu trả lời đó là bạn không thể đặt nhiều hơn một structure directive lên một phần tử được. Hãy thử xem ví dụ sau:

```typescript
for (let item of list) {
  if (somethingGood) {
    // more code
  }
}
```

Rõ ràng là chúng ta không đặt for và if vào cùng một hàng được. Vậy nên giải pháp lúc này là tách chúng ra, và sử dụng một tầng wrapper nữa.
Ví dụ:

```html
<div *ngFor="”let" item of list”>
  <div *ngIf="”somethingGoood”">More code</div>
</div>
```

Giả sử nếu bạn không được phép hoặc không muốn sinh ra một div thừa thì sao? Chúng ta có thể convert NgIf về dạng ng-template như buổi trước hoặc dùng ng-container để dùng:

```html
<div *ngFor="”let" item of list”>
  <ng-container *ngIf="”somethingGoood”"> More code </ng-container>
</div>
<div *ngFor="”let" item of list”>
  <ng-template [ngIf]="”somethingGoood”"> More code </ng-template>
</div>
```

## SUMMARY

Trong ngày thứ 5, chúng ta cần hiểu cách dùng cấu trúc NgForOf và một số lưu ý khi sử dụng các local variable. Thêm nữa chúng ta cũng cần tìm hiểu cách để sử dụng các cấu trúc lồng nhau với nhiều Structure directive
Link document các bạn cần tìm hiểu trong Day 5

- https://angular.io/guide/structural-directives
- https://angular.io/api/common/NgForOf
- https://www.tiepphan.com/thu-nghiem-voi-angular-2-built-in-directives-ngif-ngfor-ngswitchcase/
- https://www.tiepphan.com/thu-nghiem-voi-angular-2-ngfor-index-first-last-even-odd-trackby/
- https://www.youtube.com/watch?v=dXDC-4KGIGI

Mục tiêu Day 6 sẽ là về `Attribute directive` để áp dụng style, class binding.

## Youtube Video

[![Day 05](https://img.youtube.com/vi/q7CQPEPSkD0/0.jpg)](https://youtu.be/q7CQPEPSkD0)

## Author

[Tiep Phan](https://github.com/tieppt)

## HASHTAG

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day5`
