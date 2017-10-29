# User Management system build with Angular4
User management system build using Angular4 bootstrap with examples how to manage user roles using Angular.
This presentation of live project - for demonstration purposes only.

## Example managing user roles with directives:
Managing visible elements - should ony be used for small pieces of code, since it's not structural directive like ngIf.
For this project I used this directive to hide element like buttons, table rows, links, texts.

## DEMO preview
![alt text](https://i.imgur.com/ItvRimL.gifv "Demo IMS")

##### role-permissions.directive.ts
```javascript
import {Directive, ElementRef, Input, OnInit} from '@angular/core';
import {UserService} from '../service/user.service';

@Directive({
  selector: '[userRole]'
})
export class RolePermissionsDirective implements OnInit {

  private currentUser;

  @Input() userRole;

  constructor(private elRef: ElementRef,
              private userService: UserService) {

    this.elRef.nativeElement.style.display = 'none';
  }

  ngOnInit() {
    const roles = this.userRole.split(',');
    this.currentUser = this.userService.currentUser;
    let displayType = 'block';

    switch (this.elRef.nativeElement.localName) {
      case 'td':
      case 'th':
        displayType = 'table-cell';
        break;
      case 'span':
        displayType = 'inline-block';
      default:
        break;
    }
    for (let i = 0; roles.length > i; i += 1) {
      // superadmin
      if (this.currentUser.role === 'superadmin' && roles[i] === 'superadmin') {
        this.elRef.nativeElement.style.display = displayType;
      }
      // admin
      if (this.currentUser.role === 'admin' && roles[i] === 'admin') {
        this.elRef.nativeElement.style.display = displayType;
      }
      // user
      if (this.currentUser.role === 'user' && (roles[i] === 'user' || roles[i] === '')) {
        this.elRef.nativeElement.style.display = displayType;
      }
    }
  }
}

```

##### HTML markup
```html
<!-- userRole=[alias of users that can see the element] -->
<li userRole="superadmin,admin"><a routerLink="['/main']">Users</a></li>
```

Whenever it's possible to set accessible routes - use AuthGuard. Following snippets checks user role before accessing route.
To keep currentUser state in service class I'm fetching user data from localStorage and creating observable current user.
##### authcodeguard.guard.ts
```javascript
import {Injectable} from '@angular/core';
import {ActivatedRouteSnapshot, CanActivate, Router, RouterStateSnapshot} from '@angular/router';
import {Observable} from 'rxjs/Observable';
//service
import {UserService} from './service/user.service';

@Injectable()
export class AuthCodeGuard implements CanActivate {
  public currentUser;

  constructor(private login: UserService,
              private router: Router,
              private userService: UserService) {
  }

  canActivate(next: ActivatedRouteSnapshot,
              state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {

    this.currentUser = this.userService.currentUser;

      if (this.currentUser !== undefined) {

        // superadmin
        if (this.currentUser.role === 'superadmin') {
          if (next.params.org) { // organizations page
            return true;
          }
        }

        // admin
        if (this.currentUser.role === 'admin') {
          if (next.params.org) { // organizations page
            return true;
          }
        }

        // user
        if (this.currentUser.role === 'user') {
          if (next.params.org) { // organizations page
            this.router.navigate(['/main']);
            return false;
          }
        }
      }
      return true;

  }
}
```

##### app.module.ts
```javascript
  imports: [
    BrowserModule,
    FormsModule,
    PasswordStrengthBarModule,
    HttpModule,
    OrderModule,
    ReactiveFormsModule,
    NgbModule.forRoot(),
    RouterModule.forRoot([
      {
        path: '',
        component: LoginComponent
      },
      {
        path: 'login',
        component: LoginComponent
      },
      {
        path: 'activation',
        //canActivate: [AuthGuard],
        component: ActivationComponent
      },
      {
        path: 'verification',
        canActivate: [AuthGuard],
        component: VerificationComponent
      },
      {
        path: 'forgotpassword',
        component: ForgotPasswordComponent
      },
      {
        path: 'resetpassword',
        component: ResetPasswordComponent
      },
      {
        path: 'main',
        canActivate: [AuthCodeGuard],
        component: MainComponent
      },
      {
        path: 'main/:org',
        canActivate: [AuthCodeGuard],
        component: OrganizationsComponent
      },
      {
        path: 'main/:org/:id',
        canActivate: [AuthCodeGuard],
        component: OrganizationUsersComponent
      },
      {
        path: '**',
        redirectTo: '/login'
      }
    ])
  ]
 ```

