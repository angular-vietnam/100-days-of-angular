# Day 41: AngularCLI

## Introduction

Hi, 40 ngày qua, các bạn đã đi chung với `Angular` 1 chặng đường rồi. Mình cảm ơn các bạn đã đồng hành với series đến đây, cũng như cùng học, đi qua những concepts khó nhằn trong Angular, Typescript, RxJS,...

Hôm nay bài này sẽ là 1 trạm dừng chân, nghỉ ngơi chút, nhìn lại những gì đã học, đã làm. Nhiều câu lệnh ở đây đã được nói qua ở những bài trước, hôm nay mình sẽ tổng hợp lại.

Đây sẽ là 1 cái `cheat sheet` để tham khảo nhanh, cũng như đâu đó sẽ có những flag, những cách sử dụng CLI một cách hiệu quả và tăng performance trong công việc.

Chúng ta bắt đầu thôi.

## Cheat Sheet

### ng new

Init new project (and install dependency)

```sh
    `ng new <name> [options]`
    `ng n <name> [options]`
```

Options:
--directory=directory
--skipInstall=true|false
--packageManager=npm|yarn|pnpm|cnpm

### ng generate

```sh
    `ng generate <schematic> [options]`
    `ng g <schematic> [options]`
```

Shematic:

#### 1. Module

`` sh `ng g module <name>` `ng g m <name>` ``

Options:
--routing=true|false

#### 2. Component

`` sh `ng g component <name>` `ng g c <name>` ``
Options:
--export=true|false
--inlineStyle=true|false
--inlineTemplate=true|false
--selector=selector
--skipTests=true|false
--changeDetection=Default|OnPush

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

```sh
    `ng build <project> [options]`
    `ng b <project> [options]`
```

Options:
--aot=true|false
--baseHref=baseHref
--crossOrigin=
none|anonymous|use-credentials
--deployUrl=deployUrl
--optimization=true|false
--prod=true|false

### ng serve

```sh
    `ng new <name> [options]`
    `ng n <name> [options]`
```

### ng test, ng lint

```sh
    `ng test <project> [options]`
    `ng t <project> [options]`
```

Options:
--codeCoverage=true|false

```sh
    `ng lint <project> [options]`
    `ng l <project> [options]`
```

Options:
--exclude
--files
--typeCheck=true|false

### ng version

```sh
    `ng version [options]`
    `ng v [options]`
```

### ng update

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
