# 6 Angular Multi Provider Patterns That Replace Ugly Conditionals

Functional multi providers in Angular let multiple implementations contribute to a single token so the app can iterate or dispatch over them instead of using large `if/else` or `switch` chains. This is ideal for plugin-style behavior, strategy selection, and conditional logic that should be open for extension but closed for modification.

***

## 1. Problem: “ugly conditionals” in Angular

In many Angular apps, complex behavior is wired with large conditional blocks:

```ts
export class PaymentService {
  constructor(private http: HttpClient) {}

  pay(order: Order, method: 'card' | 'wallet' | 'cash'): Observable<void> {
    if (method === 'card') {
      // card flow
    } else if (method === 'wallet') {
      // wallet flow
    } else if (method === 'cash') {
      // cash flow
    } else {
      throw new Error('Unsupported method');
    }
  }
}
```

Issues:

- Violates the Open/Closed Principle: adding a new method means editing this service.  
- Hard to test each branch in isolation.  
- Spreads decision logic and dependencies across the class.

Multi providers offer a pattern where each “branch” becomes its own service, registered under the same token and selected at runtime.

***

## 2. Primer: Angular multi providers

Angular’s DI lets multiple providers contribute values to the same token when `multi: true` is set.

- Without `multi: true`, later providers override earlier ones.  
- With `multi: true`, Angular collects all provider values into an array and injects that array.

Example:

```ts
export const INTERCEPTOR_TOKEN = new InjectionToken<Interceptor[]>('interceptors');

providers: [
  { provide: INTERCEPTOR_TOKEN, useClass: AuthInterceptor,    multi: true },
  { provide: INTERCEPTOR_TOKEN, useClass: LoggingInterceptor, multi: true },
  { provide: INTERCEPTOR_TOKEN, useClass: RetryInterceptor,   multi: true },
];
```

Injecting `INTERCEPTOR_TOKEN` yields an array of all registered interceptors.

The same pattern powers `HTTP_INTERCEPTORS` and other extensible APIs.

***

## 3. Pattern 1: Strategy selection via multi providers

Goal: replace `if/else` on payment method with plug-in strategies.

### 3.1 Define a strategy interface and token

```ts
// payment-strategy.ts
import { InjectionToken } from '@angular/core';
import { Observable } from 'rxjs';
import { Order } from './order.model';

export interface PaymentStrategy {
  readonly method: string;
  pay(order: Order): Observable<void>;
}

export const PAYMENT_STRATEGIES = new InjectionToken<PaymentStrategy[]>(
  'PAYMENT_STRATEGIES',
);
```

### 3.2 Implement each strategy as a service

```ts
// card-payment.strategy.ts
import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';
import { PaymentStrategy } from './payment-strategy';
import { Order } from './order.model';

@Injectable()
export class CardPaymentStrategy implements PaymentStrategy {
  readonly method = 'card';

  pay(order: Order): Observable<void> {
    // concrete card logic
    return of(void 0);
  }
}

// wallet-payment.strategy.ts
@Injectable()
export class WalletPaymentStrategy implements PaymentStrategy {
  readonly method = 'wallet';

  pay(order: Order): Observable<void> {
    // concrete wallet logic
    return of(void 0);
  }
}
```

### 3.3 Register them as multi providers

```ts
// app.config.ts or app.module.ts
import { ApplicationConfig } from '@angular/core';
import { PAYMENT_STRATEGIES } from './payment-strategy';
import { CardPaymentStrategy } from './card-payment.strategy';
import { WalletPaymentStrategy } from './wallet-payment.strategy';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: PAYMENT_STRATEGIES, useClass: CardPaymentStrategy,   multi: true },
    { provide: PAYMENT_STRATEGIES, useClass: WalletPaymentStrategy, multi: true },
  ],
};
```

### 3.4 Dispatcher service with minimal conditionals

```ts
// payment.service.ts
import { inject, Injectable } from '@angular/core';
import { PAYMENT_STRATEGIES } from './payment-strategy';
import { Order } from './order.model';
import { Observable, throwError } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class PaymentService {
  private readonly strategies = inject(PAYMENT_STRATEGIES);

  pay(order: Order, method: string): Observable<void> {
    const strategy = this.strategies.find(s => s.method === method);

    if (!strategy) {
      return throwError(() => new Error(`Unsupported payment method: ${method}`));
    }

    return strategy.pay(order);
  }
}
```

Benefits:

- Add new payment types by adding new providers, not editing `PaymentService`.  
- Test each strategy in isolation.  
- This is effectively the Strategy Pattern implemented with multi providers.

***

## 4. Pattern 2: Pluggable feature toggles

Avoid giant `if (flagA) { ... } if (flagB) { ... }` blocks scattered across bootstrapping code.

### 4.1 Define feature hook and token

```ts
// feature-hook.ts
import { InjectionToken } from '@angular/core';

export interface FeatureHook {
  id: string;
  run(): void | Promise<void>;
}

export const FEATURE_HOOKS = new InjectionToken<FeatureHook[]>(
  'FEATURE_HOOKS',
);
```

### 4.2 Implement hook per feature

```ts
// logging-feature.hook.ts
import { Injectable } from '@angular/core';
import { FeatureHook } from './feature-hook';

@Injectable()
export class LoggingFeatureHook implements FeatureHook {
  id = 'logging';

  run(): void {
    console.log('[Feature] Logging is enabled');
  }
}

// analytics-feature.hook.ts
@Injectable()
export class AnalyticsFeatureHook implements FeatureHook {
  id = 'analytics';

  async run(): Promise<void> {
    // async initialization
  }
}
```

### 4.3 Register hooks via multi providers

```ts
providers: [
  { provide: FEATURE_HOOKS, useClass: LoggingFeatureHook,   multi: true },
  { provide: FEATURE_HOOKS, useClass: AnalyticsFeatureHook, multi: true },
];
```

### 4.4 Runner that executes enabled hooks

```ts
// feature-runner.service.ts
import { inject, Injectable } from '@angular/core';
import { FEATURE_HOOKS } from './feature-hook';
import { FeatureFlagsService } from './feature-flags.service';

@Injectable({ providedIn: 'root' })
export class FeatureRunnerService {
  private readonly hooks = inject(FEATURE_HOOKS);
  private readonly flags = inject(FeatureFlagsService);

  async runEnabledHooks(): Promise<void> {
    for (const hook of this.hooks) {
      if (this.flags.isEnabled(hook.id)) {
        await hook.run();
      }
    }
  }
}
```

Conditions are centralized in the flag service, while hooks stay open for extension.

***

## 5. Pattern 3: HTTP interceptors as a chain

Angular’s `HTTP_INTERCEPTORS` token is the canonical multi provider example for chaining behaviors without explicit conditionals.

```ts
import { HTTP_INTERCEPTORS } from '@angular/common/http';

providers: [
  { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor,    multi: true },
  { provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true },
  { provide: HTTP_INTERCEPTORS, useClass: RetryInterceptor,   multi: true },
];
```

- Each interceptor contributes to the same token, and Angular builds a runtime chain in registration order.  
- You can add or remove interceptors without modifying a central `switch` block.

This pattern generalizes to custom pipelines such as “request processors” or “event handlers” using your own multi token.

***

## 6. Pattern 4: Component plug-ins via multi providers

Imagine a generic table that needs per-domain behavior; instead of wiring many inputs and conditionals inside the generic component, use multi providers and an InjectionToken.

### 6.1 Define a table plugin interface and token

```ts
// table-plugin.ts
import { InjectionToken } from '@angular/core';

export interface TablePlugin {
  id: string;
  apply<T>(rows: T[]): T[];
}

export const TABLE_PLUGINS = new InjectionToken<TablePlugin[]>(
  'TABLE_PLUGINS',
);
```

### 6.2 Generic table component injects plugins

```ts
// generic-table.component.ts
import { Component, inject, input } from '@angular/core';
import { TABLE_PLUGINS, TablePlugin } from './table-plugin';

@Component({
  selector: 'app-generic-table',
  standalone: true,
  template: `
    <table>
      <tr *ngFor="let row of processedData">{{ row | json }}</tr>
    </table>
  `,
})
export class GenericTableComponent<T> {
  data = input<T[]>([]);
  private readonly plugins = inject(TABLE_PLUGINS, { optional: true }) ?? [];

  get processedData(): T[] {
    return this.plugins.reduce(
      (rows, plugin) => plugin.apply(rows),
      this.data() ?? [],
    );
  }
}
```

### 6.3 Host-specific plugins via multi providers

```ts
// user-table-plugin.ts
import { Injectable } from '@angular/core';
import { TablePlugin } from './table-plugin';
import { User } from './user.model';

@Injectable()
export class ActiveUsersOnlyPlugin implements TablePlugin {
  id = 'active-users-only';

  apply(rows: User[]): User[] {
    return rows.filter(u => u.isActive);
  }
}
```

```ts
// user-table-wrapper.component.ts
import { Component } from '@angular/core';
import { TABLE_PLUGINS } from './table-plugin';
import { ActiveUsersOnlyPlugin } from './user-table-plugin';
import { GenericTableComponent } from './generic-table.component';

@Component({
  selector: 'app-user-table',
  standalone: true,
  imports: [GenericTableComponent],
  template: `<app-generic-table [data]="users"></app-generic-table>`,
  providers: [
    { provide: TABLE_PLUGINS, useClass: ActiveUsersOnlyPlugin, multi: true },
  ],
})
export class UserTableComponent {
  users = [/* ... */];
}
```

The generic table has no `if (userMode) ...` logic; host components plug behavior in declaratively.

***

## 7. Pattern 5: Environment/feature configuration as multi values

Multi providers can also aggregate primitive values like strings or config fragments.

### 7.1 Config token

```ts
// app-config.ts
import { InjectionToken } from '@angular/core';

export interface FeatureConfig {
  key: string;
  value: unknown;
}

export const FEATURE_CONFIG = new InjectionToken<FeatureConfig[]>(
  'FEATURE_CONFIG',
);
```

### 7.2 Provide config fragments in different areas

```ts
// analytics.config.ts
providers: [
  {
    provide: FEATURE_CONFIG,
    useValue: { key: 'analytics.enabled', value: true },
    multi: true,
  },
];

// payments.config.ts
providers: [
  {
    provide: FEATURE_CONFIG,
    useValue: { key: 'payments.maxRetries', value: 3 },
    multi: true,
  },
];
```

### 7.3 Config service

```ts
// feature-config.service.ts
import { inject, Injectable } from '@angular/core';
import { FEATURE_CONFIG, FeatureConfig } from './app-config';

@Injectable({ providedIn: 'root' })
export class FeatureConfigService {
  private readonly entries = inject(FEATURE_CONFIG, { optional: true }) ?? [];

  get<T = unknown>(key: string, defaultValue?: T): T | undefined {
    const entry = this.entries.find(e => e.key === key);
    return (entry?.value as T | undefined) ?? defaultValue;
  }
}
```

No giant `switch` over environment or feature keys; the DI tree aggregates config fragments from different feature modules.

***

## 8. Pattern 6: Conditional provider sets (standalone APIs)

Angular’s modern `provideX` APIs allow grouping providers; multi tokens help compose sets without conditionals.

Example: provide a set of bootstrap tasks.

```ts
// bootstrap-task.ts
import { InjectionToken } from '@angular/core';

export interface BootstrapTask {
  run(): Promise<void> | void;
}

export const BOOTSTRAP_TASKS = new InjectionToken<BootstrapTask[]>(
  'BOOTSTRAP_TASKS',
);
```

```ts
// provideBootstrapTasks.ts
import { EnvironmentProviders, makeEnvironmentProviders } from '@angular/core';
import { BOOTSTRAP_TASKS } from './bootstrap-task';
import { InitIdentityTask } from './init-identity.task';
import { InitThemeTask } from './init-theme.task';

export function provideBootstrapTasks(): EnvironmentProviders {
  return makeEnvironmentProviders([
    { provide: BOOTSTRAP_TASKS, useClass: InitIdentityTask, multi: true },
    { provide: BOOTSTRAP_TASKS, useClass: InitThemeTask,    multi: true },
  ]);
}
```

Then in `appConfig`:

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideBootstrapTasks(),
    // other providers...
  ],
};
```

At runtime, a runner service injects `BOOTSTRAP_TASKS` and executes them instead of hardcoding a list of tasks in one place.

***

## 9. When to reach for multi providers

Multi providers shine when:

- There are many variants of a behavior (strategies, plugins, steps).  
- New variants should be added without editing a central conditional block.  
- Order either does not matter (iterate) or is controlled by provider ordering.

They are less suitable when:

- Only one implementation is ever required for a token.  
- The selection condition is not stable or cannot be expressed as a simple property on the implementation.

A pragmatic guideline:

- Model branching behavior as interfaces and tokens.  
- Register implementations via multi providers.  
- Let small dispatcher services handle minimal selection logic instead of embedding conditionals deep in business code.

This keeps modules open to extension by adding providers, while avoiding “ugly conditionals” that inevitably grow over time.
