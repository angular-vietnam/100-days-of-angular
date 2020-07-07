# Day 28: Angular Router - Feature Modules, Child Routes and Services

Tiếp tục với Angular Router từ hôm trước, hôm nay chúng ta sẽ tìm hiểu những thành phần tiếp theo như Feature Module, Child Routes và một số Services hay sử dụng.

## Feature Modules

Giả sử với [ứng dụng hôm trước](https://stackblitz.com/edit/angular-100-days-of-code-day-27-router-basic), chúng ta mong muốn tách ra nhiều NgModule khác nhau để chia nhỏ ứng dụng ra thay vì chỉ sử dụng một NgModule duy nhất thì có được không? Chẳng phải chúng ta có thể sử dụng nhiều NgModule trong một ứng dụng Angular hay sao??? Làm thế nào để có nhiều Feature Modules mà có support Router?

Câu trả lời chính là sử dụng `RouterModule.forChild` ở các Feature Modules.

### Tách ArticleModule

Đầu tiên chúng ta sẽ tạo mới một NgModule và đưa những phần cần quản lý bởi NgModule đó vào trong như components, services, etc.

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ArticleListComponent } from './article-list/article-list.component';
import { ArticleDetailComponent } from './article-detail/article-detail.component';

@NgModule({
  imports: [
    CommonModule,
  ],
  declarations: [ArticleListComponent, ArticleDetailComponent]
})
export class ArticleModule { }
```

Tiếp theo, chúng ta sẽ config RouterModule giống như đã từng làm với AppRoutingModule, nhưng thay vì gọi `forRoot` thì chúng ta sẽ gọi `forChild` (nguyên nhân tại sao thì các bạn quay trở lại Day 27).

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Routes, RouterModule } from '@angular/router';
import { ArticleListComponent } from './article-list/article-list.component';
import { ArticleDetailComponent } from './article-detail/article-detail.component';

const routes: Routes = [
  {
    path: 'article',
    component: ArticleListComponent
  },
  {
    path: 'article/:slug',
    component: ArticleDetailComponent
  }
];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(routes),
  ],
  declarations: [ArticleListComponent, ArticleDetailComponent]
})
export class ArticleModule { }
```

Như thế là chúng ta đã tạo xong Feature Module kèm theo Router, bây giờ chúng ta cần import nó vào AppModule để có thể sử dụng.

```ts
import { ArticleModule } from './article/article.module';

@NgModule({
  imports: [
    BrowserModule,
    FormsModule,
    ArticleModule, // <== lưu ý thứ tự import này
    AppRoutingModule
  ],
  declarations: [ AppComponent ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```

Bây giờ chúng ta có thể vào app với path `article` để xem Article List.

![App Feature Route](assets/day28-router-1.gif)

## Route config redirect
Để redirect người dùng khi vào một route nào đó đến một route khác, bạn có thể config như sau:

```ts
const routes: Routes = [
  {
    path: '',
    redirectTo: 'article',
    pathMatch: 'full'
  }
];
```

Các bạn lưu ý rằng ở trong trường hợp redirect, thông thường chúng ta nên dùng strategy để check path là `full`.

Vậy có những strategy nào?

Với config `pathMatch`, chúng ta sẽ có thể có 2 strategy là `full` và `prefix`, giá trị mặc định khi bạn không set cho `pathMatch` sẽ là `prefix`.

- Đối với `full`, bạn sẽ compare path mà người dùng đang muốn navigate tới có bằng hay không - tương tự như dùng `==`. Ví dụ: URL người dùng muốn là `tiepphan.com/abc/xyz` thì chúng ta có path tương ứng là `abc/xyz` nên nếu người dùng yêu cầu đi đến `abc/cde` thì sẽ không thỏa mãn với điều kiện chúng ta đang có.
- Đối với `prefix` thì chỉ cần bằng prefix là được. Ví dụ, nếu người dùng muốn vào `tiepphan.com/abc/xyz` thì path `abc` cũng thỏa mãn, do đó những yêu cầu như render component nào, cũng sẽ được thực thi.

Code sample cho phần này có đầy đủ tại đây: https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-1?file=src%2Fapp%2Farticle%2Farticle.module.ts

## Routing Module
Có một kỹ thuật trong Angular Router đó là Routing Module, được dùng để tách phần routing ra thành một module riêng, và được sử dụng kèm với một NgModule thông thường. Trường hợp của AppRoutingModule là một ví dụ.

Bạn hoàn toàn có thể áp dụng kỹ thuật này với các Feature Module như sau.

```ts
const routes: Routes = [
  {
    path: 'article',
    component: ArticleListComponent
  },
  {
    path: 'article/:slug',
    component: ArticleDetailComponent
  }
];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(routes) // <== config routing
  ],
  declarations: [],
  exports: [RouterModule] // <== exports this NgModule
})
export class ArticleRoutingModule { }
```

Ở đây chúng ta sẽ config routing với `RouterModule.forChild(routes)`, sau đó chúng ta exports `RouterModule` ra ngoài để `ArticleModule` có thể sử dụng những directives/components mà `RouterModule` cung cấp mà không cần imports `RouterModule`.

```ts
import { ArticleRoutingModule } from './article-routing.module';

@NgModule({
  imports: [
    CommonModule,
    ArticleRoutingModule
  ],
  declarations: [ArticleListComponent, ArticleDetailComponent]
})
export class ArticleModule { }
```

Full code: https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-2?file=src%2Fapp%2Farticle%2Farticle-routing.module.ts

## Child Routes

Nhìn vào config phía dưới đây các bạn sẽ thấy rằng có một phần prefix khá giống nhau. Vậy chúng ta có cấu trúc nào cho dạng parent-child hay không?

**Cách 1**
```ts
const routes: Routes = [
  {
    path: 'article',
    component: ArticleListComponent
  },
  {
    path: 'article/:slug',
    component: ArticleDetailComponent
  }
];
```

Angular Router cho phép bạn truyền vào cấu trúc parent-child như sau:

```ts
const routes: Routes = [
  {
    path: 'article',
    children: [
      {
        path: '',
        component: ArticleListComponent,
      },
      {
        path: ':slug',
        component: ArticleDetailComponent
      }
    ]
  },
];
```

Đây là cách config tương đương với **Cách 1**. Chúng ta sẽ có dạng parent + child cho `path` ở trên.

Ngoài ra, parent route có thể activate một component, chúng ta thường gọi nó là layout component. Trong component này nhất định phải có chứa `router-outlet`, nó sẽ là điểm đánh dấu để activate các child component.

```ts
const routes: Routes = [
  {
    path: 'article',
    component: ArticleComponent, // <== this component can be called `Layout component`
    children: [
      {
        path: '',
        component: ArticleListComponent,
      },
      {
        path: ':slug',
        component: ArticleDetailComponent
      }
    ]
  },
];
```

Full code: https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-3?file=src%2Fapp%2Farticle%2Farticle-routing.module.ts


## ActivatedRoute Service.

> Provides access to information about a route associated with a component that is loaded in an outlet. Use to traverse the RouterState tree and extract information from nodes. [ActivatedRoute Service](https://angular.io/api/router/ActivatedRoute)

Service này cung cấp một số public API cho phép chúng ta biết được thông tin về route đang activated và component đã được loaded (activated).
### Retrieve params
Ví dụ trong Day 27, chúng ta muốn lấy thông tin của `params`, lúc đó chúng ta đã inject service này vào `ArticleDetailComponent` như sau:

```ts
export class ArticleDetailComponent implements OnInit {
  article$: Observable<Article>;
  constructor(private _route: ActivatedRoute, private _api: ArticleService) {}

  ngOnInit(): void {
    let slug = this._route.snapshot.paramMap.get('slug');
    this.article$ = this._api.getArticleBySlug(slug);
  }
}
```

Ngoài cách sử dụng snapshot ở trên chúng ta có thể sử dụng Observable để observe như sau:

```ts
export class ArticleDetailComponent implements OnInit {
  article$: Observable<Article>;
  constructor(private _route: ActivatedRoute, private _api: ArticleService) {}

  ngOnInit(): void {
    this.article$ = this._route.paramMap.pipe(
      map(params => params.get('slug')),
      switchMap(slug => this._api.getArticleBySlug(slug))
    );
  }
}
```

> Nếu bạn chưa hiểu về RxJS và Observable thì có thể quay lại các bài học trước để tìm hiểu.

Lý do tại sao chúng ta sử dụng Observable ở đây mà không phải là snapshot?

Về nguyên tắc mặc định, khi di chuyển vào một path, Angular Router sẽ cố gắng reuse component trước đó, nếu chưa có, hoặc khác config thì mới tạo mới component.

Ví dụ, bạn đang ở `/article`, và di chuyển vào từng article `/article/bai-viet-1`, và theo config ở phần trước, chúng ta thấy rằng đây là 2 config khác nhau, nên Angular Router sẽ tạo mới component, lúc này snapshot và `paramMap` Observable sẽ có cùng giá trị cho `slug` là `bai-viet-1`.

Một trường hợp khác là khi bạn từ `/article/bai-viet-1` navigate sang `/article/bai-viet-2`, lúc này chúng sẽ sử dụng cùng config, và do component `ArticleDetailComponent` đã được activated rồi, Angular Router sẽ không tạo lại nó nữa, mà reuse luôn. Lúc này snapshot không thay đổi, vì snapshot chỉ tạo một lần duy nhất khi tạo `ArticleDetailComponent`, còn `paramMap` Observable sẽ emit một giá trị mới cho `slug`.

Vậy nên tùy theo từng trường hợp cụ thể mà bạn sẽ dùng các cách khác nhau để có thể lấy về dữ liệu tương ứng.

So sánh giữa hai giải pháp:

Sử dụng `paramMap` Observable:

![paramMap Observable](assets/day28-router-5.gif)

Sử dụng `paramMap` snapshot:

![paramMap snapshot](assets/day28-router-6.gif)

Full code:

- https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-4?file=src/app/article/article-detail/article-detail.component.ts
- https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-5?file=src%2Fapp%2Farticle%2Farticle-detail%2Farticle-detail.component.ts
- https://stackblitz.com/edit/angular-100-days-of-code-day-28-router-feature-6?file=src%2Fapp%2Farticle%2Farticle-detail%2Farticle-detail.component.ts

### Retrieve config: query params, route data, etc
Ngoài việc cung cấp API cho `params`, ActivatedRoute Service cũng cho phép bạn lấy/observe query params thông qua `queryParamMap`.

Ví dụ bạn vào một URL là `tiepphan.com/page/2?sort=createdDate`, thì bạn có thể lấy về `sort` query qua `snapshot.queryParamMap.get('sort')` hoặc
```ts
queryParamMap.subscribe(query => {
  console.log(query.get('sort'));
})
```

Tương tự chúng ta có thể lấy về `route data`, các bạn có thể tìm hiểu kỹ hơn ở đây: https://angular.io/api/router/ActivatedRoute

## Router Service

> A service that provides navigation and URL manipulation capabilities. [Router API](https://angular.io/api/router/Router)

Service này cung cấp cho chúng ta các cách để thao tác với URL, hoặc có thể sử dụng để navigate trong component chẳng hạn.

Ví dụ: Bạn có một button, khi người dùng click vào đó sẽ thực hiện một số task, nếu thành công sẽ navigate về trang chủ chẳng hạn. Lúc này bạn có thể sử dụng 1 trong hai method sau để navigate.

```ts
navigateByUrl(url: string | UrlTree, extras: NavigationExtras = { skipLocationChange: false }): Promise<boolean>;
navigate(commands: any[], extras: NavigationExtras = { skipLocationChange: false }): Promise<boolean>;
```

```ts
class SomeComponent {
  constructor(private router: Router) {}
  onClick() {
    // do something
    this.router.navigate(['/article']);
  }
}
```

Ngoài ra bạn có thể observe Router Event để làm gì đó:

```ts
this.router.events.pipe(
  filter(e => e instanceof NavigationEnd)
).subscribe(e => {
  console.log(e);
});
```

## Summary
Day 28 này cũng đã có nhiều concept về Angular Router, đây đều là những concept không thể thiếu khi bạn phát triển một ứng dụng thực tế, vì thế các bạn nên đọc thêm nhiều về code của nó trên github, cũng như documentation từ Angular.io

Mục tiêu của ngày 29 sẽ là **Angular Router Lazy Loading**

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/guide/router
- https://www.tiepphan.com/angular-router-series/

## Author

[Tiep Phan](https://github.com/tieppt)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day28`
