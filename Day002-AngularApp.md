# Day 2: EXPLORE ANGULAR APP

Từ project được generate bởi Angular CLI chúng ta có thể thấy được trong đó có khá nhiều các file/folder, vậy application của chúng ta bắt đầu từ đâu.
Đầu tiên, từ folder src bạn có thể thấy được file index.html, bên trong tag body sẽ có một tag HTML khá khác lạ (app-root trong hầu hết trường hợp). Tag này không hề tồn tại trong HTML, ắt hẳn đây là một custom tag/selector của application, hay nói cách khác, đây là cái gì đó bao ngoài của một view nào đó trong ứng dụng.

Tiếp theo, bạn mở file main.ts, đây là nơi khởi đầu của application, bên trong là các đoạn code TypeScript (TS) thông thường, nó là một module (ES module/TS module), nó đã import một số thứ khác từ một số thư viện/module khác để sử dụng. Không có gì đặc biệt lắm trong file này, nó chỉ gọi đến một số hàm nào đó để thực thi việc bootstrap application.

Và sau một vài chỉ dẫn bạn đã có thể tìm thấy file app/app.module.ts, đây là một file TypeScript module khác, bên trong có một class kèm theo cái móc móc gì đó NgModule (một TS decorator), đó chính là một NgModule ở trong một ứng dụng Angular. Đến đây có thể thấy đã bắt đầu confuse, gì mà TS module rồi lại còn NgModule. Thực ra bạn có thể hiểu đơn giản đó là TS module là cách mà chúng ta tổ chức code thành các phần nhỏ có thể là các file, các folder, mỗi một file TS sẽ đóng gói thành một module, nó có thể import module khác, hoặc export một số phần code trong nó cho module khác sử dụng. Còn NgModule là cách mà chúng ta tổ chức phần chức năng của một Angular application. Mỗi Angular app sẽ được chia thành nhiều NgModule, ít nhất đến thời điểm hiện tại thì app của chúng ta mới có một NgModule tên là AppModule (hay được gọi là root NgModule). Trong tương lai NgModule có thể không còn cần thiết cho ứng dụng nữa.

Ở trong AppModule, chúng ta đã thấy được một trong những thành phần quan trọng nhất của các ứng dụng Angular đó là các Component, ở đây là AppComponent, nó được import từ file app.component.ts
Với mỗi một ứng dụng Angular, một component sẽ định nghĩa ra một view tương ứng. Hãy thử tưởng tượng bạn là một người kiến trúc lên các tòa nhà cao tầng, việc chúng ta ghép các Component (có sẵn hoặc phải tự tạo) để tạo ra tòa nhà đó cũng giống như việc chúng ta xây dựng các ứng dụng dựa trên Component (Angular, React đều dựa trên ý tưởng chia ứng dụng thành các component và integrate chúng lại với nhau). Ví dụ, trong ứng dụng của chúng ta có thể có các component như: Main Page, Header, Side Nav, Footer. Và bên trong mỗi component lại có thể là sự hợp thành của nhiều component khác nữa.

Đối với AppComponent, đây là root component của ứng dụng, bạn có thể lại thấy một TS decorator khác nữa tên là Component, và bên dưới là một TS class thông thường.
Ở trong ứng dụng Angular, TS decorator mà Angular cung cấp thông thường sẽ để gắn thêm meta-data cho class/property/method, đối với class AppComponent, decorator Component sẽ gắn thêm một số meta-data như selector, template (view của component, chính là chúng ta sẽ định nghĩa component sẽ hiển thị những gì), etc
View của một component có thể coi là phần HTML mở rộng, nó có nhiều tính năng hơn HTML thông thường. Ở trong view chúng ta có thể sẽ nhúng các component/directive khác.
Vậy là đã rõ, tag app-root mà chúng ta nhìn thấy từ index.html sau một vòng tìm hiểu chúng ta sẽ tìm ra nó thuộc về AppComponent.

## KHỞI TẠO THÊM MỘT COMPONENT MỚI

Một Angular application sẽ được tạo từ nhiều component, nên chúng ta sẽ không để hết code vào AppComponent, bây giờ hãy làm thử một component khác xem sao.
Chúng ta sẽ tạo mới file hello.component.ts cùng trong thư mục của app.component.ts và thực hiện coding.

```typescript
export class HelloComponent {}
```

Nó là một TS class rất đơn giản phải không, bây giờ chúng ta sẽ gắn meta-data cho nó như sau.

```typescript
import { Component } from "@angular/core";
@Component({
  selector: "app-hello",
  template: ` <h2>Hello there!</h2> `,
})
export class HelloComponent {}
```

Thế là đã xong một component đó, easy.
Với selector trên nếu chúng ta sử dụng như AppComponent thì có được không.
Bây giờ bạn mở file app.component.html lên và chèn thêm selector của HelloComponent vừa tạo xem sao.

```typescript
<app-hello></app-hello>
```

Save file và khởi chạy ứng dụng với lệnh `ng serve` như day 1 xem sao.
Không được rồi, chúng ta đã gặp lỗi.

error NG8001: 'app-hello' is not a known element:

1. If `app-hello` is an Angular component, then verify that it is part of this module.
2. If `app-hello` is a Web Component then add 'CUSTOM_ELEMENTS_SCHEMA' to the '@NgModule.schemas' of this component to suppress this message.
   Rõ ràng rồi, chúng ta nhìn thấy lỗi cũng đã gợi ý cho chúng ta cách fix. Chúng ta mới chỉ tạo ra component mà chưa declare nó sẽ thuộc NgModule nào cả, giống như khi bạn gia nhập một công ty mới, ngày đầu tiên bạn chưa biết mình thuộc ai quản lý, lúc này có thể bạn sẽ không thể vào văn phòng vì không ai biết bạn là ai cả. Vì vậy chúng ta cần làm các bước để khai sinh cho component vừa mới được sinh ra.

   Trong app hiện tại có NgModule duy nhất là AppModule, chúng ta sẽ thêm vào đó, nhưng vấn đề là thêm vào đâu?
   Khi bạn mở AppModule sẽ thấy AppComponent được đưa vào NgModule decorator, chắc là component mới kia cũng thế. Và một thoáng nhìn, chúng ta thấy cái keyword khá dễ mường tượng declarations, sau khi hover vào chúng ta sẽ thấy editor sẽ hiển thị một message:
   **The set of components, directives, and pipes (declarables) that belong to this module.**
   `@usageNotes` — The set of selectors that are available to a template include those declared here, and those that are exported from imported NgModules.
   Declarables must belong to exactly one module. The compiler emits an error if you try to declare the same class in more than one module. Be careful not to declare a class that is imported from another module.

   Yeah, chính là nó đó, giờ chỉ việc import component lên đầu và thêm HelloComponent vào declarations là xong.
```typescript
import { HelloComponent } from './hello.component'
```

```typescript
declarations: [
  AppComponent,
  HelloComponent,
],
```

App đã chạy thành công, bây giờ nhìn kỹ lại thì việc để tạo được một component cũng không quá khó. Ngoài ra, declarations array có thể dùng cho cả component, pipe, directive (chắc là cái concept gì đó nữa ở trong Angular).
Ngoài cách tạo component bằng tay như trên, bạn có thể tạo bằng Angular CLI như sau:
ng generate component hello
Với cách tạo bằng tool thì bạn sẽ không cần phải làm những giai đoạn bằng tay lặp đi lặp lại nữa.
That's all for today.

Các bạn hãy thử tìm hiểu cấu trúc ứng dụng và tạo thêm nhiều component nữa nào.

## Youtube Video

https://youtu.be/jgFw8tAgKNs

## Link tham khảo

Link document các bạn cần tìm hiểu trong Day 2

- https://angular.io/guide/architecture
- https://angular.io/guide/architecture-modules
- https://angular.io/guide/architecture-components
 
Mục tiêu Day 3 sẽ là về **data binding**.

## Author

[Tiep Phan](https://github.com/tieppt)

## HASHTAG

`#100DaysOfCodeAngular #100DaysOfCode`
`#AngularVietNam100DoC_Day2`
