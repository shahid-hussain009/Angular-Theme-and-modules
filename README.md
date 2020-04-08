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
create a folder name admin inside src/admin and create blow files(5)


