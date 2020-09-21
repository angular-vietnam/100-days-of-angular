# Day 37: Async Validator trong Angular Form

Trong [day 36][day36], chúng ta đã tìm hiểu về validate reactive forms trong Angular, cũng như viết một custom validator đơn giản để check xem input có dấu cách hay ko.

Day 37 này sẽ nó thêm về Async Validator trong Angular.

## Prerequisites

Vì dụ hôm nay sẽ hơi khác ví dụ từ các bài trước, tuy vẫn có đôi chút quen thuộc.

Mình sẽ build một form đăng kí user - `registerForm` bao gồm:

1. Username textbox

- Không được bỏ trống, có độ dài từ 6 đến 32 ký tự, chỉ chứa ký tự alphabet.
- Giả sử như có username: trungvo, tieppt, chautran đã tồn tại trong hệ thống. Khi người dùng nhập một trong ba user kể trên thì sẽ ko cho phép đăng kí.

2. Password textbox

- Không được bỏ trống, có độ dài từ 6 đến 32 ký tự, chỉ chưa các ký tự alphabet, digit, và phải chứa ít nhất một ký tự đặc biệt trong list: `!@#\$%^&\*`

3. Retype password

- Cùng yêu cầu như password kể trên.
- Để đảm bảo chắc chắn user nhập đúng password. Giá trị của textbox này phải giống hệt giá trị textbox password ở trên.

Mình sẽ setup form như ở dưới nhé.

```ts
this.registerForm = this._fb.group({
  username: [
    "",
    Validators.compose([
      Validators.required,
      Validators.minLength(6),
      Validators.pattern(/^[a-z]{6,32}$/i),
    ]),
  ],
  password: [
    "",
    Validators.compose([
      Validators.required,
      Validators.minLength(6),
      Validators.pattern(PASSWORD_PATTERN),
    ]),
  ],
  confirmPassword: [
    "",
    Validators.compose([
      Validators.required,
      Validators.minLength(6),
      Validators.pattern(PASSWORD_PATTERN),
    ]),
  ],
});
```

Phần HTML của form sẽ có dạng như sau.

```html
<div class="container">
  <form
    class="register-form"
    [formGroup]="registerForm"
    autocomplete="off"
    (ngSubmit)="submitForm()"
  >
    <h2>Register</h2>
    <div class="row-control">
      <mat-form-field appearance="outline">
        <mat-label>Username</mat-label>
        <input matInput placeholder="Username" formControlName="username" />
      </mat-form-field>
      <pre>{{ registerForm.get("username")?.errors | json }}</pre>
    </div>
    <div class="row-control">
      <mat-form-field appearance="outline">
        <mat-label>Password</mat-label>
        <input
          type="password"
          matInput
          placeholder="Password"
          formControlName="password"
        />
      </mat-form-field>
      <pre>{{ registerForm.get("password")?.errors | json }}</pre>
    </div>
    <div class="row-control">
      <mat-form-field appearance="outline">
        <mat-label>Confirm Password</mat-label>
        <input
          type="password"
          matInput
          placeholder="Confirm Password"
          formControlName="confirmPassword"
        />
      </mat-form-field>
      <pre>{{ registerForm.get("confirmPassword")?.errors | json }}</pre>
    </div>
    <div class="row-control row-actions">
      <button
        mat-raised-button
        color="primary"
        type="submit"
        [disabled]="registerForm.invalid"
      >
        Register
      </button>
    </div>
    <pre>{{ registerForm.value | json }}</pre>
  </form>
</div>
```

## Custom Validators

Dựa vào requirements, ta cần viết 2 custom validator:

1. Async validator để gọi API check xem username đã tồn tại trong hệ thống hay chưa
2. Sync validator để check xem password type lần hai có trùng khớp với password đầu tiên hay ko.

## Async Validator để validate username

Nhắc lại một chút về Async Validator. Đây là các validate function sẽ trả về Promise hoặc Observable. Ví dụ như bạn muốn validate xem username nhập vào đã có trong hệ thống hay chưa. Thông thường bắt buộc bạn phải gửi một yêu cầu lên server để làm việc này, HTTP request thường sẽ trả về Promise/Observable.

Vì ko có API nên mình sẽ mock một hàm để check username và sẽ trả về `false` nếu như input là một trong 3 giá trị: `trungvo, tieppt, chautran`, nếu ko sẽ trả lại true. Mỗi khi `validateUsername` dc gọi mình cũng sẽ in ra console một dòng text `Trigger API call` để tiện cho việc demo ở dưới.

```ts
validateUsername(username: string): Observable<boolean> {
  console.log("Trigger API call");
  let existedUsers = ["trungvo", "tieppt", "chautran"];
  let isValid = existedUsers.every(x => x !== username);
  return of(isValid).pipe(delay(1000));
}
```

Giờ mình sẽ tiến hành viết custom async validator để validate username trùng khớp. Để viết đc một async validator, các bạn có hai lựa chọn:

- Viết một function, nhận vào là một `AbstractControl` và output trả về dạng `Promise<ValidationErrors | null> | Observable<ValidationErrors | null>`. Cả function sẽ có signature dạng `validate(control: AbstractControl): Promise<ValidationErrors | null> | Observable<ValidationErrors | null>`.
- Implement interface [AsyncValidator][asyncinterface], trong đó có định nghĩa sẵn là bạn phải implement function `validate`.

### validateUserNameFromAPI

Mình sẽ làm theo cách thứ nhất. Nếu làm theo cách thứ hai phải truyền thêm một số thông tin như API service vào contructor khi khởi tạo async validator.

- Nếu API trả về là `true`, thì hàm `validateUserNameFromAPI` sẽ trả về null, tức là input này hoàn toàn ko có lỗi gì.
- Còn nếu API trả về `false`, thì hàm `validateUserNameFromAPI` sẽ trả về một object với data là bất cứ gì bạn muốn, nhưng thông tin nên có giá trị một chút để sau này còn dùng hiển thị thông báo lỗi cho user chẳng hạn

```ts
validateUserNameFromAPI(
  control: AbstractControl
): Observable<ValidationErrors | null> {
  return this._api.validateUsername(control.value).pipe(
    map(isValid => {
      if (isValid) {
        return null;
      }
      return {
        "usernameDuplicated": true
      }
    })
  );
}
```

Sau khi viết xong function, chúng ta cần config control để sử dụng validator đó.

```ts
this.registerForm = this._fb.group({
  username: [
    "",
    Validators.compose([
      Validators.required,
      Validators.minLength(6),
      Validators.pattern(/^[a-z]{6,32}$/i),
    ]),
    this.validateUserNameFromAPI.bind(this),
  ],
});
```

Tại sao phải có `bind(this)` thì đây là một chủ đề khá dài dòng 😂 Mình sẽ giải thích sau nhé.

Kết quả như hình dưới. Khi điền đủ 6 kí tự alpha, khi đó username đã pass toàn bộ sync validator thì async validator sẽ đc trigger ngay sau khi bạn điền kí tự thứ 6. Sau đó, mỗi kí tự khi được nhập từ bàn phím vào sẽ trigger một API lên server để validate, như ta đã thấy có console.log "Trigger API call".

![Async Validator trong Angular Form](/assets/day37-01.gif)

### validateUserNameFromAPIDebounce

Bạn có thấy screenshot có điểm nào quen quen ko? Use case khi điền vào searchbox ko trigger API call ngay lập tức mà chỉ call API nếu như giữa hai keystroke cách nhau một khoảng thời gian, thường là `300ms`. Chúng ta cũng có thể implement behavior đó tương tự như dùng async validator. Sửa đoạn code ở trên có dùng timer như ở dưới:

```ts
validateUserNameFromAPIDebounce(
  control: AbstractControl
): Observable<ValidationErrors | null> {
  return timer(300).pipe(
    switchMap(() =>
      this._api.validateUsername(control.value).pipe(
        map(isValid => {
          if (isValid) {
            return null;
          }
          return {
            usernameDuplicated: true
          };
        })
      )
    )
  );
}
```

Sau đó config control để dùng `validateUserNameFromAPIDebounce`.

```ts
this.registerForm = this._fb.group({
  username: [
    "",
    Validators.compose([
      Validators.required,
      Validators.minLength(6),
      Validators.pattern(/^[a-z]{6,32}$/i),
    ]),
    this.validateUserNameFromAPIDebounce.bind(this),
  ],
});
```

Kết quả đây, sau khi đã type `chautr` đủ 6 kí tự. Mình sẽ type ba lăng nhăng và sau khi dừng ko type nữa thì API sẽ được trigger sau 300ms, chứ ko phải sau mỗi keystroke.

![Async Validator trong Angular Form](/assets/day37-02.gif)

## Chú ý!

> Angular doesn't wait for async validators to complete before firing ngSubmit. So the form may be invalid if the validators have not resolved.

Ở form Register, sau khi mình đã điền đúng hai password và giờ mình mới điền username. Trong khoảng thời gian delay 1 giây khi async validator đang đợi kết quả kiểm trả từ API thì nút Register đã được enable trở lại. Nếu user nhanh tay bấm Register thì ngSubmit vẫn dc trigger và bạn vẫn sẽ thực hiện phần phần code cho submit form. Sau 1 giây sau khi đã validate xong, nếu username bị trùng lặp thì nút Register mới lại đc vô hiệu hóa.

Để chứng minh thì mình sẽ add thêm hàm `onSubmit`:

```ts
submitForm() {
  console.log("Submit form leh");
}
```

![Async Validator trong Angular Form](/assets/day37-03.gif)

Có thể thấy là mình vẫn bấm được nút Register trong khi đang validate username 😂 Để fix lỗi này thì mình có tham khảo [một câu trả lời trên stackoveflow][stack].

Ý tưởng là thay vì ngSubmit sẽ trigger thẳng hàm submit, thay vào đó mình sẽ tạo ra một Subject tên là `formSubmit$` và handle chỉ khi nào status của form chuyển thành `VALID` thì `formSubmit$` mới emit một value, từ đó mới call hàm `submitForm`.

```ts
this.formSubmit$
  .pipe(
    tap(() => this.registerForm.markAsDirty()),
    switchMap(() =>
      this.registerForm.statusChanges.pipe(
        startWith(this.registerForm.status),
        filter((status) => status !== "PENDING"),
        take(1)
      )
    ),
    filter((status) => status === "VALID")
  )
  .subscribe((validationSuccessful) => this.submitForm());
```

```html
<form
  class="register-form"
  [formGroup]="registerForm"
  autocomplete="off"
  (ngSubmit)="formSubmit$.next()"
></form>
```

Test thôi anh em. Như trong hình thì trong khoảng thời gian đang validate mà nút Register đc enable thì có bấm nút cũng sẽ ko trigger hàm `submitForm` và ko có console hiện ra.

![Async Validator trong Angular Form](/assets/day37-04.gif)

## Summary

Day 37 chúng ta đã tìm hiểu về async validator với reactive form. Anh em chú ý mấy điểm này:

- Muốn viết async validator thì theo cú pháp `validate(control: AbstractControl): Promise<ValidationErrors | null> | Observable<ValidationErrors | null>`
- Angular sẽ ko chờ async validator hoàn thành rồi mới submit form nên phải thật cẩn thận trong một số trường hợp.

Use case để validate confirm password trùng với password mình sẽ viết trong bài sau nhé. Bài Async Validator này cũng có khá nhiều kiến thức cần nắm.

Mục tiêu của ngày 38 sẽ là **Angular Template Form Validation**

## Code sample

- https://stackblitz.com/edit/100-days-of-angular-day-37-async-validator

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://trungk18.com/experience/angular-async-validator/
- https://www.tiepphan.com/thu-nghiem-voi-angular-reactive-forms-trong-angular/
- https://www.tiepphan.com/thu-nghiem-voi-angular-template-driven-forms-trong-angular/

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day37`

[day35]: Day035-reactive-forms.md
[day36]: Day036-reactive-forms-2.md
[validators]: https://angular.io/api/forms/Validators
[asyncinterface]: https://angular.io/api/forms/AsyncValidator
[stack]: https://stackoverflow.com/questions/49516084/reactive-angular-form-to-wait-for-async-validator-complete-on-submit
