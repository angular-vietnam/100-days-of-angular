# Day 3: ANGULAR DATA BINDING

So, **what is data binding**?

In real-life, if two people has a binding relationship, a change happened in everyday life with one of them would affect
the other person somehow. For example, if a couple has a happy marriage, there is a higher chance of the wife receiving
phone notification informing her that her husband's bank account has received a transfer from his company, exactly at
the husband's payday every month.

Such binding relationships exist among data in programming as well. In today's article, let's explore the categories of
data binding in Angular. These categories differ by the direction of binding relationship(s) between component class and
Angular template.

## INTERPOLATION

As mentioned in [the previous day's article][day2], Angular component, at its core, is just a TypeScript class being
extended with extra metadata, such as Angular template, thanks to the `Component` decorator. So, how can data
from the TypeScript class be sent to and displayed inside its corresponding Angular template?  
Let the double curly braces `{{` and `}}` show you the way:

```typescript
import { Component } from "@angular/core";

@Component({
  selector: "app-hello",
  template: `
    <h2>Hello there!</h2>
    <h3>Your name: {{ user.name }}</h3>
    <p>Your name: {{ user.age }}</p>
  `,
})
export class HelloComponent {
  user = {
    name: "Tiep Phan",
    age: 30,
  };
}
```

Just that simple, we have the user info displayed inside the profile page.

## PROPERTY BINDING

### *HTML* versus *DOM* & *Property* versus *Attribute*
 When an HTML document is processed by the browser, the HTML document is parsed into a data representation for JavaScript (and many other scripting languages such as Python, but they are not the concerns of this article) to read and manipulate the document named **DOM** (Document Object Model). Inside the corresponding DOM representation of an HTML document, the document is a *tree* data structure, and its elements are *tree nodes*. 
 
Among these HTML elements, there are HTML tags that require you to pass data into their *attributes*. When represented as DOM nodes, these tags' attributes correspond to JS objects' *properties*. To give an example with an HTML element, `<input type=”text” value=”something”>` has `type` and `value` as the attributes. After being parsed to DOM node, the HTML element corresponds to an JS object of type [`HTMLInputElement`][HTMLInputElement-doc]:

```js
obj = {
  type: 'text',
  value: 'something',
  attributes: [] // of type `NamedNodeMap`
  // ...other properties and methods
}
```

As seen above, the mentioned `type` and `value` attributes are mapped into the JS object properties accordingly. However, not all the attributes are mapped 1-to-1 to properties. For instance, in Angular, `class` attribute and `className` property do not have the same value. 

### Back to Angular:

Apply these knowledge as we've learnt, We would want to make our Angular application dynamically update some selected properties accordingly to the data they bind to. The question is how? Spoilers alert: we will not resort to `document.querySelector` or `jQuery`.

The Angular way of doing this is the **property binding** syntax with the square brackets `[]`:

```typescript
@Component({
  selector: "app-hello",
  template: `
    <input type="text" [value]="user.name"/>
  `,
})
export class HelloComponent {
  user = {
    name: "Tiep Phan",
    age: 30,
  };
}
```

In this example, `value` property is assigned with an JS expression of `user.name`, where `user` is a public property of `HelloComponent`. When `user` is modified in `HelloComponent`, Angular will handle the updating of `value` property for us.

What's interesting is that `type="text"` is also recognised by Angular as property binding, as it's identical to `[type]="'text''"` (beware of the single quotes inside the outer double quotes). To us, `'text'` is just a string literal; but to Angular, it's still a valid JS expression. So, the extra keystrokes for explicitly specifying property binding is not necessary. 

**Notes**: Besides property binding for HTML elements, we can also specify property binding for components as well.

## EVENT BINDING

Without DOM events, we wouldn't able to let users have interesting interactions with our websites. For example, we would want an alert popped up after the user clicks a button. Traditionally with DOM, `addEventListener` is the favoured solution. To develop such feature in Angular, we need to understand **event binding**. 

To attach an event listener to an HTML element of the component template, the event binding syntax with round brackets `()` is used:

```typescript
@Component({
  selector: "app-hello",
  template: `
    <h2>Hello there!</h2>
    <button (click)="showInfo()">Click me!</button>
  `,
})
export class HelloComponent {
  showInfo() {
    alert("Inside Angular Component method");
  }
}
```

Compared with the below traditional HTML version of this Angular template using an inline event listener, the round brackets clarify our intention of calling into a component method. Without them, the `showInfo` function declaration should have been declared in or imported to the HTML document itself.

```html
<button onclick="showInfo()">Click me!</button>
```

## TWO-WAY BINDING

The previous categories of data binding has shown us how to bind data from component class to the Angular template and vice versa. The last category, **two-way binding** enables us to bind data in both directions, with the syntax uses `[]` wrapping outside of `()`:

```typescript
@Component({
  selector: "app-hello",
  template: `
    <input type="text" [(ngModel)]="user.name"/>
  `,
})
export class HelloComponent {
  user = {
    name: "Tiep Phan",
    age: 30,
  };
}
```

`FormsModule` must be imported in order to use `ngModel`. However, in today's article, you only need to know that the above version is a shorthand for:

```typescript
@Component({
  selector: "app-hello",
  template: `
    <input type="text" [ngModel]="user.name" (ngModelChange)="user.name = $event"/>
  `,
})
export class HelloComponent {
  user = {
    name: "Tiep Phan",
    age: 30,
  };
}
```

`ngModel` and custom two-way binding will be the topics for another day. (Spoilers: Day 9)

## SUMMARY

In this day 3, we've skimmed over the four categories of data binding in Angular, with basic demonstrative code example for each one of them. Combined what you've learnt from these first 3 days of this series, you're now capable of making a simple profile card with Angular:

- https://www.w3schools.com/howto/howto_css_profile_card.asp

If you want to dive further into today's topic with the official Angular documentation, here's the link: 

- https://angular.io/guide/architecture-components#data-binding

For the next day's article, we'll explore conditional `if else` in Angular.

## Youtube Video

[Vietnamse :vietnam:] https://youtu.be/WrMywdbnQfk

## Author

- [Tiep Phan](https://github.com/tieppt)
- Translator: [Minh Tu Hoang](https://github.com/m1nhtu99-hoan9)

## HASHTAG

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day3`

[day2]: ./Day002-AngularApp.md
[HTMLInputElement-doc]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement