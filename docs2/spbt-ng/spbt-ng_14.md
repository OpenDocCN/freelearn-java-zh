

# 第十四章：在 Angular 中添加认证

在上一章中，我们使用 NgRx 的构建块完成了我们的 Angular 应用程序的 CRUD 功能。我们还学习了在应用程序中编写 actions、reducers 和 effects 的逐步过程，这些将被用于修改状态值。我们还学习了在应用程序中使用和不使用 effects 的区别。Effects 对于我们与允许数据库更改同步到 NgRx 存储的外部 API 进行通信至关重要。

在本章中，我们将学习如何在我们的 Angular 应用程序中添加认证；我们将实现一个登录页面，该页面将提供有效的 JWT，保护路由，并使用 NgRx 应用 API 认证。

在本章中，我们将涵盖以下主题：

+   添加用户认证

+   保护路由

+   调用 API

# 技术要求

本章的完整代码可以在以下位置找到：https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-14。

# 添加用户认证

在开发应用程序时，添加用户认证是主要要求之一。此功能允许我们限制页面和功能不被未经授权的用户访问。我们可以以不同的方式实现用户认证，其中一种实现方式是提供一个要求凭证的登录页面。

让我们一步一步地看看实现认证功能的步骤。

## 认证 API

让我们首先回顾一下我们在 Spring Boot 项目中创建的认证 API。认证端点如下：

+   `{BASE_URL}/authenticate`：主要的认证端点接受包含电子邮件和密码字段的对象，并返回一个有效的 JWT，该 JWT 将用于调用端点。以下是该端点的示例响应对象：

    ```java
    // valid JWT
    ```

    ```java
    {
    ```

    ```java
       "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0QGdtYWl sLmNvbSIsImlhdCI6MTY1OTQyODk2MSwiZXhwIjoxNjU5NDY0OTYxfQ.WU_aZjmlfw--LCovx4cZ4_hcOTGiAgPnSaM0bjdv018"
    ```

    ```java
    }
    ```

+   `{BASE_URL}/register`：创建新有效登录凭证的端点。正如在*第七章*中所述，*使用 JWT 添加 Spring Boot 安全性*，JWT 主要用于无法维护客户端状态的 RESTful 网络服务，因为它包含与用户相关的某些信息。这将在我们请求的端点头部中使用。

在我们的项目中，让我们在 `core/services` 文件夹下创建一个名为 `AuthenticateService` 的服务，通过执行以下命令：

```java
 ng g core/services/authenticate
```

在成功创建服务后，我们将以下代码放置在服务中：

```java
export class AuthenticateService {
  constructor(private http: HttpClient) { }
  // for login endpoint
  login(data: {email: string, password: string}):
    Observable<any> {
    return this.http.post<any>(
     `${environment.authURL}/authenticate`,
      data).pipe(
      tap((data: any) => data),
      catchError(err => throwError(() => err))
   )
  }
  // for register endpoint
  register(data: {email: string, password: string}):
    Observable<any> {
    return this.http.post<any>(
      `${environment.authURL}/register`, data).pipe(
      tap((data: any) => data),
      catchError(err => throwError(() => err))
   )
  }
}
```

`AuthenticateService` 将持有我们将用于登录页面的两个端点。现在，让我们为我们的应用程序创建拦截器。

## HTTP 拦截器

`intercept()` 函数，它将使我们能够获取出站请求并调用下一个拦截器或后端。

我们将主要使用拦截器来修改端点请求的头部，这将负责为每个调用的请求添加 `Authorization: Bearer {JWT}` 头部。

要实现拦截器，我们将创建`core/interceptors/header.interceptor.ts`文件，并将以下代码放置在其中：

```java
@Injectable()
export class HeaderInterceptor implements HttpInterceptor {
  intercept(httpRequest: HttpRequest<any>, next:
    HttpHandler): Observable<HttpEvent<any>> {
    const Authorization = localStorage.getItem('token') ?
      `Bearer ${localStorage.getItem('token')}` : '';
    if(httpRequest.url.includes('api/v1'))
    return next.handle(httpRequest.clone({ setHeaders: {
      Authorization } }));
    else
    return next.handle(httpRequest);
  }
}
```

在前面的代码示例中，我们已经为`intercept()`函数添加了一个新的实现。第一步是从我们的本地存储中检索有效的 JWT，它将被用于 HTTP 头中。我们只有在请求端点包含`api/v1`子字符串时才会使用 JWT，因为这些是受保护的端点。

下一步是克隆请求并添加`Authorization: Bearer {JWT}`头到克隆的请求中，并调用`next()`函数以带有添加的头的 API。

我们现在已经创建了我们的拦截器；最后一步是将拦截器添加到`AppModule`中。

让我们看看以下代码：

```java
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass:
               HeaderInterceptor, multi: true }
  ],
```

在前面的代码示例中，我们现在将拦截反英雄端点的每个 HTTP 调用，并将生成的 JWT 添加到请求头中。

## 认证模块

下一步是创建认证模块；这个模块将负责包含登录和注册页面，该页面将接受用户和凭据并调用认证和注册端点。

要创建认证模块，我们将执行以下命令：

```java
ng g m auth
```

在成功创建认证模块后，我们将导入我们认证模块所需的几个模块：

```java
@NgModule({
  declarations: [
  ],
  imports: [
    CommonModule,
    MaterialModule,
    FormsModule,
    ReactiveFormsModule,
    CoreModule,
  ]
});
```

现在，我们将创建我们模块的不同部分。

### 认证表单

我们将为我们的认证模块创建主要表单；这被认为是我们的模块中的哑组件，因为它将接受和发射表单的值到登录和注册页面。

要创建认证表单组件，我们将执行以下命令：

```java
ng g c auth/components/auth-form
```

在成功创建组件后，我们现在将实现表单的代码。在`auth-form`组件的类型 Script 文件中，我们将放置以下代码：

```java
export class AuthFormComponent {
  @Input() error: string = "";
  @Input() title: string = "Login"
  @Output() submitEmitter = new EventEmitter();
  form: FormGroup;
  constructor(private fb: FormBuilder) {
    this.form = this.fb.group({
      email: [''],
      password: ['']
    })
  }
  submit() {
    this.submitEmitter.emit(this.form.value);
  }
}
```

在前面的代码示例中，我们可以看到我们创建了一个具有电子邮件和密码表单控件的响应式表单。我们还创建了一个发射器，它将表单的值传递给父组件，因为这个组件将被登录和注册页面使用。现在，我们将实现`auth-form`组件的 HTML 代码和 CSS。

注意

请参阅提供的链接以获取完整的代码实现：

[`github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-14/superheroes/src/app/auth/components/auth-form`](https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-14/superheroes/src/app/auth/components/auth-form)

在实现的代码中，我们已经将响应式表单与电子邮件和密码输入绑定。我们还创建了一个条件，按钮会根据页面当前是登录还是注册而改变。

我们已经成功创建了我们的认证表单；现在，我们将创建登录和注册页面。

### 登录和注册页面

登录和注册页面被认为是我们的应用程序的智能组件，因为这些是会调度调用认证 API 的动作的组件。

要创建登录和注册页面，我们将执行以下命令：

```java
ng g c auth/page/login auth/page/register
```

在成功创建两个页面后，我们将运行登录和注册组件的代码：

登录页面

```java
//TS File
export class LoginComponent{
  constructor(private authService: AuthenticateService,
    private router: Router) {
  }
  submit(data:{email:string, password:string}) {
    this.authService.login(data).subscribe((data) => {
      this.router.navigate(['/anti-heroes']);
      localStorage.setItem('token', data.token);
   });
  }
}
// HTML File
<app-auth-form (submitEmitter)="submit($event)"></app-auth-form>
```

注册页面

```java
// TS File
export class RegisterComponent {
  error: string = "";
  constructor(private authService: AuthenticateService) {
  }
  submit(data: User) {
    this.authService.register(data).subscribe((data) => {
      this.router.navigate(['/']);
    });
  }
}
// HTML File
<app-auth-form title="Register" (submitEmitter)="submit($event)"></app-auth-form>
```

在前面的代码示例中，我们可以看到登录页面和注册页面正在使用相同的认证表单组件。一旦表单提交，它将表单值传递给`login()`或`register()`函数以进行认证或创建用户。如果登录成功，我们将用户重定向到反英雄列表页面，并将 API 生成的令牌放置在本地存储中。

## 路由模块

下一步是创建`auth-routing`模块，该模块将定义认证模块的路由。要创建该模块，让我们执行以下命令：

```java
ng g m auth/auth-routing --flat
```

在创建路由模块后，我们将运行以下代码：

```java
const routes: Routes = [
  {
    path: "",
    component: LoginComponent
  },
  {
    path: "register",
    component: RegisterComponent
  }
];
@NgModule({
  declarations: [],
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class AuthRoutingModule {}
```

我们还需要修改我们的`app-routing`模块，因为我们需要基本路径重定向到登录页面；让我们实现以下修改：

```java
const routes: Routes = [
  {
    path: "",
    redirectTo: "login",
    pathMatch: "full",
  },
  {
    path: "login",
    loadChildren: () =>
    import("./auth/auth.module").then((m) => m.AuthModule),
  },
  {
    path: "anti-heroes",
    loadChildren: () =>
      import("./anti-hero/anti-hero.module").then((m) =>
             m.AntiHeroModule),
  }
];
```

在前面实现的代码中，我们可以看到，一旦我们访问基本路径，现在将加载`AuthModule`并将我们重定向到登录页面，如图*图 14.1*所示。

![图 14.1 – 登录页面](img/B18159_14_01.jpg)

图 14.1 – 登录页面

我们现在应该能够使用数据库中的用户登录。如果没有创建用户，我们可以通过注册页面创建一个新的用户，一旦登录成功，我们将被重定向到反英雄列表页面，如图*图 14.2*所示。

![图 14.2 – 反英雄列表页面](img/B18159_14_02.jpg)

图 14.2 – 反英雄列表页面

我们还可以观察到我们的有效 JWT 已经放置在我们的本地存储中，因为 HTTP 拦截器正在使用 JWT。当我们打开应用程序发出的请求时，我们可以看到头中有生成的 JWT：

```java
Accept:
application/json, text/plain, */*
Accept-Encoding:
gzip, deflate, br
Accept-Language:
en-AU,en-US;q=0.9,en;q=0.8,bs;q=0.7,fr-CA;q=0.6,fr;q=0.5,tl;q=0.4
Authorization:
Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0QGdtYWlsLmNvbSIsImlhdCI6MTY1OTY5ODM2NSwiZXhwIjoxNjU5NzM0MzY1fQ.2SDLmvQcME5Be9Xj-zTeRlc6kGfQVNCMIWUBOBS5afg
```

在前面的示例头中，我们可以看到`Authorization`头包含应用程序对每个 API 请求的有效 JWT。JWT 在头中的放置是在我们登录成功并被重定向到`AntiHeromodule`时完成的。

### 令牌验证

下一步我们需要做的是添加令牌验证来检查我们的令牌是否已经过期。为了实现这个功能，我们通过执行以下命令添加`@auth0/angular-jwt`库：

```java
npm install @auth0/angular-jwt --save
```

`@auth0/angular-jwt`库提供了有用的函数，例如`isTokenExpired()`，它检查 JWT 是否已过期，以及`decodeToken()`，它从 JWT 中检索信息。

在成功安装库后，我们将以下代码添加到我们的认证服务中：

```java
  isAuthenticated(): boolean {
    const token = localStorage.getItem('token') ?? '';
    // Check whether the token is expired and return
    // true or false
    return !this.jwtHelper.isTokenExpired(token);
  }
```

我们还需要将 JWT 模块导入到我们的`app.module.ts`文件中：

```java
 imports: [
    JwtModule.forRoot({})
  ],
```

我们将在登录和注册页面上使用`isAuthenticated()`函数来检查我们的本地存储中是否存在 JWT。如果存在有效的 JWT，我们将重定向应用程序到反英雄列表页面。

让我们看看以下实现：

```java
//login page (TS File)
constructor(private authService: AuthenticateService, private router: Router,) {
    this.checkJWT();
  }
checkJWT() {
    if(this.authService.isAuthenticated()) {
      this.router.navigate(['/anti-heroes'])
    }
  }
```

### 注销实现

我们需要实现的最后一个功能是注销功能。为了添加此功能，我们只需要添加一个将令牌从我们的存储中删除的函数。让我们看看以下代码实现：

authenticate.service.ts

```java
export class AuthenticateService {
… other functions
doLogout() {
    let removeToken = localStorage.removeItem('token');
    if (removeToken == null) {
      this.router.navigate(['login']);
    }
  }
```

在前面的代码示例中，我们添加了一个`doLogout()`函数，该函数从存储中删除令牌并将应用程序重定向到登录页面。现在，让我们编辑我们的`navbar`组件以添加注销按钮：

navbar.component.html

```java
<p>
    <mat-toolbar color="primary">
      <span>Angular CRUD</span>
      <span class="example-spacer"></span>
      <button *ngIf="loggedIn" (click)="submit('logout')"
        mat-icon-button>
        <mat-icon>logout</mat-icon>
      </button>
    </mat-toolbar>
  </p>
```

navbar.component.css

```java
.example-spacer {
    flex: 1 1 auto;
  }
```

navbar.component.ts

```java
export class NavbarComponent implements OnInit{
  @Output() actionEmitter = new EventEmitter();
  @Input() loggedIn = false;
  submit(action: string) {
    this.actionEmitter.emit(action);
  }
}
```

在前面的代码实现中，我们为我们的导航栏组件创建了一个发射器。这将发射我们在导航栏中触发的操作，并将其传递到我们的应用程序组件中。

最后一步是在注销按钮被点击时在我们的应用程序组件中调用`doLogout()`函数。让我们看看以下代码实现：

app.component.html

```java
<app-navbar [loggedIn]="url != '/' && !url.includes('login')" (actionEmitter)="submit($event)"></app-navbar>
<div class="container">
    <router-outlet></router-outlet>
</div>
```

app.component.ts

```java
export class AppComponent {
  title = 'superheroes';
  url: string = "";
  constructor(private authService: AuthenticateService,
              private router: Router){
    this.getRoute();
  }
  submit(action: string) {
    switch (action) {
      case 'logout':
        this.authService.doLogout();
        break;
      default:
        break;
    }
  }
  getRoute() {
    this.router.events.subscribe(data => {
    if(data instanceof NavigationEnd) {
      this.url = data.url;
    }
   });
  }
}
```

在前面的代码实现中，我们将认证服务注入到我们的应用程序组件中，并调用了`doLogout()`函数。如果操作是`logout`，我们还在路由更改上添加了一个监听器来检查我们的路由是否当前在登录或注册页面上，如果是，我们将从导航栏组件中移除注销按钮。

我们已经成功实现了应用程序的用户认证，但随着我们继续本章的学习，我们还将改进此实现。在下一节中，我们将讨论如何在 Angular 应用程序中保护路由。

# 保护路由

Angular 的一个基本特性是路由守卫。如果我们要保护我们的路由免受未经认证的直接访问，或者防止用户在意外导航时丢失更改，守卫非常有用。

守卫是 Angular 提供的接口，允许我们通过提供的条件来控制路由的可访问性。这些直接应用于我们想要保护的路由。

让我们看看 Angular 提供的一些守卫：

+   `CanActivate`：这是在我们要阻止访问的路由上实现的。

    +   **方法签名**：

        ```java
        canActivate(route:ActivatedRouteSnapshot, state:RouterStateSnapshot):Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree
        ```

前面的代码定义了`CanActivate`守卫的签名。该函数接受`ActivatedRouteSnapshot`和`RouterStateSnapshot`参数，并返回一个`Observable`或`Promise`，可以是`Boolean`或`UrlTree`类型。

+   **创建** **守卫**：

    ```java
    export class AuthGuard implements CanActivate {,
    ```

    ```java
    constructor(priavte auth: AuthService, private router: Router) {}
    ```

    ```java
    canActivate(route: ActivatedRouteSnapshot, state:RouterStateSnapshot): Observable<boolean> |
    ```

    ```java
    Promise<boolean> | boolean {
    ```

    ```java
      // return true permitted in the route, else return
    ```

    ```java
      // false
    ```

    ```java
     }
    ```

    ```java
    }
    ```

在前面的代码示例中，我们创建了一个名为`AuthGuard`的新类；我们还使用`CanActivate`守卫实现了它，并为所需的逻辑添加了`canActivate()`函数。

+   **使用** **守卫**：

    ```java
    // route-module file
    ```

    ```java
    { path: 'hero',
    ```

    ```java
     component: HeroComponent,
    ```

    ```java
     canActivate: [AuthGuard]
    ```

    ```java
    }
    ```

在上述代码示例中，我们使用新创建的`AuthGuard`类在我们的英雄路由中保护它，防止没有有效 JWT 的用户访问。

+   `CanActivateChild`：这与`CanActivateGuard`类似，但这个守卫用于防止访问子路由。一旦将其添加到父路由，守卫将保护所有子路由。

    +   **方法签名**:

        ```java
        canActivateChild(route:ActivatedRouteSnapshot, state:RouterStateSnapshot):Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree
        ```

上述代码示例定义了`CanActivateChild`守卫的签名。该函数接受`ActivatedRouteSnapshot`和`RouterStateSnapshot`参数，并返回一个可以是`Boolean`或`UrlTree`类型的`Observable`或`Promise`。

+   **创建** **守卫**:

    ```java
    export class AuthGuard implements CanActivateChild {
    ```

    ```java
    constructor(private auth: AuthService, private router: Router) {}
    ```

    ```java
    canActivateChild(route: ActivatedRouteSnapshot, state:RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
    ```

    ```java
      // return true permitted in the route, else return
    ```

    ```java
      // false
    ```

    ```java
     }}
    ```

在上述代码示例中，我们创建了一个名为`AuthGuard`的新类。我们还使用`CanActivateChild`守卫实现了它，并添加了`canActivateChild()`函数以实现所需的逻辑。

+   **使用** **守卫**:

    ```java
    {
    ```

    ```java
      path: user',
    ```

    ```java
      canActivateChild: [AuthGuard],
    ```

    ```java
      component: UserComponent,
    ```

    ```java
      children: [
    ```

    ```java
       { path: ':id', component: ProfileComponent},
    ```

    ```java
       { path: ':id/edit', component: SettingsComponent}]
    ```

    ```java
    }
    ```

在上述代码示例中，我们使用新创建的`AuthGuard`类在我们的用户路径中保护其子路由，这些子路由导航到`ProfileComponent`和`SettingsComponent`组件，防止没有有效 JWT 的用户访问。

+   `CanLoad`：这个守卫用于懒加载模块。`CanActivate`守卫只能防止用户通过路由导航；而`CanLoad`守卫则防止导航到和下载懒加载的模块。

    +   **方法签名**:

        ```java
        canLoad(route:Route,segments:UrlSegment[]):Observable<boolean>|Promise<boolean>|boolean
        ```

上述代码示例定义了`CanLoad`守卫的签名。该函数接受`Route`和`UrlSegment[]`参数，并返回一个可以是`Boolean`类型的`Observable`或`Promise`。

+   **创建** **守卫**:

    ```java
    import { CanLoad, Route, Router } from '@angular/router';
    ```

    ```java
    export class AuthGuard implements CanLoad {
    ```

    ```java
    constructor(private router: Router) {}
    ```

    ```java
    canLoad(route:Route,segments:UrlSegment[]):Observable <boolean>|Promise<boolean>|boolean {
    ```

    ```java
     // return true or false based on a condition to load
    ```

    ```java
     // a module or not
    ```

    ```java
    }}
    ```

在上述代码示例中，我们创建了一个名为`AuthGuard`的新类。我们还使用`CanLoad`守卫实现了它，并添加了`canLoad()`函数以实现所需的逻辑。

+   **使用** **守卫**:

    ```java
      {
    ```

    ```java
        path: "hero",
    ```

    ```java
        loadChildren: () =>
    ```

    ```java
          import("./hero/hero.module").then((m) =>
    ```

    ```java
                 m.AntiHeroModule),
    ```

    ```java
          canLoad: [AuthGuard]
    ```

    ```java
      }
    ```

在上述代码示例中，我们使用新创建的`AuthGuard`类在我们的英雄路由中保护它，防止用户在没有有效 JWT 的情况下访问和下载资源。

+   `CanDeactivate`：这是一个用于防止用户离开当前路由的守卫。这在应用中填写表单等场景中非常有用，可以避免在意外导航时丢失一些更改。

    +   **方法签名**:

        ```java
        canDeactivate(component: T, currentRoute: ActivatedRoute Snapshot, currentState: RouterStateSnapshot,nextState?: RouterStateSnapshot): Observable<boolean|UrlTree>|Promise<boolean|UrlTree>|boolean |UrlTree;
        ```

上述代码示例定义了`CanDeactivate`守卫的签名。该函数接受一个泛型组件，以及`ActivatedRouteSnapshot`和`RouterStateSnapshot`参数，并返回一个可以是`Boolean`或`UrlTree`类型的`Observable`或`Promise`。

+   **创建** **守卫**:

    ```java
    // CanDeactivateGuard service
    ```

    ```java
    import { Observable } from 'rxjs/Observable';
    ```

    ```java
    import { CanDeactivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
    ```

    ```java
    export interface CanComponentDeactivate {
    ```

    ```java
    canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
    ```

    ```java
    }
    ```

    ```java
    export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
    ```

    ```java
      canDeactivate(component:CanComponentDeactivate,current Route:ActivatedRouteSnapshot, currentState:RouterState Snapshot, nextState?: RouterStateSnapshot): Observable <boolean> | Promise<boolean> | boolean {
    ```

    ```java
      return component.canDeactivate();
    ```

    ```java
    }
    ```

    ```java
    }
    ```

在上述实现中，我们创建了一个接口，该接口将在`CanDeactivateGuard`服务的组件中使用：

```java
export class FormComponent implements OnInit, CanComponentDeactivate {
canDeactivate(): Observable<boolean> | Promise<boolean> | boolean {
/* return true or false depends on a specific condition if you want to navigate away from this route or not.*/
}
}
```

在上述代码示例中，我们实现了为组件创建的接口。

+   **使用** **守卫**:

    ```java
    { path: ':id/edit', component: FormComponent, canDeactivate: [CanDeactivateGuard] }
    ```

在前面的代码示例中，我们使用了新创建的`CanDeactivateGuard`来防止用户根据`canDeactivate()`函数上应用的条件从`FormComponent`导航出去。

我们已经了解了我们可以在应用程序中使用的不同守卫。现在，让我们在我们的 Angular 项目中实现它。

## 项目实施

在我们的应用程序中需要应用的第一道防线是`CanLoad`守卫。这是必要的，因为我们希望在没有有效的 JWT 的情况下保护我们的反英雄路由不被访问。要创建`CanLoad`守卫，执行以下命令：

```java
ng g g core/guards/auth
```

执行命令后，选择`CanLoad`选项以生成新的`AuthGuard`类。这里我们需要更改的唯一事情是`canLoad()`函数的实现。我们想要应用的条件是如果 JWT 有效，则允许加载路由和模块。

让我们看看以下实现：

```java
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanLoad {
  constructor(private router: Router, private auth:
    AuthenticateService) {}
  canLoad(route: Route, segments:UrlSegment[]):
    Observable<boolean | UrlTree> | Promise<boolean |
      UrlTree> | boolean | UrlTree {
      if (!this.auth.isAuthenticated()) {
        this. router.navigate(['login']);
        return false;
      }
      return true;
}
}
```

在前面的代码示例中，我们使用了`isAuthenticated()`函数来检查 JWT 是否有效且未过期。如果它是有效的，这将返回`true`并允许我们导航路由。否则，它将重定向我们到登录页面。

最后一步是在反英雄路由中应用`AuthGuard`类；在`app-routing.module.ts`文件中，我们将使用以下代码：

```java
const routes: Routes = [
… other routes here
  {
    path: "anti-heroes",
    loadChildren: () =>
      import("./anti-hero/anti-hero.module").then((m) =>
        m.AntiHeroModule),
      canLoad: [AuthGuard]
  }
];
```

我们现在已经在我们的反英雄路由中成功应用了`CanLoad`守卫。为了测试它是否工作，我们可以尝试删除本地存储中的令牌，这将导致我们重定向到没有有效 JWT 的登录页面。

我们需要的最后一个路由守卫是`CanDeactivate`守卫；我们将在这个反英雄表单上应用这个守卫，以防止用户在离开表单时丢失更改。要创建我们的`CanDeactivate`守卫，我们将执行以下命令：

```java
ng g g core/guards/form
```

执行命令后，选择`CanDeactivate`选项，这将生成新的`FormGuard`类。我们将向此类添加一个接口，我们将在`form`组件中使用它。

让我们看看以下代码：

```java
export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> |
    Promise<boolean> | boolean;
}
@Injectable({
  providedIn: 'root'
})
export class FormGuard implements CanDeactivate<unknown> {
  canDeactivate(
    component: CanComponentDeactivate,
    currentRoute: ActivatedRouteSnapshot,
    currentState: RouterStateSnapshot,
    nextState?: RouterStateSnapshot): Observable<boolean |
      UrlTree> | Promise<boolean | UrlTree> | boolean |
      UrlTree {
      return component.canDeactivate ?
        component.canDeactivate() : true;
  }
}
```

在前面的代码示例中，我们创建了`CanComponentDeactivate`接口，该接口将由表单组件实现。这意味着条件将被放置在组件中而不是守卫中。在`FormComponent`中，我们将添加以下代码：

```java
export class FormComponent implements OnInit, CanComponentDeactivate {
   … other code implementation
  canDeactivate(): Observable<boolean> | Promise<boolean> |
    boolean {
    const confirmation = window.confirm('Are you sure?');
    return confirmation;
  }
   … other code implementation
}
```

在前面的代码示例中，我们实现了`FormComponent`，并添加了我们创建的`CanComponentDeactivate`接口；我们添加了一个`window.confirm()`，这将弹出一个对话框，询问用户是否想要离开当前路由。这是一个简单的守卫实现，因为我们还可以添加其他条件，例如，如果我们只想在表单有更改时提出这个问题。

最后一步是在`FormComponent`路由中应用守卫。

让我们看看以下代码：

```java
const routes: Routes = [
 … other routes
  {
    path: "form",
    children: [
      {
        path: "",
        canDeactivate: [FormGuard],
        component: FormComponent
      },
      {
        path: ":id",
        canDeactivate: [FormGuard],
        component: FormComponent
      }
    ]
  },
];
```

一旦我们应用了`CanDeactivate`守卫，从反英雄表单导航出去将弹出一个对话框，如图*14**.3*所示。

![图片](img/B18159_14_03.jpg)

14.3 – 在离开表单时显示对话框

我们现在已经在我们的 Angular 应用程序中成功应用了守卫；在下一节中，我们将使用 NgRx 状态管理直接改进我们的 API 认证调用。

# 调用 API

在之前的一节中，我们已经通过在组件中直接调用认证服务来创建了用户认证。我们还使用`setItem`函数将生成的 JWT 存储在我们的本地存储中，这也在我们的登录组件中发生。

我们想要实现的是减少我们组件的责任，并且正如我们记得的，我们正在使用 NgRx 状态管理来调用 API，我们组件的唯一责任是派发动作，而 NgRx 将完成剩余的工作。

在本节中，我们将通过使用 NgRx 状态管理的构建块来改进我们的 API 调用。

## 创建 actions

我们首先需要创建我们认证功能的 actions。我们将在`auth/state`文件夹中创建一个名为`auth.actions.ts`的文件，并将有以下的代码：

```java
import { createAction, props } from '@ngrx/store';
export enum AuthActions {
 LOGIN = '[AUTH] Login',
 SET_TOKEN = '[AUTH] Set Token',
 CREATE_USER = '[AUTH] Create User',
 AUTH_ERROR = '[AUTH] AUTH_ERROR',
}
export const setToken = createAction(
    AuthActions.SET_TOKEN,
    props<{ token: string }>(),
);
export const setError = createAction(
    AuthActions.AUTH_ERROR,
    props<{ error: any }>(),
);
```

在前面的代码中，我们可以看到我们创建了四种动作类型：`LOGIN`类型将被用于调用登录 API 的效果；`CREATE_USER`类型将被用于调用注册 API 的效果；`SET_TOKEN`类型将被用于在登录 API 调用后，将生成的 JWT 设置到 store 中的 reducer；最后，`AUTH_ERROR`类型将被用于在登录或注册 API 返回错误时在 store 中设置错误。

## 创建 effects

在创建我们的 actions 之后，现在我们将创建用于调用登录和注册 API 的效果。我们将在`auth/state`文件夹中创建一个名为`auth.effects.ts`的文件，并将有以下的实现：

登录 Effect

```java
@Injectable()
  loginUser$ = createEffect(() => {
    return this.actions$.pipe(
        ofType(AuthActions.LOGIN),
        mergeMap(((data: {type: string, payload: User}) =>
          this.authService.login(data.payload)
          .pipe(
            map(data => ({ type: AuthActions.SET_TOKEN,
                           token: data.token })),
            tap(() =>
              this.router.navigate(["anti-heroes"])),
            catchError(async (data) => ({ type:
              AuthActions.AUTH_ERROR, error: data.error }))
          ))
        ))
    }, {dispatch: true}
```

注册 Effect

```java
  createUser$ = createEffect(() => {
    return this.actions$.pipe(
        ofType(AuthActions.CREATE_USER),
        mergeMap(((data: {type: string, payload: User}) =>
          this.authService.register(data.payload)
          .pipe(
            tap(() =>  this.router.navigate(["login"])),
            catchError(async (data) => ({ type:
              AuthActions.AUTH_ERROR, error: data.error }))
          ))
        ))
    }, {dispatch: true}
  );
```

在前面的代码中，我们已经为登录和注册 API 创建了效果。在`loginUser$`效果中，一旦登录 API 调用成功，它将派发`SET_TOKEN`动作并传递生成的 JWT，这也会将我们重定向到反英雄页面。

这是我们之前章节中实现的行为。另一方面，一旦注册 API 调用成功，`createUser$`效果将再次将我们重定向到登录页面。这是一个简单的行为，如果你希望注册成功后发生其他操作，你可以自定义。

我们还实现了`AUTH_ERROR`动作，当登录或注册 API 失败时将被调用。

## 创建 reducers

我们下一步需要做的是创建 reducers。我们将在`auth/state`文件夹中创建一个名为`auth.reducers.ts`的文件，并将有以下的实现：

```java
export interface AuthState {
    token: string;
    error: any
}
export const initialState: AuthState = {
    token: "",
    error: null
}
export const authReducer = createReducer(
  initialState,
  on(setToken, (state, { token }) => { return {...state,
     token}}),
  on(setError, (state, { error }) => { return {...state,
     error}}),
  );
```

在前面的代码示例中，我们可以看到`AuthState`有两个字段，分别是`token`和`error`。`token`字段将在认证 API 成功调用`setToken`动作后包含有效的 JWT，而`error`字段将包含登录或注册 API 失败时生成的错误。

## 创建 selectors

在创建 reducer 之后，我们现在将创建我们的 selector。在这种情况下，我们的 selector 将非常简单，因为我们只需要一个用于`error`字段的 selector。我们将在`auth/state`文件夹中创建一个名为`auth.selectors.ts`的文件，并且我们将有以下的实现：

```java
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { AppState } from 'src/app/state/app.state';
import { AuthState } from './auth.reducers';
export const selectAuthState = createFeatureSelector<AuthState>('authState')
export const selectError = () => createSelector(
    selectAuthState,
    (state: AuthState) => state.error
)
```

在前面的代码示例中，我们为我们的`error`字段创建了一个 selector；我们需要这个 selector 来在我们的组件中向用户显示错误信息。

## 在本地存储中同步

我们将要实现的下个功能是我们状态在本地存储中的同步。我们可以通过在我们的应用程序中使用`localStorage.setItem()`来实现这一点。然而，使用这种方法将不会是可维护的，并且存储中的值设置将位于不同的位置。

为了有更好的实现，我们将使用`ngrx-store-localstorage`库。为了安装库，我们将执行以下命令：

```java
npm install ngrx-store-localstorage --save
```

在成功安装库之后，我们应该确定我们想要与本地存储同步的状态。在我们的例子中，我们希望`auth`状态中的`token`字段能够同步。为了实现这一点，我们在`auth.module.ts`文件中进行了以下代码更改：

```java
import { localStorageSync } from 'ngrx-store-localstorage';
export function localStorageSyncReducer(reducer: ActionReducer<any>): ActionReducer<any> {
  return localStorageSync({keys: ['token']})(reducer);
}
const metaReducers: Array<MetaReducer<any, any>> = [localStorageSyncReducer];
@NgModule({
  declarations: [
   … declared components
  ],
  imports: [
   … other imported modules
    StoreModule.forFeature('authState', authReducer,
     {metaReducers}),
    EffectsModule.forFeature([AuthEffects]),
  ]
})
```

在前面的代码中，我们可以看到我们创建了一个专门的 reducer，它从`ngrx-store-localstorage`中调用`localStorageSync`，负责在本地存储中添加值。

我们也可以指定我们想要同步的字段，在这种情况下，我们在键数组中添加了 token。一旦 token 状态值发生变化，新的值也将被放置在我们的存储中。

## 分发和选择组件

最后一步是分发动作并使用 selector 来处理登录和注册组件。让我们看看以下代码实现中的登录和注册组件：

login.component.ts

```java
export class LoginComponent{
  error$ = this.store.select(selectError());
  errorSub: Subscription | undefined;
  constructor(private store: Store, private authService:
    AuthenticateService, private router: Router,
      private _snackBar: MatSnackBar) {
    this.checkJWT();
    this.getError();
  }
  submit(data: User) {
    this.store.dispatch({type: AuthActions.LOGIN,
                         payload: data})
  }
  ngOnDestroy(): void {
    this.errorSub?.unsubscribe();
  }
  getError() {
    this.error$.subscribe(data => {
      if(data) {
        this._snackBar.open(data.message, "Error");
      }
    })
  }
… other code implementation
}
```

register.component.ts

```java
export class RegisterComponent implements OnInit, OnDestroy {
  error$ = this.store.select(selectError());
  errorSub: Subscription | undefined;
  constructor(private store: Store,  private _snackBar:
              MatSnackBar) {
    this.getError();
  }
  ngOnDestroy(): void {
    this.errorSub?.unsubscribe();
  }
  submit(data: User) {
    this.store.dispatch({type: AuthActions.CREATE_USER,
                         payload: data})
  }
  getError() {
    this.errorSub = this.error$.subscribe(data => {
       if(data) {
         this._snackBar.open(data.message, "Error");
       }
     })
   }
… other code implementation
}
```

我们可以在前面的代码中看到，登录和注册页面几乎有相同的实现。我们已经在`submit`函数中移除了对登录和注册服务的调用，并用分发动作来替换。我们还使用了`selectError()`selector 来监听 API 是否产生了错误。

# 概述

通过这种方式，我们已经到达了本章的结尾；让我们回顾一下你学到的宝贵知识。

我们现在知道了如何在 Angular 应用中实现用户认证，我们使用了一个 HTTP 拦截器来拦截 HTTP 请求，以转换其头部并添加有效的 JWT 以进行 API 调用。我们还了解到了不同的路由守卫，它们允许我们保护路由免受未经授权的访问或防止在离开路由时意外丢失数据。最后，我们学习了如何通过改进 Angular 中认证的实现方式来使用 NgRx 状态管理。

下一章将教会我们如何使用 Cypress 框架在 Angular 中编写端到端测试。
