# Day 8: COMPONENT INTERACTION - PARENT LISTENS FOR CHILD EVENT

Thông thường, trong một trang HTML khi có một sự kiện nào đó phát sinh ở một thẻ HTML (ví dụ sự kiện click của thẻ button, submit của form, etc) thì chúng ta sẽ có thể listen ở đâu đó trong code JavaScript.

Vậy với những Component mà chúng ta tự định nghĩa thì có cách nào bắn ra các event mà chungs ta mong muốn hay không (component event). Câu trả lời cho vấn đề này chính là EventEmitter và @Output decorator.

## KHỞI TẠO COMPONENTS

Đầu tiên chúng ta sẽ cần khởi tạo một số component để minh họa như: Author List Component, Author Detail Component:
Các bạn chạy lệnh sau để tạo:

```
ng g c author-list
ng g c author-detail
```

`Author List component` sẽ chứa một danh sách các authors, và nó sẽ truyền vào từng author cho Author Detail hiển thị. Author Detail sẽ cho phép truyền vào input là thông tin của một author:

```typescript
export interface Author {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  gender: string;
  ipAddress: string;
}
```

```typescript
import { Component, OnInit } from "@angular/core";
import { authors } from "../authors";
@Component({
  selector: "app-author-list",
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
  ></app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
}
```

Author Detail Component

```typescript
import { Component, OnInit, Input } from "@angular/core";
import { Author } from "../authors";
@Component({
  selector: "app-author-detail",
  template: `
    <div *ngIf="author">
      <strong>{{ author.firstName }} {{ author.lastName }}</strong>
      <button (click)="handleDelete()">x</button>
    </div>
  `,
  styles: [``],
})
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  constructor() {}
  ngOnInit() {}
  handleDelete() {}
}
```

Bây giờ chúng ta muốn delete author trong Author Detail component thì sao. Điều này chúng ta không nên làm, và có thể delete xong sẽ không work. Vì thực tế thông tin này không phải của Author Detail component, nên nó không được phép edit/modify/remove, mà chúng ta sẽ gửi một event cho parent component để báo rằng chúng ta muốn xóa phần tử này đó.
Lúc này bạn sẽ cần dùng đến EvenEmitter và @Output decorator

```typescript
export class AuthorDetailComponent implements OnInit {
  @Input() author: Author;
  @Output() deleteAuthor = new EventEmitter<Author>();
  constructor() {}
  ngOnInit() {}
  handleDelete() {
    this.deleteAuthor.emit(this.author);
  }
}
```

Đó là cách để chúng ta gửi đi một custom event cho component. Giờ đây ở parent component có thể listen vào event trên và tương tác được với nó.

```typescript
@Component({
  selector: "app-author-list",
  template: `<app-author-detail
    *ngFor="let author of authors"
    [author]="author"
    (deleteAuthor)="handleDelete($event)"
  >
  </app-author-detail>`,
  styles: [``],
})
export class AuthorListComponent implements OnInit {
  authors = authors;
  constructor() {}
  ngOnInit() {}
  handleDelete(author: Author) {
    this.authors = this.authors.filter((item) => item.id !== author.id);
  }
}
```

## SUMMARY

Như vậy trong Day 8, chúng ta sẽ phải tìm hiểu cách để khai báo custom event cho một component, từ đó giúp parent component có thể listen được những event cần thiêt từ child component.
Ngoài những gì trong report trên, chúng ta có thể tìm hiểu thêm về việc tạo alias cho property và outputs array trong bài sau:

- https://www.tiepphan.com/thu-nghiem-voi-angular-2-component-event-voi-eventemitter-output/

Dưới đây là các link document mà các bạn cần tìm hiểu:

- https://angular.io/guide/component-interaction
- https://www.tiepphan.com/thu-nghiem-voi-angular-2-component-event-voi-eventemitter-output/

## Code sample

https://stackblitz.com/edit/angular-ivy-100-days-of-code-day-8?file=src%2Fapp%2Fauthor-detail%2Fauthor-detail.component.ts

Mục tiêu của Day 9 là custom two-way binding
`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day8`
