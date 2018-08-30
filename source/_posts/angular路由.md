---
title: Angular路由
categories:
  - JavaScript
tags:
  - Angular
toc: true
abbrlink: 527c21d3
date: 2018-05-24 14:56:00
---

## angular路由

> Base href

index.html中存在`<`base`>`标签,路由需要根据这个来确定应用程序的根目录。例如，当我们转到`http://example.com/page1`时，如果我们没有定义应用程序的基础路径，路由将无法知道我们的应用的托管地址是`http://example.com`还是`http://example.com/page1`。
```
<!doctype html>
<html>
  <head>
    <base href="/">
    <title>Application</title>
  </head>
  <body>
    <app-root></app-root>
  </body>
</html>
```
> Using the router

要使用路由，我们需要在 AppModule 模块中，导入 RouterModule
```
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { RouterModule } from '@angular/router';

import { AppComponent } from './app.component';

@NgModule({
  imports: [
    BrowserModule,
    RouterModule
  ],
  bootstrap: [
    AppComponent
  ],
  declarations: [
    AppComponent
  ]
})
export class AppModule {}
```
> RouterModule.forRoot()

RouterModule.forRoot() 方法用于在主模块中定义主要的路由信息，通过调用该方法使得我们的主模块可以访问路由模块中定义的所有指令。
```
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = []; // 便于我们在需要的时候导出ROUTES到其他模块中

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
})
export class AppModule {}
```
> RouterModule.forChild()

RouterModule.forChild() 与 Router.forRoot() 方法类似，但它只能应用在特性模块中。

*根模块中使用 forRoot()，子模块中使用 forChild()。*
```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = [];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES)
  ],
  // ...
})
export class ChildModule {}
```
> Dynamic routes

如果路由始终是静态的，那没有多大的用处。使用动态路由我们可以根据不同的路由参数，渲染不同的页面。
```
import { HomeComponent } from './home/home.component';
import { ProfileComponent } from './profile/profile.component';

export const ROUTES: Routes = [
  { path: '', component: HomeComponent },
  { path: '/profile/:username', component: ProfileComponent }
];
```

/routeUrl/:params

:params是路由参数，而不是URL的实际部分。

在访问路由的时候routerLink或者navigate的时候就可以直接传递参数。

```
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'profile-page',
  template: `
    <div class="profile">
      <h3>{{ username }}</h3>
    </div>
  `
})
export class SettingsComponent implements OnInit {
  username: string;
  constructor(private route: ActivatedRoute) {}
  ngOnInit() {
    this.route.params.subscribe((params) => this.username = params.username);
  }
}
```
> Child routes

每个路由都支持子路由，在setttings路由中定义了两个子路由，它们将继承父路由的路径。
```
import { SettingsComponent } from './settings/settings.component';
import { ProfileSettingsComponent } from './settings/profile/profile.component';
import { PasswordSettingsComponent } from './settings/password/password.component';

export const ROUTES: Routes = [
  {
    path: 'settings',
    component: SettingsComponent,
    children: [
      { path: 'profile', component: ProfileSettingsComponent },
      { path: 'password', component: PasswordSettingsComponent }
    ]
  }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
})
export class AppModule {}
```
SettingsComponent组件中需要添加router-outlet指令，因为我们要在设置页面中呈现子路由。
```
import { Component } from '@angular/core';

@Component({
  selector: 'settings-page',
  template: `
    <div class="settings">
      <settings-header></settings-header>
      <settings-sidebar></settings-sidebar>
      <router-outlet></router-outlet>
    </div>
  `
})
export class SettingsComponent {}
```
> loadChildren

SettingsModule 模块，用来保存所有 setttings 相关的路由信息：
```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = [
  {
    path: '',
    component: SettingsComponent,
    children: [
      { path: 'profile', component: ProfileSettingsComponent },
      { path: 'password', component: PasswordSettingsComponent }
    ]
  }
];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES)
  ],
})
export class SettingsModule {}
```
在 SettingsModule 模块中我们使用 forChild() 方法，因为 SettingsModule 不是我们应用的主模块。

另一个主要的区别是我们将 SettingsModule 模块的主路径设置为空路径 ('')。因为如果我们路径设置为 /settings ，它将匹配 /settings/settings ，很明显这不是我们想要的结果。通过指定一个空的路径，它就会匹配 /settings 路径，这就是我们想要的结果。

AppModule
```
export const ROUTES: Routes = [
  {
    path: 'settings',
    loadChildren: './settings/settings.module#SettingsModule'
  }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```
通过 loadChildren 属性，告诉 Angular 路由依据 loadChildren 属性配置的路径去加载 SettingsModule 模块。这就是模块懒加载功能的具体应用，当用户访问 /settings/** 路径的时候，才会加载对应的 SettingsModule 模块，这减少了应用启动时加载资源的大小。

- loadChildren 的属性值，该字符串由三部分组成：
    - 需要导入模块的相对路径
    - # 分隔符
    - 导出模块类的名称