# Day 27: Angular Router

Trước đây khi các ứng dụng web vẫn chủ yếu folow theo mô hình server side rendering. Tức là khi bạn mở một website, phía server sẽ gửi cho bạn toàn bộ page đó để render. Khi bạn chuyển trang, ví dụ như từ trang chủ của một website mua bán trực tuyến ở đường dẫn , bạn click vào một đường dẫn để xem phần thông tin các sản phẩm về giày dép. Phần server sẽ gửi lại toàn bộ HTML của page đó, bao gồm từ thẻ <html>, <head> và các thẻ script, cho đến phần nội dung cần được hiển thị. Điều này dẫn đến việc với mỗi click lên website, cả website sẽ được reload với phần nội dung mới. Hay còn lại refresh, hay postback trong một số ngôn ngữ.

![Postback][01]

Với mỗi lần có biểu tượng hình xoay xoay spinner, tức là đã có một request lên server để load lại toàn bộ view của website

Cách này đã được dùng cho đến mãi gần đây khi concept về Single Page Application được ra đời. Trong ứng dụng SPA, thì phần template của ứng dụng được package hoàn toàn trong cái file JS. Và với mỗi click trong ứng dụng SPA, đa số sẽ gọi một HTTP request lên server thông qua API để lấy data và update lên phần nội dung được hiển thị. Phần nằm dưới của kĩ thuật này có tên là AJAX, hẳn các bạn đã nghe qua. Nếu là ứng dụng dạng SPA như những application được viết bằng Angular, mỗi khi bạn tương tác với ứng dụng thì gần như bạn sẽ không cảm nhận được là việc tương tác giữa webpage với server như thế nào. Cụ thể là spinner ở trên tab của trình duyệt sẽ ko được hiển thị mỗi lần bạn tương tác.

Vậy làm sao ứng dụng Angular có thể biết được là bạn đang cần truy cập vào phần thông tin nào, và cách hoạt động ra sao. Đó là nhờ [Angular Router][router].

Khi người dùng thực hiện các tác vụ ứng dụng, họ cần di chuyển giữa các view khác nhau mà developer đã config. Ví dụ đơn khi bạn vào kenh14.vn, bạn sẽ thấy một danh sách các bài viết. Bạn click vô một bài rất hay, muốn gửi cho bạn bè. Thì bạn sẽ copy cái đường link có dạng kenh14.vn/bai-nay-hay-lam và gửi cho bạn của mình.

Về phía ứng dụng, khi bạn mở đường dẫn `kenh14.vn/bai-nay-hay-lam`, application phải hiển thị được đúng bài viết mà bạn đã xem. Việc này thường bao gồm hai bước.

- Ứng dụng Angular cần phải được config để bạn có thể mở được đường dẫn có dạng `kenh14.vn/bai-nay-hay-lam`
- Khi mở đường dẫn có dạng như vậy, Angular cần phải hiển thị một layout của một bài báo. Chứ ko thể hiển thị danh sách các bài viết như ở trang chủ được
- Và bài báo đó phải hiện thị nội dung mà bạn đã xem, chứ ko thể là một bài viết ngẫu nhiên được.

Chúng ta cùng xem cách làm như ở dưới nhé.

## Output cuối cùng

Sau khi follow hướng dẫn này, phần ứng dụng hoàn thành sẽ như ở ảnh dưới.

![Final Output][output]

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
ng new routing-app --routing
```

Bạn cần chú ý hai thứ
- routing-app: tên của ứng dụng bạn sẽ tạo
- `--routing`: phần mô tả rằng ứng dụng sẽ được đi kèm với Router

Ngoài ra còn một số câu hỏi CLI sẽ hỏi bạn, như là có muốn dùng loại styling gì, các bạn chọn SCSS hay CSS hay tùy vào sở thích. Mình sẽ chọn SCSS.

### Thêm components để thực hiện cho việc điều hướng (routing)



## Summary

## Code sample

Toàn bộ code trong bài viết này các bạn có thể xem ở link dưới

https://stackblitz.com/edit/angular-100-days-of-code-day-18-pipes

Mục tiêu của Day 28 là Lazy Loading Module

## Author

[Trung Vo](https://github.com/trungk18)

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day27`

[router]: https://angular.io/guide/router
[01]: assets/day-27-router-01.gif
[output]: assets/day-27-output.gif
