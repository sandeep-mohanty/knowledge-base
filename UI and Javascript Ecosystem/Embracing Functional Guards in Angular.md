# Stop Using Class-Based Guards: Embracing Functional Guards in Angular

Functional route guards are the new, recommended way to protect and control navigation in Angular; they replace most uses of class-based guards with plain functions that use `inject()` for DI.  They reduce boilerplate, are easier to compose, and align with Angular’s broader “functional-first” direction (signals, standalone APIs, functional effects, etc.).

## 1. What guards are and what changed

Route guards are pieces of logic that decide whether navigation can continue, should be cancelled, or should be redirected.  Historically they were implemented as classes implementing interfaces like `CanActivate`, `CanDeactivate`, `Resolve`, etc., and registered in providers.

From Angular 15+, these class-based guards are deprecated in favor of functional guards:

- The new types are `CanActivateFn`, `CanMatchFn`, `CanDeactivateFn`, `ResolveFn`, etc.
- A guard is now just a function `(route, state) => result`, and it can use `inject()` to get services.
- Class-based guards still work for now, but Angular’s own docs and router blog posts push developers toward the functional style.

***

## 2. Classic class-based guard vs functional guard

### 2.1 Class-based `CanActivate` example (deprecated style)

```ts
// auth.guard.ts (class-based, deprecated style)
import { Injectable } from '@angular/core';
import {
  CanActivate,
  Router,
  ActivatedRouteSnapshot,
  RouterStateSnapshot,
  UrlTree,
} from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';

@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private auth: AuthService,
    private router: Router,
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot,
  ): boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree> {
    if (this.auth.isAuthenticated()) {
      return true;
    }

    // Redirect to login with returnUrl
    return this.router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url },
    });
  }
}
```

Route configuration:

```ts
// app.routes.ts
import { Routes } from '@angular/router';
import { AuthGuard } from './auth.guard';

export const routes: Routes = [
  {
    path: 'account',
    canActivate: [AuthGuard],
    loadComponent: () =>
      import('./account/account.component').then((m) => m.AccountComponent),
  },
];
```

### 2.2 Functional `CanActivateFn` version

```ts
// auth.guard.ts (functional)
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url },
  });
};
```

Route configuration:

```ts
// app.routes.ts
import { Routes } from '@angular/router';
import { authGuard } from './auth.guard';

export const routes: Routes = [
  {
    path: 'account',
    canActivate: [authGuard],
    loadComponent: () =>
      import('./account/account.component').then((m) => m.AccountComponent),
  },
];
```

Key differences:

- No `@Injectable()` class or constructor; just a function and `inject()` calls.
- Same return types: boolean, `UrlTree`, observable, or promise.
- Less boilerplate and easier to read for simple, stateless logic.

***

## 3. Why “functional future”: motivations and benefits

Angular’s move toward functional guards is driven by several factors.

- **Less boilerplate**
  - No guard classes, decorators, or DI plumbing code.
  - A guard can be a single function in the same file as the route.

- **Better composability and reuse**
  - Guards are just functions, which you can:
    - Compose (call one guard from another).
    - Curry or partially apply to inject configuration.
  - Guards can be defined inline on routes where logic is simple.

- **More flexible configuration**
  - Class and token-based guards are less configurable, less reusable, and cannot be defined inline compared with functional ones.[8]
  - Functional guards work naturally with standalone APIs and functional router configuration.

- **Still fully integrated with DI**
  - `inject()` gives access to Angular DI in any guard function.
  - No need for a class purely to get DI.

- **Alignment with new Angular design direction**
  - Similar patterns exist for functional interceptors, signals, effects, and standalone components.
  - The router’s newer documentation emphasizes that functional guards are more lightweight and ergonomic.

***

## 4. Detailed functional guard examples

### 4.1 Basic auth + role guard with `inject`

```ts
// permissions.guard.ts
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const adminGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (!auth.isAuthenticated()) {
    return router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url },
    });
  }

  if (!auth.hasRole('admin')) {
    return router.createUrlTree(['/forbidden']);
  }

  return true;
};
```

Usage:

```ts
export const routes: Routes = [
  {
    path: 'admin',
    canActivate: [adminGuard],
    loadComponent: () =>
      import('./admin/admin.component').then((m) => m.AdminComponent),
  },
];
```

This pattern matches common IAM modeling while leveraging functional guards.

### 4.2 Inline functional guard for very local logic

Because a guard is just a function, you can define local behavior directly on the route.

```ts
import { inject } from '@angular/core';
import { Routes, CanActivateFn } from '@angular/router';
import { FeatureFlagsService } from './feature-flags.service';

const featureEnabledGuard: CanActivateFn = () => {
  const features = inject(FeatureFlagsService);
  return features.isEnabled('newDashboard');
};

export const routes: Routes = [
  {
    path: 'new-dashboard',
    canActivate: [featureEnabledGuard],
    loadComponent: () =>
      import('./new-dashboard/new-dashboard.component').then(
        (m) => m.NewDashboardComponent,
      ),
  },
];
```

Or fully inline:

```ts
export const routes: Routes = [
  {
    path: 'new-dashboard',
    canActivate: [() => inject(FeatureFlagsService).isEnabled('newDashboard')],
    loadComponent: () =>
      import('./new-dashboard/new-dashboard.component').then(
        (m) => m.NewDashboardComponent,
      ),
  },
];
```

This is much harder to do ergonomically with class-based guards, where each piece would require its own class.

### 4.3 Functional `CanDeactivateFn` example

Angular 15+ provides functional `CanDeactivateFn` for unsaved-changes checks.

```ts
// pending-changes.guard.ts
import { CanDeactivateFn } from '@angular/router';
import { EditProfileComponent } from './edit-profile.component';

export const pendingChangesGuard: CanDeactivateFn<EditProfileComponent> = (
  component,
  currentRoute,
  currentState,
  nextState,
) => {
  if (component.hasUnsavedChanges()) {
    return confirm(
      'You have unsaved changes. Do you really want to leave?',
    );
  }

  return true;
};
```

Route registration:

```ts
export const routes: Routes = [
  {
    path: 'profile/edit',
    canDeactivate: [pendingChangesGuard],
    loadComponent: () =>
      import('./edit-profile.component').then((m) => m.EditProfileComponent),
  },
];
```

### 4.4 Functional `ResolveFn` example

```ts
// user.resolver.ts
import { inject } from '@angular/core';
import { ResolveFn } from '@angular/router';
import { UserService } from './user.service';
import { User } from './user.model';

export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;
  return userService.getUser(id); // observable or promise
};
```

Route:

```ts
export const routes: Routes = [
  {
    path: 'users/:id',
    resolve: {
      user: userResolver,
    },
    loadComponent: () =>
      import('./user-detail.component').then((m) => m.UserDetailComponent),
  },
];
```

This preserves resolver behavior while removing class boilerplate.

***

## 5. Migrating from class-based to functional guards

Angular provides patterns and helpers to migrate.

### 5.1 Direct rewrite

Given a class-based guard:

```ts
@Injectable({ providedIn: 'root' })
export class OldGuard implements CanActivate {
  constructor(private auth: AuthService) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot) {
    return this.auth.isAuthenticated();
  }
}
```

Rewrite as:

```ts
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  return auth.isAuthenticated();
};
```

Then update routes from `[OldGuard]` to `[authGuard]`.[1][3]

### 5.2 Adapter pattern using a legacy class

If you want to keep classes but conform to the functional configuration, delegate from a function to the existing class.[7]

```ts
@Injectable({ providedIn: 'root' })
export class LegacyAuthGuard {
  constructor(private auth: AuthService) {}

  canActivate() {
    return this.auth.isAuthenticated();
  }
}
```

Use an adapter in routes:

```ts
import { inject } from '@angular/core';

export const routes: Routes = [
  {
    path: 'account',
    canActivate: [() => inject(LegacyAuthGuard).canActivate()],
    loadComponent: () =>
      import('./account/account.component').then((m) => m.AccountComponent),
  },
];
```

This lets you keep tested class-based logic while adopting functional configuration.

***

## 6. When to use which style in practice

A pragmatic stance for modern Angular apps is nuanced rather than absolutist.

- Prefer **functional guards** for:
  - New Angular 15+ applications, especially standalone- and signals-based architectures.
  - Guards that are mostly stateless and depend directly on services.
  - Inline, route-specific logic that belongs close to route configuration.

- Keep **class-based guards** (possibly wrapped by functional adapters) when:
  - You have a large, existing codebase with many shared guard classes.
  - You value constructor DI for explicit dependency signatures and discoverability.
  - You need a transition period where you do not want to touch well-tested guard classes.

A solid hybrid for IAM-heavy systems is:

1. Put complex authorization logic into dedicated domain services (e.g., `AuthorizationService.canAccessResource(user, resource, action)`).
2. Keep guards as thin, functional wrappers that:
   - Inject the domain service.
   - Delegate the decision.
   - Translate results into `true`, `false`, or a `UrlTree`.

This approach aligns with Angular’s functional router direction while keeping your authorization model clean, testable, and framework-agnostic.

