# Angular-Theme-Setup
### ng new projectName --routing
### Replace with your app-routing.ts with blow code
```js
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
inside your app.module.ts just add
```js
import { AppRoutingModule } from './app-routing.module';
 imports: [
    AppRoutingModule
  ],
```
## Admin
create a folder name admin inside src/admin and create blow files(4)

### 1- admin-routing.module.ts
```js
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
```js
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

``

