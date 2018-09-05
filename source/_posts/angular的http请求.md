---
title: Angular中的http请求
categories:
  - JavaScript
tags:
  - Angular
  - http
toc: true
date: 2018-09-05 14:56:00
---

## GET
> 设置查询参数

```
https://www.juphy.cn/todos?_page=1&_limit=10
```
> 创建HttpParams对象

1.
```
import { HttpClient, HttpParams } from '@angular/common/http';

const params = new HttpParams().set('_page', 1).set('_limit', 10);

this.http.get('https://www.juphy.cn/todos', {params});
```
通过链式语法调用 set() 方法，构建 HttpParams 对象。这是因为HttpParams对象是不可变的，通过set() 方法可以防止该对象被修改。每当调用 set() 方法，将会返回包含新值的 HttpParams 对象。

2. 使用formString

```
const params = new HttpParams({formString: '_page=1&_limit=10'});
```

3. 使用formObject

```
const params = new HttpParams({ fromObject: { _page: "1", _limit: "10" } });
```

4. 使用request API

```
this.http.request('GET', 'https://www.juphy.cn/todos', {params});
```

## 获取完整响应
默认情况下，httpClient服务返回的是响应体，如果需要获取响应头的相关信息，可以设置options的对象的observe属性值为response来获取完整的响应对象。
```
this.http.get('https://www.juphy.cn/todos',{
    observe: 'response'
}).subscribe(res => {
    console.dir('Response:'+res.status);
})
```
## 设置响应类型
options设置responseType属性的值，'text','arraybuffer','blob'。

## 设置Http Headers

```
const params = new HttpParams({ fromObject: { _page: "1", _limit: "10" } });
const headers = new HttpHeaders().set('Content-Type', 'application/json; charset=UTF-8');
this.http.get('url', {headers, params});
```
## 多个http请求
### 并行发送
```
const parallel$ = forkJoin(
    this.http.get('url1'),
    this.http.get('url2')
);
parallel$.subscribe(res =>{
    res是一个数组包含多个请求返回的结果。
})
```
### 顺序发送
```
const sequence$ = this.http.get('url1')
    .pipe(
        switchMap(res =>{
            // 此res是url1返回的结果
            return this.http.get('url2');
        })
    );
    sequence$.subscribe(res=>{
        res// 此res是url2返回的结果
    })
```
### 控制返回结果
```
const sequence$ = this.http.get('url1')
    .pipe(
        switchMap(res =>{
            // 此res是url1返回的结果
            return this.http.get('url2');
        }, (res1, res2) => [res1, res2]) // 此处控制返回结果
    );
    sequence$.subscribe(res=>{
        res; // [res1, res2]
    })
```
## 请求异常处理
```
import { of } from "rxjs";
import { catchError } from "rxjs/operators";

this.http.get('url').pipe(
    catchError(error => {
        console.error('Error catched', error);
        return of({description: 'Error Value Emitted'})
    })
).subscribe(res => console.log(res));
```

## Http拦截器
### 定义拦截器
```
import { Injectable } from "@angular/core";
import { HttpEvent, HttpRequest, HttpHandler, HttpInterceptor } from "@angular/common/http";
import { Observable, of, throwError } from 'rxjs';
import { mergeMap, catchError } from 'rxjs/operators';

@Injectable()
export class DefaultInterceptor implements HttpInterceptor {
  constructor(private injector: Injector){}

  private handleData(
    event: HttpResponse<any> | HttpErrorResponse
  ): Observable<any>{
    console.log(event);
    switch(event.status){
        case 200:
        break;
        case 401:
        break;
        case 403:
        break;
        case 404:
        break;
        case 500:
        break;
        default:
            if(event instanceof HttpErrorResponse){
                console.warn(
                    '未可知错误，大部分是由于后端不支持CORS或无效配置引起',
                    event
                );
            }
        break;
    }
    return of(event);
  }
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
     let url= req.url; // 设置url形式，统一加上服务端前缀
     const newReq = req.clone(
        url: url,
        headers: req.headers.set("X-CustomAuthHeader", "iloveangular")
     );
     return next.handle(newReq).pipe(
        mergeMap((event: any) => {
            // 允许统一对请求错误进行处理，这是因为一个请求若是业务上错误的情况下其HTTP请求的状态是200的情况下需要
            if(event instanceof HttpReponse && event.status === 200) return this.handleData(event);
            // 若一切正常，则后续操作
            return of(event);
        }),
        catchError((err: HttpErrorResponse) => this.handleData(err))
        );
    }
}
```

### 应用拦截器
app.module.ts
```
import DefaultInterceptor
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true }
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## Http进度事件
```
this.http.get('url', {
    observe: 'events',
    reportProgress: true
}).subscribe((event: HttpEvent<any>) => {
    switch(event.type){
          case HttpEventType.Sent:
            console.log("Request sent!");
            break;
          case HttpEventType.ResponseHeader:
            console.log("Response header received!");
            break;
          case HttpEventType.DownloadProgress:
            const kbLoaded = Math.round(event.loaded / 1024);
            console.log(`Download in progress! ${kbLoaded}Kb loaded`);
            break;
          case HttpEventType.Response:
            console.log("Done!", event.body);
    }
})
```
控制台输出：
```
Request sent!
Response header received!
Download in progress! 6Kb loaded
Download in progress! 24Kb loaded
Done!
```

