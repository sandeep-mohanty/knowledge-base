# Role-Based Access Control (RBAC) in Angular: The Complete Guide

Role-Based Access Control (RBAC) in Angular means driving what users can see and do (routes, components, buttons, API calls) from their roles and permissions instead of ad-hoc conditionals scattered across the app. A robust RBAC setup in Angular typically combines JWT-based authentication, centralized authorization services, route guards, and structural directives for hiding or disabling UI elements.

***

## RBAC fundamentals in Angular

RBAC centers on mapping users → roles → permissions and enforcing that mapping in both backend and frontend.

- Roles: Coarse-grained labels such as `admin`, `manager`, `user`, `viewer`.  
- Permissions: Specific actions like `order.create`, `user.update`, `report.view`.  
- Principle: The backend remains the authority for role/permission assignment; the Angular app consumes these as claims in JWT or via an API.

Example (simplified mapping):

```ts
export type Role = 'Admin' | 'Manager' | 'User';

export const ROLE_PERMISSIONS: Record<Role, string[]> = {
  Admin:   ['user.create', 'user.update', 'user.delete', 'order.create', 'order.view'],
  Manager: ['order.create', 'order.view'],
  User:    ['order.view'],
};
```

For production systems, this mapping usually lives on the server, and the Angular app just reads granted roles/permissions.

***

## Authentication and getting roles/permissions

Most real apps use JWT-based authentication and encode roles/permissions in the token or fetch them post-login.

Example JWT payload:

```json
{
  "sub": "12345",
  "name": "Jane Doe",
  "roles": ["Admin", "Manager"],
  "permissions": ["order.create", "order.view"],
  "iat": 1672345678,
  "exp": 1672945678
}
```

A basic Angular `AuthService` decodes the token and exposes current user info:

```ts
// auth.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

export interface CurrentUser {
  id: string;
  name: string;
  roles: string[];
  permissions: string[];
}

@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly userSubject = new BehaviorSubject<CurrentUser | null>(null);
  readonly user$ = this.userSubject.asObservable();

  get user(): CurrentUser | null {
    return this.userSubject.value;
  }

  get isAuthenticated(): boolean {
    return !!this.userSubject.value;
  }

  login(token: string): void {
    const payload = this.decodeJwt(token);
    const user: CurrentUser = {
      id: payload.sub,
      name: payload.name,
      roles: payload.roles ?? [],
      permissions: payload.permissions ?? [],
    };
    this.userSubject.next(user);
    localStorage.setItem('access_token', token);
  }

  logout(): void {
    this.userSubject.next(null);
    localStorage.removeItem('access_token');
  }

  private decodeJwt(token: string): any {
    const [, payload] = token.split('.');
    return JSON.parse(atob(payload));
  }
}
```

Centralizing current user data like this simplifies downstream RBAC checks.

---

## Core RBAC service: `AuthorizationService`

A dedicated service encapsulates “can this user do X?” logic, avoiding duplicated role checks across components and guards.

```ts
// authorization.service.ts
import { inject, Injectable } from '@angular/core';
import { AuthService } from './auth.service';

@Injectable({ providedIn: 'root' })
export class AuthorizationService {
  private readonly auth = inject(AuthService);

  hasRole(role: string | string[]): boolean {
    const user = this.auth.user;
    if (!user) return false;

    const roles = Array.isArray(role) ? role : [role];
    return roles.some(r => user.roles.includes(r));
  }

  hasPermission(permission: string | string[]): boolean {
    const user = this.auth.user;
    if (!user) return false;

    const perms = Array.isArray(permission) ? permission : [permission];
    return perms.some(p => user.permissions.includes(p));
  }

  isInRoleAndCan(role: string, permission: string): boolean {
    return this.hasRole(role) && this.hasPermission(permission);
  }
}
```

This becomes the single point of truth for authorization checks on the client.

---

## Route-level RBAC with guards

Angular route guards are the first line for protecting whole pages or feature areas.

### Role-based route guard (functional style)

Using functional guards with `inject` keeps the code concise and testable:

```ts
// role.guard.ts
import { inject } from '@angular/core';
import {
  ActivatedRouteSnapshot,
  CanActivateFn,
  Router,
  RouterStateSnapshot,
} from '@angular/router';
import { AuthService } from './auth.service';
import { AuthorizationService } from './authorization.service';

export const roleGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const auth = inject(AuthService);
  const authorization = inject(AuthorizationService);
  const router = inject(Router);

  const requiredRoles = route.data['roles'] as string[] | undefined;

  if (!auth.isAuthenticated) {
    return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
  }

  if (requiredRoles && !authorization.hasRole(requiredRoles)) {
    return router.createUrlTree(['/forbidden']);
  }

  return true;
};
```

Route configuration with `data.roles`:

```ts
// app.routes.ts
import { Routes } from '@angular/router';
import { roleGuard } from './role.guard';

export const routes: Routes = [
  {
    path: 'admin',
    canActivate: [roleGuard],
    data: { roles: ['Admin'] },
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent),
  },
  {
    path: 'orders',
    canActivate: [roleGuard],
    data: { roles: ['Admin', 'Manager'] },
    loadComponent: () => import('./orders/orders.component').then(m => m.OrdersComponent),
  },
];
```

This pattern keeps route-level rules declarative and maintainable.

***

## Permission-based guards for fine-grained control

For larger systems, it is often better to check permissions rather than roles at the route level.

```ts
// permission.guard.ts
import { inject } from '@angular/core';
import {
  ActivatedRouteSnapshot,
  CanActivateFn,
  Router,
  RouterStateSnapshot,
} from '@angular/router';
import { AuthService } from './auth.service';
import { AuthorizationService } from './authorization.service';

export const permissionGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const auth = inject(AuthService);
  const authorization = inject(AuthorizationService);
  const router = inject(Router);

  const requiredPermissions = route.data['permissions'] as string[] | undefined;

  if (!auth.isAuthenticated) {
    return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
  }

  if (requiredPermissions && !authorization.hasPermission(requiredPermissions)) {
    return router.createUrlTree(['/forbidden']);
  }

  return true;
};
```

Usage in routes:

```ts
export const routes: Routes = [
  {
    path: 'orders/new',
    canActivate: [permissionGuard],
    data: { permissions: ['order.create'] },
    loadComponent: () =>
      import('./orders/new-order.component').then(m => m.NewOrderComponent),
  },
];
```

This allows backend to evolve role→permission mapping without changing front-end route definitions.

***

## RBAC in the template: structural directives

Guarding routes is not enough; the UI must also hide or disable unauthorized elements.

### `appHasPermission` structural directive

A custom directive can show or hide content based on permissions:

```ts
// has-permission.directive.ts
import {
  Directive,
  Input,
  TemplateRef,
  ViewContainerRef,
} from '@angular/core';
import { AuthorizationService } from './authorization.service';

@Directive({
  selector: '[appHasPermission]',
  standalone: true,
})
export class HasPermissionDirective {
  private permission: string | string[] | null = null;

  constructor(
    private tpl: TemplateRef<unknown>,
    private vcr: ViewContainerRef,
    private authz: AuthorizationService,
  ) {}

  @Input()
  set appHasPermission(value: string | string[]) {
    this.permission = value;
    this.updateView();
  }

  private updateView(): void {
    this.vcr.clear();

    if (this.permission && this.authz.hasPermission(this.permission)) {
      this.vcr.createEmbeddedView(this.tpl);
    }
  }
}
```

Usage in templates:

```html
<button *appHasPermission="'order.create'" (click)="createOrder()">
  Create order
</button>

<div *appHasPermission="['user.update', 'user.delete']">
  <button (click)="editUser()">Edit</button>
  <button (click)="deleteUser()">Delete</button>
</div>
```

A similar directive can be built for roles (`appHasRole`) that reuses `AuthorizationService.hasRole`.

***

## Combining guards and directives for full coverage

A complete Angular RBAC setup typically combines several layers:

- Route guards (`authGuard`, `roleGuard`, `permissionGuard`) for page-level access  
- Structural directives for conditionally rendering or disabling UI elements  
- A shared `AuthorizationService` to ensure all checks are consistent and testable  
- Backend enforcement (e.g., Spring Security, ASP.NET Core, Node middleware) as the ultimate source of truth

High-level example:

```ts
// auth.guard.ts – authentication-only
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (!auth.isAuthenticated) {
    return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
  }

  return true;
};
```

```ts
// routes.ts – combining guards
export const routes: Routes = [
  {
    path: 'dashboard',
    canActivate: [authGuard],
    loadComponent: () => import('./dashboard.component').then(m => m.DashboardComponent),
  },
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard],
    data: { roles: ['Admin'] },
    loadComponent: () => import('./admin.component').then(m => m.AdminComponent),
  },
  {
    path: 'orders/new',
    canActivate: [authGuard, permissionGuard],
    data: { permissions: ['order.create'] },
    loadComponent: () => import('./new-order.component').then(m => m.NewOrderComponent),
  },
];
```

```html
<!-- template examples -->
<a routerLink="/admin" *appHasPermission="'user.manage'">Admin Area</a>
<button *appHasPermission="'order.create'">Create Order</button>
```

This layered approach keeps authorization concerns explicit and consistent across navigation and UI.

---

## Practical tips and pitfalls

Some practical guidelines for production-ready RBAC in Angular:

- Keep role/permission decisions in a dedicated authorization service, not scattered `if (user.role === 'Admin')` checks.  
- Prefer permissions over raw roles on the frontend for large systems, so backend can change mappings without UI churn.  
- Do not trust client-side checks alone; always re-check permissions on the server before executing sensitive operations.  
- Cache permissions (e.g., in memory) but handle token expiration and logout carefully.  
- For complex scenarios, consider a client-side authorization library like CASL or `ngx-permissions` to model abilities declaratively.

Used together, these patterns give a maintainable, testable RBAC setup that cleanly separates authentication, authorization, routing, and UI concerns in Angular.
