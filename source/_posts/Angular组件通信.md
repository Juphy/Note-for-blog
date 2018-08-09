---
title: Angular组件通信 #文章标题
date: 2018-06-20 14:56:00  #写作时间
categories: #文章分类
- JavaScript
tags: #文章标签
- Angular
toc: true # 生成目录
---

## 输入属性（父组件->子组件）
> @Input，自定义属性

app.component.ts
```
import { Component } from '@angular/core';

@Component({
  selector: 'exe-app',
  template: `
   <exe-counter [count]="initialCount"></exe-counter>
  `
})
export class AppComponent {
  initialCount: number = 5;
}
```
counter.component.ts
```
import { Component, Input } from '@angular/core';

@Component({
    selector: 'exe-counter',
    template: `
      <p>当前值: {{ count }}</p>
      <button (click)="increment()"> + </button>
      <button (click)="decrement()"> - </button>
    `
})
export class CounterComponent {
    @Input() count: number = 0;

    increment() {
        this.count++;
    }

    decrement() {
        this.count--;
    }
}
```
## 输出属性（子组件->父组件）
> @Output()，自定义事件

app.component.ts
```
import { Component } from '@angular/core';

@Component({
  selector: 'exe-app',
  template: `
   <p>{{changeMsg}}</p>
   <exe-counter [count]="initialCount"
    (change)="countChange($event)"></exe-counter>
  `
})
export class AppComponent {
  initialCount: number = 5;

  changeMsg: string;

  countChange(event: number) {
    this.changeMsg = `子组件change事件已触发，当前值是: ${event}`;
  }
}
// 自定义事件change，接收发送过来的数据。
```
counter.component.ts
```
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
    selector: 'exe-counter',
    template: `
      <p>当前值: {{ count }}</p>
      <button (click)="increment()"> + </button>
      <button (click)="decrement()"> - </button>
    `
})
export class CounterComponent {
    @Input() count: number = 0;

    @Output() change: EventEmitter<number> = new EventEmitter<number>();

    increment() {
        this.count++;
        this.change.emit(this.count);
    }

    decrement() {
        this.count--;
        this.change.emit(this.count);
    }
}
// 当值改变时，通过事件发射数据接收。
```
## 双向绑定
> [()]，Angular的双向绑定

*通过修改绑定属性的方式，使用双向绑定即可，此时在子组件中只需要接收数据。*

## 模板变量
> 通过子组件标签的#name,则name就相当于子组件component。

parent.component.ts
```
import {Component, OnInit} from '@angular/core';
import {ChildComponent} from './child-component.ts';

@Component({
  selector: 'parent-component',
  template: `
    <child-component #child></child-component>
    <button (click)="child.name = childName">设置子组件名称</button>
  `
})

export class ParentComponent implements OnInit {

  private childName: string;

  constructor() { }

  ngOnInit() {
    this.childName = 'child-component';
  }
}
```
child.component.ts
```
import {Component} from '@angular/core';

@Component({
  selector: 'child-component',
  template: `I'm {{ name }}`
})

export class ChildComponent {
  public name: string;
}
```
## 路由传参
### 在查询参数中传递参数
传递参数页面
```
<a [routerLink]="['/cinema-chain/cinema']" [queryParams]="{chain: 1}">查看影院</a>
```
点击跳转时，/cinema-chain/cinema?chain=1（?chain=1就是从路由里面传递过来的参数）。

接收参数的页面
```
 constructor(private activatedRoute: ActivatedRoute) {
    const chain = this.activatedRoute.snapshot.queryParams['chain'];
  }
```
### 在url路由路径中传递参数
在path中传递参数就需要先修改原有的路径使其可以携带参数。
```
const routes: Routes = [
  {path: 'main/:type', loadChildren: './index/index.module#IndexModule'},
  {path: 'upload', loadChildren: './components/upload/upload.module#UploadModule'},
  {path: 'operation', loadChildren: './components/operation/operation.module#OperationModule'},
  {path: 'compare/:type', loadChildren: './components/compare/compare.module#CompareModule'},
  {path: '**', component: PageNotFoundComponent},
];
整个路径被划分成两段变量
```
传递参数页面
```
<a [routerLink]="['/home',2]">主页</a>
这里的routerLink是一个数组，第一个值为路由的跳转路径，第二值为路由携带参数的值，这里传递的值为2

或者这样传递
 constructor(private router: Router) {
    this.router.navigate(['/product',1]);
    this.router.navigateByUrl('/product/1');
 }

```
页面跳转的结果：/home/2

接收参数页面
```
 constructor(private activatedRoute: ActivatedRoute) {
    const chain = this.activatedRoute.snapshot.params['id'];
  }
```
*不能同时使用参数查询方式和路由路径Url 方式传递同一个页面的参数，否则报错。*

### 参数快照和参数订阅
参数快照：获取路由中传递的参数的值得一个方法就用到了参数快照snapshot。
```
<a [routerLink]="['/home',2]">主页</a>

change_id(){
  this.router.navigate(['/home',1]);
}
路由路径中想home同时传递了两个参数，1和2
```
当在页面第一次加载的时候会创建一次home，将2这个值传入页面，当点击按钮出发change_id事件的时候也会导航到home，但是在此之前主页已经被创建，并已经被赋值，此时导航到主页，主页并不会再次被创建，所以自然不会再次获取第二次导航过来的路由所携带的参数和值，但是路径变为了/home/1。

然而页面上的值仍然是2，获取当前路由所传递的参数值失败。这就是参数快照的弱点，为了解决这个问题引入了参数订阅：subscribe()。
```
 constructor(private activatedRoute: ActivatedRoute) {
    this.activatedRoute.params.subscribe(params => {
        const id = params['id'];
    });
  }
```
采用参数订阅的方式subscribe()获取到一个类型为Params的属性params，并返回params里面的Id复制给本地变量homeID，这样就不会出现路径在变，但是页面里面的参数值不变的情况；

## @ViewChild 装饰器
> 父组件获取子组件数据需要借助@ViewChild(),子组件直接引用。

app.component.ts
```
import { Component, ViewChild, AfterViewInit } from '@angular/core';
import { ChildComponent } from './child.component';

@Component({
  selector: 'my-app',
  template: `
    <h4>Welcome to Angular World</h4>
    <exe-child></exe-child>
  `,
})
export class AppComponent {
  title: number = 123;
  @ViewChild(ChildComponent)
  childCmp: ChildComponent;

  ngAfterViewInit() {
    this.childCmp.name = 'child-component';
  }
}
```
child.component.ts
```
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'exe-child',
    template: `
      <p>Child Component</p>
    `
})
export class ChildComponent {
    name: string = '';
    constructor(private appcomponent:AppComponent) {
        this.appcomponent.title
    }
}
```
## 基于RxJS Subject
message.service.ts
```
import {Injectable} from '@angular/core';
import {of} from 'rxjs/observable/of';
import {Subject} from 'rxjs/Subject';
import {Observable} from 'rxjs/Observable';

@Injectable()
export class MessageService {
  private subject = new Subject<any>();
  message: any;

  sendMessage(message: any) {
    this.message = message;
    this.subject.next(message);
    this.subject.complete();
  }

  clearMessage() {
    this.message = null;
    this.subject.next();
  }

  getMessage(): Observable<any> {
    // return this.subject.asObservable(); // 数据一直在维持，会产生变化
    return of(this.message); // 数据值传递一次
  }
}
```
home.component.ts
```
import { Component } from '@angular/core';
import { MessageService } from './message.service';

@Component({
    selector: 'exe-home',
    template: `
    <div>
        <h1>Home</h1>
        <button (click)="sendMessage()">Send Message</button>
        <button (click)="clearMessage()">Clear Message</button>
    </div>`
})

export class HomeComponent {
    constructor(private messageService: MessageService) {}

    sendMessage(): void {
        this.messageService.sendMessage('Message from Home Component to App Component!');
    }

    clearMessage(): void {
        this.messageService.clearMessage();
    }
}
```
app.component.ts
```
import { Component, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs/Subscription';
import { MessageService } from './message.service';

@Component({
    selector: 'my-app',
    template: `
    <div>
       <div *ngIf="message">{{message.text}}</div>
       <exe-home></exe-home>
    </div>
    `
})

export class AppComponent implements OnDestroy {
    message: any;
    subscription: Subscription;

    constructor(private messageService: MessageService) {
        this.subscription = this.messageService
                                  .getMessage().subscribe( message => {
                                      this.message = message;
                                 });
    }

    ngOnDestroy() {
        this.subscription.unsubscribe();
    }
}
```

*更多[RxJS知识](https://github.com/RxJS-CN)*
