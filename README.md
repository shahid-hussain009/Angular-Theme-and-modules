# Angular Admin Module and Authentication
### ng new projectName --routing
### Replace with your app-routing.ts with blow code
```ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';


const routes: Routes = [
    {
        path: 'admin',
        loadChildren: () => import('./admin/admin.module')
            .then(m => m.AdminModule),
    }
];

@NgModule({
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule]
})
export class AppRoutingModule { }

```
### inside your app.module.ts just add
```js
import { AppRoutingModule } from './app-routing.module';
 imports: [
    AppRoutingModule
  ],
```
## src/app/admin
create a folder name admin inside src/app/admin and create blow files(4)

### 1- admin-routing.module.ts
```ts
import { RouterModule, Routes } from '@angular/router';
import { NgModule } from '@angular/core';

import { AdminComponent } from './admin.component';
import { DashboardComponent } from './dashboard/dashboard.component';

const routes: Routes = [{
    path: '',
    component: AdminComponent,
    children: [
        {
            path: 'dashboard',
            component: DashboardComponent
        },
        {
            path: '**',
        },
    ],
}];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class AdminRoutingModule {
}


```
### 2- admin.component.ts
```ts
import { Component } from '@angular/core';


@Component({
  selector: 'ngx-admin',
  styleUrls: ['admin.component.css'],
  templateUrl: './admin.component.html',
})
export class AdminComponent {

}

```
### 3- admin.component.html
```html
<app-header></app-header>
<app-sidebar></app-sidebar>
<div class="content-wrapper">
    <router-outlet></router-outlet>
</div>
<app-footer></app-footer>
```
### 4- admin.module.ts
```ts
import { NgModule } from '@angular/core';

import { AdminComponent } from './admin.component';
import { AdminRoutingModule } from './admin-routing.module';
import { DashboardComponent } from './dashboard/dashboard.component';
import { HeaderComponent } from './layout/header/header.component';
import { FooterComponent } from './layout/footer/footer.component';
import { SidebarComponent } from './layout/sidebar/sidebar.component';

@NgModule({
  imports: [
    AdminRoutingModule,
  ],
  declarations: [
    AdminComponent,
    DashboardComponent,
    HeaderComponent,
    FooterComponent,
    SidebarComponent,
  ],
})
export class AdminModule {
}

```
# Authentication Process Start here

## Make a login component inside auth/login
To genrate component use  ng g c auth/login --skipTests=true
inside login.component.ts

```ts
import { AuthService } from 'src/app/shared/auth.service';
import { NgForm } from '@angular/forms';
import { Router } from '@angular/router';
export class LoginComponent implements OnInit {
    emailRegex = /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    serverErrorMessages: string;

    constructor(private authService: AuthService, private router: Router) { }

    ngOnInit() {
    }
    onLogin(form: NgForm) {
        if (form.invalid) {
            return;
        }
        this.authService.login(form.value).subscribe(
            res => {
                this.authService.setToken(res['token']);
                this.authService.setUsername(res['username']);
                this.router.navigateByUrl('/dashboard');

            },
            err => {
                this.serverErrorMessages = err.message;
            }
        );
    }
}

```
### Make auth servie inside src/app/shared/auth.service.ts
To genrate service use  ng g s shared/auth --skipTests=true

```ts
noAuthHeader = { headers: new HttpHeaders({ 'NoAuth': 'True' }) };

    constructor(private http: HttpClient, private router: Router) { }

    login(authCredentail){
        return this.http.post(environment.apiBaseUrl+'/user/login', authCredentail, this.noAuthHeader);
    }
    // set token
    setToken(token: string){
        localStorage.setItem('token', token);
    }
    getToken()
    {
        return localStorage.getItem('token');
    }

    logout(){
        localStorage.removeItem('token');
    }
    deleteToken(){
        localStorage.removeItem('token');
    }

    setUsername(username:any){
        localStorage.setItem('username', username);
        
    }
    getUsername(){
        return localStorage.getItem('username');
    }

    getUserPayload()
    {
        var token = this.getToken();
        if(token)
        {
            var userPayload = atob(token.split('.')[1]);
            return JSON.parse(userPayload);
        }
        else
            return null;
    }

    isLoggedIn(){
        var userPayload = this.getUserPayload();
        if(userPayload)
            return userPayload.exp > Date.now() / 100000;
        else
            return false;
    }

```

### make auth folder inside app/auth  two files auth.guard.ts and auth.interceptor.ts
to genrate guard just use ng g g auth/auth
### auth.guard.ts
inside auth/auth.guard.ts past bellow code
```ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree, Router } from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from '../shared/auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
    constructor(private authService: AuthService, private router: Router) {}
 
    canActivate(
        next: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): boolean {
          if (!this.authService.isLoggedIn()) {
            this.router.navigateByUrl('/login');
            this.authService.deleteToken();
            return false;
          }
        return true;
      }
  
}

```
### auth.interceptor.ts
inside auth/auth.interceptor.ts file and paste below code
```ts
import {
    HttpInterceptor,
    HttpRequest,
    HttpHandler
  } from "@angular/common/http";
  import { Injectable } from "@angular/core";
  
  import { AuthService } from "../shared/auth.service";
  
  @Injectable()
  export class AuthInterceptor implements HttpInterceptor {
    constructor(private authService: AuthService) {}
  
    intercept(req: HttpRequest<any>, next: HttpHandler) {
      const authToken = this.authService.getToken();
      const authRequest = req.clone({
        headers: req.headers.set("Authorization", "Bearer " + authToken)
      });
      return next.handle(authRequest);
    }
  }

```

### Routing
if you are using seprate module and routing like admin then just use AuthGuard in that admin.routing.ts file if not using seprate module then just paste the blow code inside your app.routing.ts file
```ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ModuleWithProviders } from '@angular/core';

import { SignupComponent } from './auth/signup/signup.component';
import { LoginComponent } from './auth/login/login.component';
import { AuthGuard } from './auth/auth.guard';
import { DashboardComponent } from './admin/dashboard/dashboard.component';

import { PageNotFundComponent } from './admin/page-not-fund/page-not-fund.component';


const routes: Routes = [
    { path: '', redirectTo: '/login', pathMatch: 'full' },

    { path: 'signup', component: SignupComponent },
    { path: 'login', component: LoginComponent },

    { path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] },
    { path: '**', pathMatch: 'full', component: PageNotFundComponent, canActivate: [AuthGuard] },
];

export const Routing: ModuleWithProviders = RouterModule.forRoot(routes, {
    useHash: true,
    scrollPositionRestoration: 'enabled',
    anchorScrolling: 'enabled',
    enableTracing: false
  });


```











