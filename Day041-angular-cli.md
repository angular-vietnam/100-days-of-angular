# Day 41: AngularCLI

## Introduction

Hi, 40 ngày qua, các bạn đã đi chung với `Angular` 1 chặng đường rồi. Mình cảm ơn các bạn đã đồng hành với series đến đây, cũng như cùng học, đi qua những concepts khó nhằn trong Angular, Typescript, RxJS,...

Hôm nay bài này sẽ là 1 trạm dừng chân, nghỉ ngơi chút, nhìn lại những gì đã học, đã làm. Nhiều câu lệnh ở đây đã được nói qua ở những bài trước, hôm nay mình sẽ tổng hợp lại.

Đây sẽ là 1 cái `cheat sheet` để tham khảo nhanh, cũng như đâu đó sẽ có những flag, những cách sử dụng CLI một cách hiệu quả và tăng performance trong công việc.

Chúng ta bắt đầu thôi.

## Cheat Sheet

### ng new

Khởi tạo project mới

```sh
    `ng new <name> [options]`
    `ng n <name> [options]`
```

Options:
--directory=directory: Tạo project mới trong folder chỉ định.
--skipInstall=true|false: Tạo project mà không install dependency (hữu ích khi mạng yếu).
--packageManager=npm|yarn|pnpm|cnpm: Chọn package manager cho project.

### ng generate

Generate files

```sh
    `ng generate <schematic> [options]`
    `ng g <schematic> [options]`
```

Shematic:

> Chúng ta có thể chỉ định generate những files này bằng cách thêm tên folder vào sau. Ví dụ chúng ta muốn generate 1 component trong folder `demo\demolist`:

`` sh `ng g c demo\demo-list` ``

> Ngoài ra generate mấy files này, các bạn nên dùng alias để gõ command cho nhanh. Đa số alias là chữ cái đầu của nó. Ví dụ: `module` là `m`.

#### 1. Module

`` sh `ng g module <name>` `ng g m <name>` ``

Options:
--routing=true|false: Tạo module với routing module.

#### 2. Component

`` sh `ng g component <name>` `ng g c <name>` ``
Options:
--export=true|false: Export component vừa tạo.
--inlineStyle=true|false: Chỉ định Style nằm chung trong file.
--inlineTemplate=true|false: Chỉ định HTML nằm chung trong file.
--selector=selector: Tên của selector của component.
--skipTests=true|false: Không generate file spec.

#### 3. Service

`` sh `ng g service <name>` `ng g s <name>` ``
Options:
--skipTests=true|false

#### 4. Class

`` sh `ng g class <name>` `ng g cl <name>` ``
Options:
--skipTests=true|false

#### 5. Interface

`` sh `ng g interface <name>` `ng g i <name>` ``

#### 6. Directive

`` sh `ng g directive <name>` `ng g d <name>` ``
Options:
--export=true|false
--selector=selector
--skipTests=true|false

#### 7. Guard

`` sh `ng g guard <name>` `ng g g <name>` ``
Options:
--skipTests=true|false

#### 8. Pipe

`` sh `ng g pipe <name>` `ng g p <name>` ``
Options:
--export=true|false
--skipTests=true|false

#### 9. Enum

`` sh `ng g enum <name>` `ng g e <name>` ``

### ng build

Compile code và đưa vào thư mục `/dist`

```sh
    `ng build <project> [options]`
    `ng b <project> [options]`
```

Options:
--prod=true|false: Build project với production mode.
--aot=true|false: Sử compiler ahead of time. (Khái niệm này sẽ nói ở những bài sau)
--baseHref=baseHref: Chỉ định baseHref sẽ dùng.
--deployUrl=deployUrl: Chỉ định deployment url sẽ dùng.

### ng serve

Build và Serve code, tự động build lại khi có sự thay đổi file.

```sh
    `ng new <name> [options]`
    `ng n <name> [options]`
```

Options:
-o: Tự động mở project trong browser.
--port=port: Chỉ định port sẽ dùng (default là 4200).

### ng test, ng lint

Chạy unit tests trong project.

```sh
    `ng test <project> [options]`
    `ng t <project> [options]`
```

Options:
--watch=true|false: Tự động test lại khi có thay đổi ở file.
--codeCoverage=true|false: Thêm test coverage report.

Chạy những linting tools.

```sh
    `ng lint <project> [options]`
    `ng l <project> [options]`
```

Options:
--exclude: Chỉ định những file không lint.
--typeCheck=true|false: Check type cho linting.

### ng version

Xem version của Angular CLI.

```sh
    `ng version [options]`
    `ng v [options]`
```

### ng update

Update packages

```sh
    `ng update [options]`
```

## Summary

Day 41 chúng ta đã học được command và flag thường dùng với Angular CLI.
Mục tiêu của ngày 42 sẽ là **Angular Schematics**.

## References

Các bạn có thể đọc thêm ở các bài viết sau

- https://angular.io/cli
- https://malcoded.com/posts/angular-fundamentals-cli/
- https://baswanders.com/angular-cli-cheatsheet-an-overview-of-the-most-used-commands/
- https://www.digitalocean.com/community/tutorials/angular-angular-cli-reference

## Author

[Khanh Tiet](https://github.com/januaryofmine)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day41`

[day39]: Day039-custom-attribute-directive.md
[day40]: Day040-custom-structural-directive.md
