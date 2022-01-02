# Day 27: Angular Router

Trước đây khi các ứng dụng web vẫn chủ yếu follow theo mô hình server side rendering. Tức là khi bạn mở một website, phía server sẽ gửi cho bạn toàn bộ page đó để render. Khi bạn chuyển trang, ví dụ như từ trang chủ của một website mua bán trực tuyến, bạn click vào một đường dẫn để xem phần thông tin các sản phẩm về giày dép. Phần server sẽ gửi lại toàn bộ HTML của page đó, bao gồm từ thẻ <html>, <head> và các thẻ script, cho đến phần nội dung cần được hiển thị. Điều này dẫn đến việc với mỗi click lên website, cả website sẽ được reload với phần nội dung mới. Thuật ngữ đó là refresh, hay postback trong một số ngôn ngữ.

![Postback][01]

Với mỗi lần có biểu tượng hình xoay xoay spinner, tức là đã có một request lên server để load lại toàn bộ view của website

Cách này đã được dùng cho đến mãi gần đây khi concept về Single Page Application được ra đời. Trong ứng dụng SPA, thì phần template của ứng dụng được package hoàn toàn trong cái file JS. Và với mỗi click trong ứng dụng SPA, đa số sẽ gọi một HTTP request lên server thông qua API để lấy data và update lên phần nội dung được hiển thị. Phần nằm dưới của kĩ thuật này có tên là AJAX, hẳn các bạn đã nghe qua. Nếu là ứng dụng dạng SPA như những application được viết bằng Angular, mỗi khi bạn tương tác với ứng dụng thì gần như bạn sẽ không cảm nhận được là việc tương tác giữa webpage với server như thế nào. Cụ thể là spinner ở trên tab của trình duyệt sẽ ko được hiển thị mỗi lần bạn tương tác.

Vậy làm sao ứng dụng Angular có thể biết được là bạn đang cần truy cập vào phần thông tin nào, và cách hoạt động ra sao. Đó là nhờ [Angular Router][router].

Khi người dùng thực hiện các tác vụ ứng dụng, họ cần di chuyển giữa các view khác nhau mà developer đã config. Ví dụ khi bạn vào `tiepphan.com`, bạn sẽ thấy một danh sách các bài viết. Bạn click vô một bài rất hay, muốn gửi cho bạn bè. Thì bạn sẽ copy cái đường link có dạng `tiepphan.com/bai-nay-hay-lam` và gửi cho bạn của mình.

Về phía ứng dụng, khi bạn mở đường dẫn `tiepphan.com/bai-nay-hay-lam`, application phải hiển thị được đúng bài viết mà bạn đã xem.

- Ứng dụng Angular cần phải được config để bạn có thể mở được đường dẫn có dạng `tiepphan.com/bai-nay-hay-lam`
- Khi mở đường dẫn có dạng như vậy, Angular cần phải hiển thị một layout của một bài báo. Chứ ko thể hiển thị danh sách các bài viết như ở trang chủ được
- Bài báo đó phải hiện thị nội dung mà bạn đã xem, chứ ko thể là một bài viết ngẫu nhiên được.

Chúng ta cùng xem cách làm như ở dưới nhé.

## Output cuối cùng

Sau khi follow hướng dẫn này, phần ứng dụng hoàn thành sẽ như ở ảnh dưới.

![Step 6][06]

Yêu cầu của ứng dụng này

- Mở ứng dụng, hiển thị danh sách các bài viết
- Khi click vô mỗi bài viết, sẽ hiển thị bài viết chi tiết
- Copy đường link của bài viết chi tiết, mở trên trình duyệt khác vẫn được
- Nếu đường dẫn không đúng, báo lỗi cho người dùng.

## Trước khi bắt tay vào làm ứng dụng với routing

Trước khi bắt đầu, các bạn có một số kiến thức nền tảng về Angular để có thể dễ dàng tiếp cận với những gì được đề cập trong hướng dẫn.

- Component
- Template
- Sử dụng Angular CLI

## Khởi tạo một ứng dụng có hỗ trợ Angular Router

Để tạo một ứng dụng có hỗ trợ router với CLI, các bạn chỉ cần chạy câu lệnh dưới đây

```bash
ng new day27-routing --routing
```

Bạn cần chú ý hai thứ

- `day27-routing`: tên của ứng dụng bạn sẽ tạo
- `--routing`: phần mô tả rằng ứng dụng sẽ được đi kèm với Router

Ngoài ra còn một số câu hỏi CLI sẽ hỏi bạn, như là có muốn dùng loại styling gì, các bạn chọn SCSS hay CSS hay tùy vào sở thích. Mình sẽ chọn SCSS.

![Step 2][02]

### Thêm components để thực hiện cho việc điều hướng (routing)

Như đã nói ở trên, mình sẽ cần một component để hiện thị danh sách các bài viết, và một component để hiển thị một bài viết chi tiết.

Để tạo mới một component với CLI, bạn chạy câu lệnh ở dưới trong đó `article-list` là tên của component.

Đây là component article-list

```bash
ng generate component article-list
```

Còn đây là article-detail

```bash
ng generate component article-detail
```

![Step 3][03]

### Config router cơ bản

Khi tạo ứng dụng mới theo step ở trên thì CLI đã mặc định tạo ra một module với tên gọi `AppRoutingModule` và tự động import vào `AppModule` cho chúng ta.

```ts
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule // Đây chính là AppRoutingModule được tạo tự động bằng CLI
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Có ba thành phần chính khi làm việc với Router

1. Bạn cần import `RouterModule` and `Routes` vào trong router module của bạn, ở đây, chính là `AppRoutingModule`

```ts
const routes: Routes = [];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

Chú ý là `AppRoutingModule` vừa import và export `RouterModule`. Điều này có nghĩa là khi bạn import `AppRoutingModule` vào các module khác, bạn ko cần import lại `RouterModule` để sử dụng nữa vì nó đã được đc re-export từ `AppRoutingModule`.

RouterModule mặc định sẽ provide hai method là `forRoot` và `forChild`. Hai method này đều dùng để config routes, tuy nhiên.
- `forRoot`, dc gọi <u>một lần duy nhất</u> khi bạn config route trong `AppRoutingModule`. forRoot cũng dùng để configures/initializes router.
- `forChild`, dc gọi trong các module khác để config routes.

Việc dùng forRoot còn liên quan đến khởi tạo service internal của RouterModule. Như trong [Day 15][day15] anh Tiệp có nhắc đến việc sử dụng DI trong Angular. Gọi forRoot một lần duy nhất để đảm bảo rằng các service của RouterModule được khởi tạo và chỉ có một instance duy nhất. Nếu bạn gọi forRoot nhiều lần trong các module khác nhau, có thể dẫn đến các behavior không đoán được khi dùng Router.

Các bạn có thể đọc thêm ở [stackoverflow](https://stackoverflow.com/a/44680396/3375906)

2. Config routes trong mảng `Routes`

Với yêu cầu của mình và hai component vừa được tạo, thì cấu hình sẽ như ở dưới

```ts
const routes: Routes = [
  {
    path: "detail",
    component: ArticleDetailComponent,
  },
  {
    path: "",
    component: ArticleListComponent,
  },
];
```

Mỗi object `Route` có hai thành phần chính quan trọng là `path` và `component`. Ở đây với định nghĩa như ở trên, thì khi bạn mở ứng dụng, mặc định sẽ load component `ArticleListComponent`, và nếu bạn mở đường link `/detail` thì router sẽ load component `ArticleDetailComponent` tương ứng.

3. Sử dụng route trong ứng dụng

Sau khi đã config ở bước hai, thì giờ mình có thể sử dụng router được rồi. Dùng `routerLink` trong thẻ `<a>` để gán phần URL như đã config và `<router-outlet>` sẽ là nơi phần component được load tương ứng khi mở URL tương ứng.

```html
<ul class="nav nav-pills card-header-pills">
  <li class="nav-item">
    <a class="nav-link" routerLink="/">Home</a>
  </li>
  <li class="nav-item">
    <a class="nav-link" routerLink="detail">Detail</a>
  </li>
</ul>
<router-outlet></router-outlet>
```

Chạy thử thì thấy router đã hoạt động như kì vọng.

![Step 4][04]

### Thứ tự route khi config

Để ý khi mình config route ở step 2, mình đặt `detail` ở trên phần route trống. Đó là vì thứ tự trong mảng Routes là quan trọng và ảnh hưởng đến việc load route. Route nào càng chi tiết thì nên được config trước những route ít chi tiết hơn. Ví dụ

`/detail/123/edit` - để show edit form cho item id 123
`/detail/123` - để show thông tin chi tiết cho item id 123

Thì phần config cho route edit nên được đẩy lên trước. Những phần nào generic thì nên để phía sau những phần detail hơn.

## Lấy thông tin từ route

Ví dụ trên đã hoạt động nhưng một ứng dụng thực tế cần nhiều hơn thế. Mỗi khi navigate qua lại giữa các route, thông thường một số loại data sẽ được trao đổi và nơi dễ dàng nhất để làm điều này chính là trên URL. Cụ thể là khi mình mở một article, mình cần id hoặc slug (kiểu thân thiện hơn id) ở trên URL để có thể dựa vào id hoặc slug đó để lấy thông tin của bản ghi chi tiết.

Bây giờ mình sẽ render ra một danh sách các bài viết dựa vào data.

```ts
const Articles: Article[] = [
  {
    id: "1",
    slug: "bai-viet-1",
    title: "Bai viet 1",
    content: "Day la noi dung bai viet 1",
    updateAt: "2020-07-06T13:26:31.785Z",
  },
  {
    id: "2",
    slug: "bai-viet-2",
    title: "Bai viet 2",
    content: "Day la noi dung bai viet 2 nhe",
    updateAt: "2020-07-15:00:00.000Z",
  },
];
@Injectable({
  providedIn: "root",
})
export class ArticleService {
  getArticles(): Observable<Article[]> {
    return of(Articles).pipe(delay(500));
  }
}
```

Và trong component list, mình sẽ render ra dựa vào data ở trên.

```ts
export class ArticleListComponent implements OnInit {
  articles$: Observable<Article[]>;
  constructor(private _api: ArticleService) {}

  ngOnInit(): void {
    this.articles$ = this._api.getArticles();
  }
}
```

```html
<div class="row" *ngIf="articles$ | async as articles">
  <div class="col-md-3" *ngFor="let article of articles">
    <div class="card text-center">
      <div class="card-header">
        {{ article.title }}
      </div>
      <div class="card-body">
        <p class="card-text">{{ article.content }}</p>
        <a [routerLink]="article.slug" class="btn btn-primary">
          Xem {{ article.title }}
        </a>
      </div>
    </div>
  </div>
</div>
```

Kết quả sẽ trông như thế này. Nhưng khi bấm vào hai cái button thì chưa hoạt động đâu nhé.

![Step 5][05]

Giờ phần việc còn lại là config để route detail có thể nhận dc slug khi mình truyền qua URL, các bạn sửa lại phần Router config như này nhé.

```ts
const routes: Routes = [
  {
    path: ":slug",
    component: ArticleDetailComponent,
  },
  {
    path: "",
    component: ArticleListComponent,
  },
];
```

Thay vì `path: 'detail'`, giờ mình sửa lại thành `path: ':slug'`. Dấu hai chấm là cú pháp của router cho phép bạn định nghĩa ra một parameter trên URL. Phần sau dấu hai chấm là tên của parameter mà bạn có thể lấy được từ trong `ArticleDetailComponent`

Ở trong `ArticleDetailComponent`, để lấy được slug từ URL. Mình inject `ActivatedRoute` vào và lấy thông tin của params tên slug từ route snapshoot.

```ts
export class ArticleDetailComponent implements OnInit {
  article$: Observable<Article>;
  constructor(private _route: ActivatedRoute, private _api: ArticleService) {}

  ngOnInit(): void {
    let slug = this._route.snapshot.paramMap.get("slug");
    this.article$ = this._api.getArticleBySlug(slug);
  }
}
```

Xong rồi đấy, giờ xem thành quả nhé. Bạn thấy không, ngay cả khi reload lại page thì `ArticleDetailComponent` vẫn lấy được thông tin về slug và dựa vào đó để lấy data rồi hiển thị thông tin như mình mong muốn.

Phần xử lý lỗi khi không tìm thấy bài viết chúng ta sẽ cùng tìm hiểu ở các bài viết sau.

![Step 6][06]

## Summary

Hy vọng các bạn nắm được qua bài viết này

- Cách config router
- Cách lấy data từ route
- Thứ tự sắp xếp của route khi config là quan trọng.

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/guide/router
- https://www.tiepphan.com/angular-router-series/

Mục tiêu của Day 28 là Feature Module.

## Code example

https://stackblitz.com/edit/angular-100-days-of-code-day-27-router-basic

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day27`

[router]: https://angular.io/guide/router
[day15]: Day015-introduction-dependency-injection-in-angular.md
[01]: assets/day-27-router-01.gif
[02]: assets/day-27-router-02.png
[03]: assets/day-27-router-03.png
[04]: assets/day-27-router-04.gif
[05]: assets/day-27-router-05.png
[06]: assets/day-27-router-06.gif
