# Day 12: TypeScript Advanced Type

Ở ngày 12 này, chúng ta sẽ tiếp tục tìm hiểu thêm về **TypeScript** (**TS**) nhé.

> Hệ thống types của **TypeScript** thực sự rất mạnh mẽ và phức tạp, không thể truyền đạt được hết qua 1 vài bài viết. Các bạn phải thực hành, luyện tập, và nghiên cứu rất nhiều thì mới có thể nắm vững được **TS** nhé.

## Union Type

`Union Type` là những types mang tính chất: **EITHER OR** (tạm dịch là **Hoặc cái này Hoặc cái kia**). Để viết `Union Type`, chúng ta dùng **Pipe Symbol** (`|`).

Ví dụ, chúng ta có 1 hàm `listen()` như sau:

```typescript
function listen(port: unknown) {
  if (typeof port === 'string') {
    port = parseInt(port, 10);
  }
  server.listen(port);
}
```

#### `typeof`

Ở ví dụ trên, chúng ta gặp 1 cú pháp lạ `typeof`. `typeof` là 1 operator dùng để lấy về type của 1 biến. Giá trị mà `typeof` trả về luôn có type là `string`

```typescript
typeof 'string'; // string
typeof 123; // number
typeof true; // boolean
typeof {}; // object
typeof []; // object
typeof (() => {}); // object
typeof null; // null
typeof undefined; // undefined
typeof new Date(); // object

typeof String; // function
typeof Boolean; // function
typeof NaN; // number

typeof typeof 123; // string
```

Quay trở lại hàm `listen()`, hàm `listen()` nhận vào 1 tham số `port` có type là `unknown`, nghĩa là chúng ta truyền vào 1 tham số với type nào đó mà chúng ta _chưa biết_ (`unknown`) tại thời điểm viết code. Nói cách khác, hàm `listen()` trên có thể nhận: `string`, `number`, `boolean`, `array`, `object`, và `function` (kể cả `undefined` và `null`).

> `unknown` là type được giới thiệu trong **TS 3.0**. **TS** khuyến cáo sử dụng `unknown` thay vì `any` ở nhiều trường hợp các bạn muốn có 1 type _chưa biết type lúc code_ nhưng hạn chế _type chưa biết_ này. Các bạn nên đọc thêm về `unknown`: [new unknown top type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type)

Vậy là chúng ta đã biết `unknown` là type như thế nào: bất cứ giá trị nào cũng có thể gán được cho `unknown`. Điều này thật sự không hay tí nào, vì implementation của hàm `listen()` chỉ xử lý 2 types là `string` và `number` mà thôi. Giờ xử lý như thế nào? Cách tốt nhất là dùng `Union Type`. Chúng ta sẽ viết lại hàm `listen()` như sau:

```typescript
function listen(port: string | number) {
  // do listen
}

listen('3000'); // ok
listen(3000); // ok
listen(true); // TypeError: Argument of type true is not assignable to parameter type string | number
listen(); // TypeError: Invalid number of arguments, expected 1
```

**TS** _la làng_ (`Compilation Time Error`) ngay nếu như truyền vào 1 `boolean` (không phải `string` HOẶC `number`) hoặc không truyền vào gì khi gọi hàm `listen()`.

> Khi chạy `tsc` để compile code **TS** sang **JS**, các bạn cũng sẽ gặp `Compilation Error` tương tự. Tuy nhiên, code **JS** cũng sẽ được compiled ra đàng hoàng, các bạn cũng sẽ chạy được code **compiled JS** kia nhưng sẽ gặp lỗi ở runtime hay còn gọi là `Runtime Error`. Phân biệt 2 loại errors này nhé 🙂

Tương tự như tạo `Union Type` cho tham số, các bạn cũng có thể tạo `Union Type` cho giá trị trả về của hàm nhé.

```typescript
function getSomething(): string | number {
  return 'string'; // works
  return 30; // works
  return true; // TypeError: Returned expression type boolean is not assignable to type string | number
}
```

Để tái sử dụng (reuse) 1 `Union Type` bất kỳ, các bạn có thể tạo `Type Alias` cho `Union Type` đó

```typescript
type StringOrNumber = string | number;

function listen(port: StringOrNumber) {...}
function getSomething(): StringOrNumber {...}
```

## Intersection Type

Ngược với `Union Type`, `Intersection Type` là type mà kết hợp nhiều type lại với nhau. Nói cách khác, `Intersection Type` là type có tính chất: **AND** (dịch nôm na là **và**).

```typescript
function merge<T1, T2>(o1: T1, o2: T2): T1 & T2 {
  return { ...o1, ...o2 };
}

merge({ foo: 'bar' }, { bar: 'foo' });
```

Hàm `merge({ foo: 'bar' }, { bar: 'foo' })` này sẽ có giá trị trả về là `{ foo: string } & { bar: string }`.

Ngoài những cách sử dụng thông thường trong `function` hoặc trong những đoạn code mà khái niệm **OOP** thông thường không áp dụng được thì `Intersection Type` được dùng rất nhiều trong việc thiết kế hệ thống type cho những thư viện UI Components. Ví dụ:

```typescript
type StyleProp = {
	backgroundColor: string;
	color: string;
	margin: string;
	padding: string;
	...
}

type ButtonProps = {
	onClick: (event: MouseEvent) => void;
} & StyleProps;

type TextProps = {
	fontSize: string;
	fontWeight: number;
	...
} & StyleProps;
```

Những `Component` này có những type `Style` khác nhau, nhưng cũng có những type cơ bản giống nhau. Ví dụ như `Text` sẽ có thêm `fontSize`, `fontWeight` còn `Button` sẽ có `onClick`. Tác giả của những thư viện UI này sẽ sử dụng `Intersection Type` để viết thư viện UI của họ mà không phải lặp đi lặp lại nhiều 1 số type giống nhau. Cách dùng `Intersection Type` này còn có tên gọi khác là `Type Composition`.

> `Type Composition` là 1 chủ đề rất hay, và rộng lớn trong **TS**. Các bạn nên google để tự tìm hiểu thêm nhé.

## Conditional Type

`Conditional Type` có mặt trong **TS** từ version 2.8 và có thể nói đây là một trong những tính năng nổi bật nhất của **TS**. `Conditional Type`, đúng như tên gọi của nó, giúp cho chúng ta có thể tạo ra những type theo điều kiện. Điều này dẫn đến 1 hệ thống type cực kỳ linh hoạt (_robust_) mà **TS** mang lại cho người dùng. Ví dụ:

```typescript
T extends U ? X : Y;
```

Đoạn code trên có thể hiểu nôm na là khi type `T` có thể gán được cho type `U` thì sẽ trả về type `X`, còn không thì trả về type `Y`.

## Type Alias

`Type Alias` có thể hiểu là _alias_ (tên thay thế) một hoặc nhiều loại types nào đó thành 1 type, giống như `StringOrNumber` phía trên. `StringOrNumber` là 1 `Type Alias` của `string | number` (`Union Type`). `Type Alias` có thể dùng cho bất cứ loại type nào.

#### Type Alias và Union Type

Sau đây, chúng ta sẽ cùng xem qua thêm 1 ví dụ về `Type Alias` dùng cho `Union Type` nhé. Tưởng tượng chúng ta cần tạo 1 component `Flex` và component này có những yêu cầu cơ bản sau:

- `Flex` sẽ có `style` mặc định là `display: 'flex'
- `Flex` sẽ nhận vào 1 `Input` `flexDirection` để có thể gán vào `style` như sau: `flex-direction: flexDirection`

```typescript
@Component({
  selector: 'flex-container',
  template: `<ng-content></ng-content>`,
})
export class FlexComponent {
  @Input() flexDirection: string = 'row';

  @HostBinding('style.display') get display() {
    return 'flex';
  }

  @HostBinding('style.flex-direction') get direction() {
    return this.flexDirection;
  }
}
```

> `HostBinding` là khái niệm chúng ta chưa tìm hiểu trong chuỗi 100 ngày này. Các bạn chỉ cần hiểu là chúng ta dùng `HostBinding` để bind giá trị lên selector tag `<flex-container></flex-container>`

Như đa số các bạn đã biết, `flexDirection` của một **flex container** sẽ có thể có 1 trong 4 gía trị: `column`, `row`, `column-reverse`, và `row-reverse`. Nhưng ở đoạn code trên, chúng ta có thể truyền vào bất cứ 1 giá trị `string` nào cho `flexDirection`. Và điều này sẽ dẫn đến `style` của component này sẽ không được chính xác. Một lần nữa để ngăn ngừa việc truyền tham số tràn lan, chúng ta dùng `Union Type`

```typescript
type FlexDirection = 'row' | 'column' | 'row-reverse' | 'column-reverse';

@Component({
	selector: 'flex-container',
	template: `<ng-content></ng-content>`
})
export class FlexComponent {
	@Input() flexDirection: FlexDirection = 'row';

	@HostBinding('style.display') get display() {...}

	@HostBinding('style.flex-direction') get direction() {
		return this.flexDirection;
	}
}
```

Và cách dùng:

```html
<!-- app.component.html -->
<flex-container>
  <button>Submit</button>
  <button>Cancel</button>
</flex-container>

<flex-container flexDirection="column">
  <input type="email" />
  <input type="password" />
</flex-container>
```

Khi dùng `flex-container` component trên template, **TS** đã có thể gợi ý (intellisense) 4 giá trị của `flexDirection` khi các bạn muốn truyền gia trị cho `flexDirection`

#### Type Alias và Conditional Type

Kế tiếp, chúng ta sẽ tìm hiểu 1 ví dụ về `Type Alias` và `Conditional Type` nhé.

```typescript
type ObjectDictionary<T> = { [key: string]: T };
type ArrayDictionary<T> = { [key: string]: T[] };
export type Dictionary<T> = T extends []
  ? ArrayDictionary<T[number]>
  : ObjectDictionary<T>;

type StringDictionary = Dictionary<string>; // {[key: string]: string}
type NumberArrayDictionary = Dictionary<number[]>; // {[key: string]: number[]}
type UserEntity = Dictionary<User>; // {[key: string]: User}
```

Ở ví dụ trên, chúng ta có 3 `Type Alias`: `ObjectDictionary`, `ArrayDictionary`, và `Dictionary`. Trong đó, `Dictionary` có thể được xem là **True Type** (type được `export` ra cho bên ngoài sử dụng), còn `ObjectDictionary` và `ArrayDictionary` có thể được xem là **Support Type** (type dùng để hỗ trợ cho **True Type**). Và code thì khá dễ hiểu, nếu mình truyền vào 1 type dạng `number[]` cho `type parameter T` ở `Dictionary<T>` thì `T extends []` sẽ được đính giá (evaluate) là `truthy` và `Dictionary<number[]> sẽ trả về type`ArrayDictionary<number>`

Với `Type Alias` và `Conditional Type`, **TS** ngoài việc cung cấp cho chúng ta khả năng tạo những dạng type thú vị như trên và kết hợp chúng lại với nhau, thì **TS** còn cung cấp cho chúng ta 1 số _built-in_ type rất hay. Chúng ta cùng điểm qua một số _built-in_ types hay dùng nhé:

```typescript
// Exclude/Extract
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

// Exclude: Loại bỏ những types ở T nếu như những types này gán được cho U
type SomeDiff = Exclude<'a' | 'b' | 'c', 'c' | 'd'>; // 'a' | 'b'. 'c' đã bị removed.

// Extract: Loại bỏ những types ở T nếu như những types này KHÔNG gán được cho U
type SomeFilter = Extract<'a' | 'b' | 'c', 'c' | 'd'>; // 'c'. 'a' và 'b' đã bị removed.

// hoặc có thể dùng Exclude để tạo type alias NonNullable
type NonNullable<T> = Exclude<T, null | undefined>; // loại bỏ null và undefined từ T

type Readonly<T> = { readonly [P in keyof T]: T[P] }; // làm tất cả các props trong type thành readonly. Eg: Rất có lợi khi dùng cho Redux State.
type Partial<T> = { [P in keyof T]?: T[P] }; // làm tất cả các props trong type thành optional. Eg: Rất có lợi cho việc type 1 tham số khi cần truyền vào 1 type nào đó mà ko bắt buộc.
type Nullable<T> = { [P in keyof T]: T[P] | null }; // cái này tương tự như Partial, Partial sẽ trả về T[P] | undefined. Còn Nullable sẽ trả về T[P] | null

type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Record<K extends keyof any, T> = { [P in K]: T };

// Pick: Pick từ trong T những type có key là K
type Person = {
  firstName: string;
  lastName: string;
  password: string;
};

type PersonWithNames = Pick<Person, 'firstName' | 'lastName'>; // {firstName: string, lastName: string}

// Record: Gán type T cho lần lượt từng key P trong K
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>;
// { prop1: string, prop2: string, prop3: string }

type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

// ReturnType: trả về type của giá trị mà function T trả về.
type Result = ReturnType<() => string>; // string

type Omit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;

// Omit: loại bỏ type có key là K trong T
type PersonWithoutPassword = Omit<Person, 'password'>; // {firstName: string, lastName: string}
```

> Xem thêm tại: [Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html)

## Summary

Như các bạn đã thấy, đi sâu vào hệ thống `types` của **TS** thật sự không quá dễ dàng đúng không? Nhưng hiểu được và nắm được một số kĩ thuật cơ bản thì sẽ gíup các bạn được rất nhiều trong việc sử dụng **TS** không những trong **Angular** mà trong những công nghệ khác nữa.

Ngoài những dạng type mà chúng ta vừa cùng nhau khám phá, **TS** còn rất nhiều những khái niệm thú vị khác như: **Decorator**, **Enum**, và **Mixin** .v.v... Để có thể nói hết tât cả về những khái niệm của **TS** thì sẽ tốn rất nhiều thời gian và đây là series về **Angular** cho nên các bạn nên bỏ thêm thời gian của bản thân để xem qua các khái niệm mình vừa nhắc đến của **TS** nhé.

## Author

[Chau Tran](https://github.com/nartc)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day12`
