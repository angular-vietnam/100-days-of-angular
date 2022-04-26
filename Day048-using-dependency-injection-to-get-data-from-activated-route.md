# Ứng dụng dependency injection để lấy data từ trong ActivatedRoute

## Giới thiệu

Chào các bạn, trong bài viết này, mình xin chia sẻ một cách giúp giảm thiểu code trùng lặp khi lấy dữ liệu từ trong `ActivatedRoute` service bằng cách ứng dụng dependency injection.

## Các kiểu mẫu code trùng lặp

Đầu tiên minh xin liệt kê ra một số kiểu dùng code lặp đi lặp lại mà mình thường thấy nhất trong dự án.

```typescript
@Component({
  selector: 'app-my-component'
})
export class MyComponent implements OnInit {
  id$: Observable<string> = this.route.paramMap.pipe(
    map(params => params.get('id')),
    takeUntil(this.destroy$),
  );

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    // do something with this.id$
  }
}
```

Đoạn code bên trên lấy một `id` observable từ `ActivatedRoute` service. Ở những chỗ khác có thể là lấy `customerId` , hoặc là `activeTabId` cũng từ `ActivatedRoute` tương tự như trên.

Chúng ta có thể thấy pattern ở đây là dùng `ActivatedRoute` để lấy data từ trong `paramMap`, `queryParamMap`, hoặc là từ `data` , data đó có thể là lấy theo kiểu observable, hoặc là theo kiểu snapshot.

Thực tế là đoạn code trên không có gì sai cả. Nhưng khi nghĩ đến chuyện viết unit test cho component trên, chúng ta phải mock `ActivatedRoute` service.

Code để mock `ActivatedRoute` có thể giống như [thế này](https://angular.io/guide/testing-components-scenarios#activatedroutestub).

```typescript
export class ActivatedRouteStub {
  // Use a ReplaySubject to share previous values with subscribers
  // and pump new values into the `paramMap` observable
  private subject = new ReplaySubject<ParamMap>();

  constructor(initialParams?: Params) {
    this.setParamMap(initialParams);
  }

  /** The mock paramMap observable */
  readonly paramMap = this.subject.asObservable();

  /** Set the paramMap observable's next value */
  setParamMap(params: Params = {}) {
    this.subject.next(convertToParamMap(params));
  }
}
```

Và unit test cho `MyComponent` class sẽ trông như thế này

```typescript
const activatedRouteStub = new ActivatedRouteStub();

describe('MyComponent', () => {
  let fixture: ComponentFixture<MyComponent>;
  let component: MyComponent;

  beforeEach(async () => {
    // mock the value of paramMap
    activatedRoute.setParamMap({id: 1234});

    await TestBed.configureTestingModule({
      declarations: [MyComponent],
      providers: [
        {
          provide: ActivatedRoute,
          useValue: activatedRouteStub
        }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
  });

  it('should get :id from route param', (done) => {
    fixture.detectChanges();

    component.id$.subscribe(id => {
      expect(id).toBe('1234');
      done();
    });
  });
});
```

Nếu như component của bạn có dùng data từ `queryParamMap`, bạn cũng sẽ phải mock nó giống như là làm với `paramMap` vậy.

Thực tế là bạn có thể làm cho logic get data ở bên trên clean hơn bằng cách dùng dependency injection chỉ với 3 bước như sau.

## Bước 1: Viết factory function để lấy data từ ActivatedRoute

Đầu tiên là bạn sẽ tạo một file mới có tên là `activated-route.factories.ts`, và viết một factory function như bên dưới để lấy data từ `ActivatedRoute`. Hàm này bạn sẽ chỉ viết nó một lần và sẽ dùng lại nó ở nhiều chỗ khác sau này.

```typescript
import {ActivatedRoute} from '@angular/router';
import {Observable} from 'rxjs';
import {map} from 'rxjs/operators';

// this factory function will get value as an observable from route paramMap
// based on the param key you passed in
// if your current route is '/customers/:customerId' then you would call
// routeParamFactory('customerId')
export function routeParamFactory(
  paramKey: string
): (route: ActivatedRoute) => Observable<string | null> {
  return (route: ActivatedRoute): Observable<string | null> => {
    return route.paramMap.pipe(map(param => param.get(paramKey)));
  };
}

// this factory function will get value as a snapshot from route paramMap
// based on the param key you passed in
export function routeParamSnapshotFactory(
  paramKey: string
): (route: ActivatedRoute) => string | null {
  return (route: ActivatedRoute): string | null => {
    return route.snapshot.paramMap.get(paramKey);
  };
}

// same as above factory, but get value from query param
// if your current route is 'customers?from=USA
// then you would call queryParamFactory('from')
export function queryParamFactory(
  paramKey: string
): (route: ActivatedRoute) => Observable<string | null> {
  return (route: ActivatedRoute): Observable<string | null> => {
    return route.queryParamMap.pipe(map(param => param.get(paramKey)));
  };
}

// same as queryParamFactory, but get snapshot, instead of observable
export function queryParamSnapshotFactory(
  paramKey: string
): (route: ActivatedRoute) => string | null {
  return (route: ActivatedRoute): string | null => {
    return route.snapshot.queryParamMap.get(paramKey);
  };
}
```

## Bước 2: Khai báo Injection Token và provide value cho token đó trong component

Bước này bạn sẽ khai náo một `InjectionToken` và provide value cho nó như sau

```typescript
export const APP_SOME_ID = new InjectionToken<Observable<string>>(
  'stream of id from route param',
);

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.template.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
  providers: [
    {
      provide: APP_SOME_ID,
      useFactory: routeParamFactory('id'),
      deps: [ActivatedRoute]
    }
  ]
})
export class MyComponent {}
```

Trong `providers` list của component, bạn provide value cho `APP_SOME_ID` bằng cách dùng factory function `routeParamFactory` mà mình đã viết ở bên trên và truyền vào param key là `'id'`. Chuỗi `'id'` này sẽ match với config của bạn trong routes declaration. Ví dụ như bạn khai báo routes như thế này, thì `:id` trong path khi truyền vào factory function nó sẽ là `'id'`.

```typescript
const routes: Routes = [
  {
    path: ':id',
    component: MyComponent
  }
];
```

## Bước 3: inject token vô component và dùng nó

Bước tiếp theo bạn chỉ cần inject token đã khai báo ở bên trên vào `constructor` của component và sử dụng nó.

```typescript
export const APP_SOME_ID = new InjectionToken<Observable<string>>(
  'stream of id from route param',
);

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.template.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
  providers: [
    {
      provide: APP_SOME_ID,
      useFactory: routeParamFactory('id'),
      deps: [ActivatedRoute],
    },
  ],
})
export class MyComponent {
  constructor(
    @Inject(APP_SOME_ID)
    private readonly id$: Observable<string>
  ) {}

  // then do something with this.id$
}
```

Và bây giờ unit test cho component của bạn sẽ trở nên đơn giản hơn khi mình có thể truyền vào trực tiếp value cho `id$` observable mà không cần phải mock `ActivatedRoute` service nữa.

```typescript
describe('MyComponent', () => {
  let fixture: ComponentFixture<MyComponent>;
  let component: MyComponent;

  beforeEach(async () => {
    TestBed.overrideComponent(MyComponent, {
      set: {
        // you provide value for APP_SOME_ID directly here
        providers: [{
          provide: APP_SOME_ID,
          // here I use asyncScheduler to make it truely async, instead of `of('1234')`
          useValue: scheduled(of('1234'), asyncScheduler)
        }]
      }
    });

    await TestBed.configureTestingModule({
      declarations: [MyComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
  });

  it('should get :id from route param', (done) => {
    fixture.detectChanges();

    component.id$.subscribe(id => {
      expect(id).toBe('1234');
      done();
    });
  });
});
```

Cách dùng này có một số benefits như sau

- Giúp cho logic code của bạn không bị lặp đi lặp lại ở nhiều nơi, do đó code nhìn sẽ clean, dễ hiểu và dễ maintain hơn.
- Giúp cho bạn dễ viết unit test cho component hơn. Cái bạn thực sự cần chỉ là quả chuối, chứ không phải là nguyên cả khu rừng, trong đó có con khỉ đang cầm quả chuối đó.

## Bonus

Một khi bạn đã thuần thục được cách dùng injection token như trên rồi thì giờ đây bạn có thể lấy data cho một customer details theo dạng observable dùng injection token như thế này

```typescript
export const APP_CUSTOMER_ID = new InjectionToken<Observable<string>>(
  'stream of id from route param',
);

export const APP_CUSTOMER_DETAILS = new InjectionToken<Observable<Customer>>(
  'stream of customer details'
);

export const PROVIDERS: Provider[] = [
  {
    provide: APP_CUSTOMER_ID,
    useFactory: routeParamFactory('id'),
    deps: [ActivatedRoute],
  },
  {
    provide: APP_CUSTOMER_DETAILS,
    useFactory: (id$: Observable<string>, apiService: ApiService) => {
      return id$.pipe(
        switchMap((id: string) => apiService.getCustomerById(id)),
      );
    },
    deps: [APP_CUSTOMER_ID, ApiService]
  }
];

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.template.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
  providers: [PROVIDERS],
})
export class MyComponent {
  constructor(
    @Inject(APP_CUSTOMER_DETAILS)
    private readonly customer$: Observable<Customer>
  ) {}

  // then do something with this.customer$
}
```

## Lời kết

Như vậy là trong bài viết này, mình đã chia sẻ cho các bạn một kỹ thuật dùng dependency injection trong Angular để giảm thiểu code trùng lặp trong project. Thực ra thì dependency injection có rất nhiều ứng dụng rất độc đáo, nên bản thân mình nghĩ các bạn nên hiểu và ứng dụng nó càng nhiều càng tốt.

Cảm ơn các bạn đã theo dõi bài viết và hẹn gặp lại trong những bài tiếp theo. Nếu các bạn có câu hỏi gì thì có thể để lại comment ở phía dưới nhé.

## Link tham khảo

- [https://indepth.dev/posts/1306/private-providers](https://indepth.dev/posts/1306/private-providers)
- [https://indepth.dev/posts/1471/leveraging-dependency-injection-to-reduce-duplicated-code-in-angular](https://indepth.dev/posts/1471/leveraging-dependency-injection-to-reduce-duplicated-code-in-angular)


## Code repo
- [https://github.com/phhien203/ngx-router](https://github.com/phhien203/ngx-router)

## Author

[Hien Pham](https://twitter.com/HienHuuPham)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day48`
