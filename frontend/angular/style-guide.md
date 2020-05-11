# Angular style guide

## Table Of Contents

1.  Single responsibility
2.  Access modifiers
3.  Properties order
4.  Naming
5.  Modules
6.  Lazy Loading
7.  Components communication

## Single responsibility

Apply the [_single responsibility principle_ (SRP)](https://wikipedia.org/wiki/Single_responsibility_principle) to all components, services, and other elements. This helps make the app cleaner, easier to read and maintain.

**Avoid**

```ts
/* avoid */

import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { BrowserModule } from '@angular/platform-browser';
import { NgModule, Component, OnInit } from '@angular/core';

class Hero {
  id: number;
  name: string;
}

@Component({
  selector: 'my-app',
  template: `<h1>{{ title }}</h1>
    <pre>{{ heroes | json }}</pre>`,
  styleUrls: ['app/app.component.css'],
})
class AppComponent implements OnInit {
  title = 'Tour of Heroes';
  heroes: Hero[] = [];

  ngOnInit() {
    getHeroes().then((heroes) => (this.heroes = heroes));
  }
}

@NgModule({
  imports: [BrowserModule],
  declarations: [AppComponent],
  exports: [AppComponent],
  bootstrap: [AppComponent],
})
export class AppModule {}

platformBrowserDynamic().bootstrapModule(AppModule);

const HEROES: Hero[] = [
  { id: 1, name: 'Bombasto' },
  { id: 2, name: 'Tornado' },
  { id: 3, name: 'Magneta' },
];

function getHeroes(): Promise<Hero[]> {
  return Promise.resolve(HEROES); // TODO: get hero data from the server
}
```

**Try** define one thing, such as a service or component, per file.

```ts
/* Module  -> app,module.ts */.
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [],
  bootstrap: [AppComponent]
})

export class AppModule {}
```

```ts
/* Interface -> app.interface.ts */
export class Hero {
  id: number;
  name: string;
}
```

```ts
/* Mock Data -> mock-data.ts */
import { Hero } from './app.interface';

export const HEROES: Hero[] = [
  { id: 1, name: 'Bombasto' },
  { id: 2, name: 'Tornado' },
  { id: 3, name: 'Magneta' },
];
```

```ts
/* Service -> app.service.ts */
import { Injectable } from '@angular/core';

import { HEROES } from './mock-data';

@Injectable()
export class AppService {
  getHeroes() {
    return Promise.resolve(HEROES);
  }
}
```

```ts
/* Component -> app.component.ts */
import { Component, OnInit } from '@angular/core';
import { Hero } from './app.interface';
import { AppService } from './app.service';

@Component({
  selector: 'app-root',
  template: `<pre>{{ heroes | json }}</pre>`,
  styleUrls: ['./app.component.scss'],
  providers: [AppService],
})
export class AppComponent implements OnInit {
  heroes: Hero[] = [];

  constructor(private AppService: AppService) {}

  ngOnInit() {
    this.AppService.getHeroes().then((heroes) => (this.heroes = heroes));
  }
}
```

## Access modifiers

- If your properties are using in the template, then they cannot be privates.

```ts
// Bad

@Component({
  selector: 'app-cool',
  template: `<div>I'm {{ name }}</div>`,
})
export class AppComponent {
  private name = 'wolox';
}
```

```ts
// Good

@Component({
  selector: 'app-cool',
  template: `<div>I'm {{ name }}</div>`,
})
export class AppComponent {
  name = 'wolox';
}
```

> That's applied conversely, namely, if you don't use the properties in the template, they must be privates. All of this is because when your app pass to production mode, since Ahead-of-Time compilation don't allow this visibility in the template

- If your property is not mutated in neither place in your class, then it must be `readonly`.

## Properties order

- decorators
  - inputs
  - outputs
  - others
- public properties
- public readonly properties
- private properties
- private readonly properties
- setter and getter properties (accessors)
  - public
  - private

```ts
// Bad

@Component({
  selector: 'app-cool',
  template: `<div>I'm a nice App</div>`,
})
export class AppComponent {
  get woo() {
    return this.anyValue;
  }

  set woo(value) {
    this.anyValue = value;
  }

  private name = 'wolox';
  isCool = true;
  readonly place = 'world';
  @Ouput() fooChange = new EventEmiter();
  @Input() foo;

  constructor() {}
}
```

```ts
// Good

@Component({
  selector: 'app-cool',
  template: `<div>I'm a nice App</div>`,
})
export class AppComponent {
  @Input() foo;
  @Ouput() fooChange = new EventEmiter();
  isCool = true;
  private name = 'wolox';
  readonly place = 'world';

  get woo() {
    return this.anyValue;
  }

  set woo(value) {
    this.anyValue = value;
  }

  constructor() {}
}
```

## Naming

The correct naming is an important process for avoid clashing.
Clashing mean that if you use third-party libraries avoid they are the same name, or conversely.

When you create a new project you must change the prefix. Angular applies `app` by default.
You can change it when you create a new wordspace and invoke a command `ng new <project-name>`, you can add `--prefix=myprefix`.

Beside, some decorators require a necessary naming conventions for its selectors, such as:

**For Pipes**

```ts
// Bad.
// kebab-case

@Pipe({
  name: 'app-cool-pipe',
  ...
})
```

```ts
// Good.
// lowerCamelCase

@Pipe({
  name: 'appCoolPipe',
  ...
})
```

**For Directives**

```ts
// Bad.
// kebab-case

@Directive({
  selector: '[my-cool-directive]',
  ...
})
```

```ts
// Good.
// lowerCamelCase

@Directive({
  selector: '[myCoolDirective]',
  ...
})
```

**For Components**

```ts
// Bad.
// lowerCamelCase

@Component({
  selector: 'appCoolComponent',
  ...
})
```

```ts
// Good.
// kebab-case

@Component({
  selector: 'app-cool-component',
  ...
})
```

## Modules

A module can be easily loaded in different places in your app. Also, the modules can be isolated for [testing](https://angular.io/guide/testing#angular-testbed) as a code unit.

Of course, this module concept is based by shared particulars features:

- Small features.
- Not proving general services or external services.

A module can attend a Component, Directive or Pipe. The key is: _If you need to reuse any of them, you need a shared module._

## Lazy Loading

When your app is growing, it will be more lazy for a complete initial launch. Then, lazy loading will speeds up the app load time by splitting it into multiple bundles and loading them on demand. But, for this we need the Router help.

```ts
{
  path: 'home',
  loadChildren: () => import('somePath').then(m => m.myModule)
}
```