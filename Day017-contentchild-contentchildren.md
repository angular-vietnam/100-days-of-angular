# Day 17: ContentChild và ContentChildren trong Angular

Trong những ngày qua chúng ta đã tìm hiểu khác nhiều thứ liên quan đến Content Projection. Đối với view, chúng ta có thể query các phần tử trên view với ViewChild hay ViewChildren, vậy đối với content, chúng ta có thể query nó được không? Angular cũng cung cấp cho chúng ta các APIs: `ContentChild` và `ContentChildren` để có thể tương tác được với content truyền vào.

## Query single directive/component

Giả sử chúng ta sẽ sử dụng Tab Component đã được tạo từ Day 16, bây giờ chúng ta có một component Couter để đếm xem liệu chúng ta có bao nhiêu component đã được khởi tạo như sau:

```html
<app-bs-tab-group>
  <app-tab-panel title="Tab 1">
    content tab 1
    <app-counter></app-counter>
  </app-tab-panel>
  <app-tab-panel title="Tab 2">
    content tab 2
    <app-counter></app-counter>
  </app-tab-panel>
  <app-tab-panel title="Tab 3">
    content tab 3
    <app-counter></app-counter>
  </app-tab-panel>
</app-bs-tab-group>
<app-counter></app-counter>
```
Thật bất ngờ, chúng ta hi vọng rằng chỉ có 1 instance của counter, nhưng thực thế chúng ta đang có đến 4 instances, chỉ là có 1 instances được hiển thị.
Vậy nếu trong trường hợp các tabs của chúng ta có những component có phần phức tạp, và chúng ta mong muốn chúng được lazy initialize thì làm thế nào?

Có cách nào để `TabPanelComponent` sẽ nhận vào content, nhưng nó có thể render bất cứ khi nào cần không? Vì rõ ràng với cách sử dụng như ở trên thì cả 4 instances của `app-counter` đều được khởi tạo bên ngoài rồi mới project vào cho `TabPanelComponent`.

Nghĩ đến đây chắc hẳn các bạn sẽ nhớ ra chúng ta chỉ cần truyền vào một `TemplateRef` là được phải không. Vậy làm thế nào để `TabPanelComponent` có thể query một `TemplateRef`?

Đấy là lúc bạn sẽ có thể chọn cho mình giải pháp là sử dụng `ContentChild`. Và đây là typing của nó, khá là giống với `ViewChild`.

```ts
ContentChild(selector: string | Function | Type<any>, opts?: {
  read?: any;
  static?: boolean;
}): any
```

Trước tiên, chúng ta sẽ tạo một directive đã.
```ts
import { Directive } from '@angular/core';

@Directive({
  selector: 'ng-template[tabPanelContent]'
})
export class TabPanelContentDirective {
  constructor() { }
}
```

Directive sẽ giúp chúng ta thêm các tính năng lên một phần tử (DOM Node, Component chẳng hạn), chúng ta có thể thấy directive ở trên muốn target đến bất kỳ thẻ `ng-template` nào có kèm thêm attribute `[tabPanelContent]`.

Và đây là cách sử dụng `ContentChild` để lấy về directive đó.
```ts
export class TabPanelComponent implements OnInit, OnDestroy {
  @Input() title: string;
  @ViewChild(TemplateRef, {static: true}) panelBody: TemplateRef<unknown>;

  @ContentChild(TabPanelContentDirective, {static: true}) explicitBody: TemplateRef<unknown>;

  constructor(private tabGroup: TabGroupComponent) { }

  ngOnInit() {
    this.tabGroup.addTabPanel(this);
  }
  ngOnDestroy() {
    this.tabGroup.removeTabPanel(this);
  }

}
```

Và chúng ta có thể thay đổi cách sử dụng `TabPanelComponent` như sau:
```html
<app-tab-panel title="Tab 1">
  <ng-template tabPanelContent>
    content tab 1
    <app-counter></app-counter>
  </ng-template>
</app-tab-panel>
```

Nhưng nếu bạn đặt debugger (hoặc console.log) thì sẽ thấy nó trả về một instance của `TabPanelContentDirective`, vậy làm thế nào để chúng ta lấy được `TemplateRef` instance?

Lúc này bạn có thể inject `TemplateRef` vào constructor của directive hoặc chúng ta có thể sử dụng cách khác, đó là thay đổi cách `read` một element.

```ts
@ContentChild(TabPanelContentDirective, {static: true, read: TemplateRef}) explicitBody: TemplateRef<unknown>;
```

Đơn giản thế là được rồi.

Bây giờ chúng ta chỉ cần thêm thắt một chút là Tab component sẽ chạy như những gì chúng ta mong muốn.

```ts
export class TabPanelComponent implements OnInit, OnDestroy {
  @Input() title: string;
  @ViewChild(TemplateRef, {static: true}) implicitBody: TemplateRef<unknown>;

  @ContentChild(TabPanelContentDirective, {static: true, read: TemplateRef}) explicitBody: TemplateRef<unknown>;

  get panelBody(): TemplateRef<unknown> {
    return this.explicitBody || this.implicitBody;
  }

}
```

```html
<app-bs-tab-group>
  <app-tab-panel title="Tab 1">
    <ng-template tabPanelContent>
      content tab 1
      <app-counter></app-counter>
    </ng-template>

  </app-tab-panel>
  <app-tab-panel title="Tab 2">
    <ng-template tabPanelContent>
      content tab 2
      <app-counter></app-counter>
    </ng-template>
  </app-tab-panel>
  <app-tab-panel title="Tab 3">
    <ng-template tabPanelContent>
      content tab 3
      <app-counter></app-counter>
    </ng-template>
  </app-tab-panel>
</app-bs-tab-group>

<app-counter></app-counter>
```

Vậy là bây giờ chúng ta chỉ có 2 instances được khởi tạo.

> Lưu ý: với các lazy initialize như trên, mỗi lần chúng ta active một tab nó sẽ tạo lại `TemplateRef` kia một lần.

## Query multiple content with ContentChildren

Quay trở lại với Tab component ở trên, nếu chúng ta không sử dụng Dependency Injection thì chúng ta có thể nào query tất cả các `TabPanelComponent` được project vào không?

Đây chính là lúc mà bạn có thể sử dụng đến `ContentChildren`.

```ts
export class TabGroupComponent implements OnInit {

  @Input() tabActiveIndex = 0;
  @Output() tabActiveChange = new EventEmitter<number>();

  @ContentChildren(TabPanelComponent) tabPanelList: QueryList<TabPanelComponent>

  constructor() { }

  ngOnInit() {
  }

  selectItem(idx: number) {
    this.tabActiveIndex = idx;
    this.tabActiveChange.emit(idx);
  }

}
```

```ts
export class TabPanelComponent {
  @Input() title: string;
  @ViewChild(TemplateRef, {static: true}) implicitBody: TemplateRef<unknown>;

  @ContentChild(TabPanelContentDirective, {static: true, read: TemplateRef}) explicitBody: TemplateRef<unknown>;

  get panelBody(): TemplateRef<unknown> {
    return this.explicitBody || this.implicitBody;
  }

}
```
Chỉ cần có thế là chúng ta đã có thể query được tất cả theo yêu cầu.

> Lưu ý: `ContentChildren` does not retrieve elements or directives that are in other components' templates, since a component's template is always a black box to its ancestors.

## Listen to changes event

`ContentChildren` sẽ được init trước khi lifecycle `ngAfterContentInit` được call, đây cũng là thời điểm mà bạn có thể bắt đầu thực hiện các thao tác với nó. Ví dụ chúng ta có thể listen vào `changes` event để update selected tab như sau:

```ts
export class TabGroupComponent implements OnInit, AfterContentInit {

  @Input() tabActiveIndex = 0;
  @Output() tabActiveChange = new EventEmitter<number>();

  @ContentChildren(TabPanelComponent) tabPanelList: QueryList<TabPanelComponent>

  constructor() { }

  ngOnInit() {
  }

  ngAfterContentInit() {
    this.tabPanelList.changes.subscribe(() => {
      if (this.tabPanelList.length <= this.tabActiveIndex) {
        this.selectItem(0);
      }
    });
  }

  selectItem(idx: number) {
    this.tabActiveIndex = idx;
    this.tabActiveChange.emit(idx);
  }

}
```

## Content vs View

Bây giờ bạn khá băn khoăn, đâu là view đâu là content, vì có quá nhiều thứ gây confuse ở đây.
Câu trả lời cho bạn đây:

- View: là phần template mà component trực tiếp quản lý (thêm, sửa, xóa), nó có thể hiểu là tất cả những gì mà bạn defined cho component đó bên trong `templateUrl` hoặc `template` properties của `@Component` ngoại trừ những gì có trong `ng-content`. View của một component được coi như một black box đối với tất cả các component khác (shadow DOM).

- Content: là phần template được project vào thông qua cặp thẻ mở/đóng của một component/directive. Nó không trực tiếp quản lý. (Nó còn được gọi với tên light DOM).


## Summary

Như vậy, trong Day 17 bạn sẽ cần tìm hiểu sự khác biệt giữa view và content, làm thế nào để query một hoặc một số element được project vào component/directive đó.

Để tìm hiểu sâu hơn, các bạn cần theo dõi thêm một số nguồn sau đây:

- https://www.tiepphan.com/thu-nghiem-voi-angular-content-projection-trong-angular/
- https://www.tiepphan.com/thu-nghiem-voi-angular-thuc-hanh-content-projection-va-lifecycle-angular/
- https://www.tiepphan.com/thu-nghiem-voi-angular-querylist-changes-event-trong-angular/
- https://angular.io/api/core/ContentChild
- https://angular.io/api/core/ContentChildren
- https://netbasal.com/understanding-viewchildren-contentchildren-and-querylist-in-angular-896b0c689f6e

## Code sample
- https://stackblitz.com/edit/angular-ivy-100-days-of-code-day-17?file=src%2Fapp%2Fapp.component.html
- https://stackblitz.com/edit/angular-ivy-100-days-of-code-day-17-contentchildren?file=src%2Fapp%2Ftab-group%2Ftab-group.component.ts

Mục tiêu của Day 18 là **Pipe trong Angular**.

## Author

[Tiep Phan](https://github.com/tieppt)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day17`
