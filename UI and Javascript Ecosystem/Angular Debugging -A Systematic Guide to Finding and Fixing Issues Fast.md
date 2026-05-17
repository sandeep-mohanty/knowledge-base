# Angular Debugging: A Systematic Guide to Finding and Fixing Issues Fast

> *The difference between a junior and a senior developer isn't that seniors write fewer bugs. It's that seniors have a systematic process for finding them, and they've seen enough failure patterns to know where to look first.*

---

## Why Most Angular Debugging Takes Too Long

The typical debugging session looks like this: something isn't working, a developer stares at the code for a few minutes, adds some `console.log` statements, refreshes the browser, changes something that seems related, refreshes again, checks Stack Overflow, changes something else. An hour passes. The bug is fixed — but the developer isn't entirely sure why.

This approach has a name: **shotgun debugging**. It works eventually, but it's expensive, unreproducible, and teaches you nothing transferable. The next similar bug takes just as long.

Angular bugs cluster into predictable categories. Change detection not triggering. Signals not propagating. Subscriptions leaking or never firing. Lifecycle hooks running in the wrong order. Template bindings silently failing. Knowing these categories — and having a systematic process for each — turns hours-long debugging sessions into minutes.

This guide covers the full Angular debugging toolkit: the mental process, the tools, the specific failure patterns, and the code-level techniques that experienced Angular developers use to diagnose issues systematically.

---

## The Debugging Mental Model: Hypothesize, Not Guess

There's a meaningful difference between guessing and forming a hypothesis.

**Guessing:** "Maybe it's the service. Let me change it and see."

**Hypothesizing:** "The UI isn't updating after the API call returns. That means either the data isn't arriving, or the data is arriving but change detection isn't seeing it. I'll check the network tab first to rule out the data issue, then check whether the component uses OnPush."

A hypothesis is testable. It tells you what to look at and what outcome would confirm or refute it. When the outcome doesn't match, you learn something — you've eliminated one possibility and narrowed the search space. Guessing doesn't narrow anything.

Before touching any code, answer three questions:

1. **What is the exact expected behavior?** Be specific. "The user list should update when I click Save" is better than "it should work."
2. **What is actually happening?** Equally specific. "The list doesn't update at all" or "the list updates but shows the old data for 2 seconds."
3. **When did it last work?** If you can identify a recent commit where it worked, `git bisect` will find the breaking change in minutes.

These three questions force you to understand the problem before you touch the code. Surprisingly often, the act of answering them precisely reveals the bug before any tooling is needed.

---

## The Angular Debugging Toolkit

### Angular DevTools

The single most powerful debugging tool available to Angular developers, and the most underused. Install it as a Chrome/Firefox extension and it adds an Angular panel to your browser DevTools.

**What it shows you:**

- The complete component tree with their state
- Which components triggered change detection and why
- How long change detection took for each component
- The current value of every `@Input`, `@Output`, signal, and injected service in a selected component

**Practical use:** when the UI isn't updating, open Angular DevTools, select the component, and check whether its data has actually changed. If the data is correct in DevTools but the template shows stale values, you have a change detection problem. If the data is wrong, the bug is upstream — in the service, the store, or the API call.

```typescript
// You can also access component instances programmatically in the console
// Select a component in Angular DevTools, then in console:
$ng0  // Reference to the selected component instance
$ng0.users  // Inspect any property
$ng0.loadUsers()  // Call any method
```

### Source Maps and Breakpoints

`console.log` debugging has its place, but breakpoints are faster for complex issues. Ensure source maps are enabled (they are by default in development builds) and use the browser's Sources panel to set breakpoints directly in your TypeScript files.

```json
// angular.json — verify source maps are enabled for development
{
 "configurations": {
   "development": {
     "sourceMap": true,  // TypeScript source maps
     "optimization": false
   }
 }
}
```

Conditional breakpoints are particularly useful for debugging loops and lists:

```
Right-click breakpoint → Edit breakpoint → Condition: user.id === '42'
```

This pauses execution only when a specific condition is true, avoiding the need to click Continue hundreds of times.

### The `ng` Global Debug Object

In development mode, Angular exposes a global `ng` object in the browser console with powerful inspection capabilities:

```javascript
// In browser console (development mode only)

// Get a component instance from a DOM element
const el = document.querySelector('app-user-list');
const component = ng.getComponent(el);
component.users  // Inspect component state

// Manually trigger change detection on a component
ng.applyChanges(component);

// Get injected services from a component's injector
const userService = ng.getContext(el);

// Inspect the host element's component
ng.getOwningComponent(document.querySelector('.user-card'));

// Get all listeners on an element
ng.getListeners(el);
```

This is invaluable for inspecting component state without adding temporary `console.log` statements to the source code.

### Augury (for Older Angular Projects)

For projects still on older Angular versions where Angular DevTools has limited support, Augury provides similar component tree inspection capabilities.

---

## The 15 Most Common Angular Bugs and How to Diagnose Each

### 1. UI Not Updating After Data Changes

**Symptom:** an API call returns, the data arrives, but the template still shows stale values.

**Diagnosis process:**

First, verify the data actually arrived. Add a temporary log or use Angular DevTools to inspect the component's properties. If the data is wrong, the bug is in the data layer, not change detection.

If the data is correct but the UI is stale, check the component's change detection strategy:

```typescript
// Is ChangeDetectionStrategy.OnPush in use?
@Component({
 changeDetection: ChangeDetectionStrategy.OnPush,
 // ...
})
```

With `OnPush`, Angular only re-renders when:
- An `@Input` reference changes (not deep mutation)
- An `async` pipe receives a new emission
- A signal updates
- `markForCheck()` or `detectChanges()` is called manually

**The mutation trap:** this is the most common cause of stale UI with OnPush:

```typescript
// BROKEN: Mutating the array — same reference, OnPush doesn't detect change
this.users.push(newUser);

// FIXED: New array reference — OnPush sees the change
this.users = [...this.users, newUser];

// BROKEN: Mutating an object property
this.user.name = 'Alice';

// FIXED: New object reference
this.user = { ...this.user, name: 'Alice' };
```

**With Signals:** signals handle this correctly by design — updating a signal always notifies dependents. But if you're mixing signals with regular properties:

```typescript
// Signal update triggers re-render correctly
this.users.update(current => [...current, newUser]);

// But if a template reads a regular property alongside signals,
// the regular property won't trigger re-render on OnPush
this.regularProperty = 'new value'; // Template won't update without markForCheck()
```

**Fix for legitimate cases where you can't avoid mutation:**

```typescript
constructor(private cdr: ChangeDetectorRef) {}

updateUser(user: User): void {
   this.users.push(user); // Mutation for some reason
   this.cdr.markForCheck(); // Tell Angular to re-check this component
}
```

---

### 2. ExpressionChangedAfterItHasBeenCheckedError

**Symptom:** the browser console shows `ExpressionChangedAfterItHasBeenCheckedError` in development mode. The application often still works, making developers dismiss this as a false alarm. It is not. It indicates a real architectural problem that will cause subtle bugs in production.

**What it means:** during Angular's change detection cycle, a value was read, change detection completed, and then the value changed again before the cycle fully ended. Angular runs a second verification pass in development mode to catch exactly this.

**Common causes:**

```typescript
// CAUSE 1: Setting a property inside ngAfterViewInit that a parent template reads
@Component({
 template: `<child [title]="childTitle"></child>`
})
export class ParentComponent implements AfterViewInit {
 childTitle = '';

 @ViewChild(ChildComponent) child!: ChildComponent;

 ngAfterViewInit() {
   // WRONG: This runs after change detection, but changes data that
   // was already checked in the current cycle
   this.childTitle = this.child.computedTitle;
 }
}

// FIX: Defer the assignment to the next cycle
ngAfterViewInit() {
   Promise.resolve().then(() => {
       this.childTitle = this.child.computedTitle;
   });
   // Or use a signal, which handles this correctly:
   // this.childTitle.set(this.child.computedTitle);
}
```

```typescript
// CAUSE 2: Functions in templates that return different values each call
@Component({
 template: `<div>{{ getCurrentTime() }}</div>`  // Returns new Date() every call
})
```

Functions in templates are called multiple times per change detection cycle. If a function returns a new object or array reference each time, or if its return value changes between calls (like `new Date()`), this error will fire.

**Fix:** compute the value in the component and bind to a property, or use a pipe with memoization.

---

### 3. Memory Leaks From Unsubscribed Observables

**Symptom:** the application gradually slows down over time. Memory usage climbs in the browser DevTools performance tab. Events fire multiple times after navigating away from and back to a route.

**Diagnosis:** the multiple-firing symptom is the clearest signal. If clicking a button causes an action to run twice, then three times on the next visit to the route, you have a subscription accumulating with each navigation.

```typescript
// THE LEAK: subscription recreated on every component initialization
// but never cleaned up
@Component({ /* ... */ })
export class UserListComponent implements OnInit {
   users: User[] = [];

   constructor(private userService: UserService) {}

   ngOnInit(): void {
       // New subscription added every time this component is created
       // Old subscriptions are never cancelled
       this.userService.users$.subscribe(users => {
           this.users = users;
       });
   }
}
```

**Systematic fixes, in order of preference:**

```typescript
// BEST: async pipe — Angular manages subscription lifecycle automatically
@Component({
 template: `
   @for (user of users$ | async; track user.id) {
     <app-user-card [user]="user" />
   }
 `
})
export class UserListComponent {
   users$ = inject(UserService).users$;
   // No ngOnInit, no manual subscription, no cleanup needed
}
```

```typescript
// GOOD: takeUntilDestroyed (Angular 16+) — cleanest manual subscription pattern
@Component({ /* ... */ })
export class UserListComponent implements OnInit {
   private destroyRef = inject(DestroyRef);
   users: User[] = [];

   ngOnInit(): void {
       inject(UserService).users$
           .pipe(takeUntilDestroyed(this.destroyRef))
           .subscribe(users => this.users = users);
       // Automatically unsubscribed when component is destroyed
   }
}
```

```typescript
// ACCEPTABLE: Manual unsubscription for cases where you need it
export class UserListComponent implements OnInit, OnDestroy {
   private subscription = new Subscription();
   users: User[] = [];

   ngOnInit(): void {
       this.subscription.add(
           this.userService.users$.subscribe(users => this.users = users)
       );
       this.subscription.add(
           this.routeService.params$.subscribe(params => this.loadUser(params.id))
       );
       // Group multiple subscriptions — one unsubscribe clears all
   }

   ngOnDestroy(): void {
       this.subscription.unsubscribe();
   }
}
```

**Detecting leaks in production:** use the browser's Memory tab in DevTools. Take a heap snapshot, navigate away from a component, take another snapshot, and compare. Objects from the navigated-away component should not appear in the second snapshot. If they do, something is keeping them referenced — usually a subscription that was never cleaned up.

---

### 4. Signals Not Updating Dependents

**Symptom:** a signal's value changes, but computed signals or template expressions that depend on it don't update.

**Diagnosis:**

```typescript
@Component({
 template: `<div>{{ fullName() }}</div>`
})
export class ProfileComponent {
   firstName = signal('Alice');
   lastName = signal('Smith');

   // CORRECT: computed — automatically tracks signal reads
   fullName = computed(() => `${this.firstName()} ${this.lastName()}`);

   updateName(): void {
       this.firstName.set('Bob'); // fullName updates automatically
   }
}
```

The most common signal debugging mistake is reading a signal value outside of a reactive context:

```typescript
// BUG: Reading the signal value at construction time, not reactively
export class MyComponent {
   data = signal<User[]>([]);

   // This captures the value ONCE at initialization — never updates
   count = this.data().length;  // Not reactive!

   // CORRECT: computed reads the signal reactively
   count = computed(() => this.data().length);
}
```

Another subtle issue — mutating signal values without using the signal's update mechanism:

```typescript
// BUG: Mutating the array inside a signal doesn't trigger updates
this.users().push(newUser);  // Signal's array is mutated but signal itself isn't "set"

// CORRECT: Use update() which reads current value and sets new one
this.users.update(current => [...current, newUser]);

// ALSO CORRECT for simple replacement
this.users.set([...this.users(), newUser]);
```

---

### 5. HTTP Requests Not Firing

**Symptom:** an Observable from `HttpClient` is created but the API call never appears in the Network tab.

**Root cause:** Observables are lazy. Creating an Observable does nothing — it only executes when subscribed.

```typescript
// BUG: Observable created but never subscribed — no HTTP request fires
loadUsers(): void {
   this.http.get<User[]>('/api/users'); // This does nothing
}

// FIX: Subscribe to trigger execution
loadUsers(): void {
   this.http.get<User[]>('/api/users').subscribe(users => {
       this.users.set(users);
   });
}

// BETTER: Use async pipe in template — cleaner, automatic cleanup
users$ = this.http.get<User[]>('/api/users');
// Template: @for (user of users$ | async; track user.id) { ... }

// BEST for reactive data with signals:
users = toSignal(this.http.get<User[]>('/api/users'), { initialValue: [] });
```

**Diagnosis:** if you're not sure whether the Observable is being subscribed to, add a `tap` operator temporarily:

```typescript
this.http.get<User[]>('/api/users').pipe(
   tap(data => console.log('HTTP response received:', data)),
   tap({ subscribe: () => console.log('Subscription created') })
).subscribe(/* ... */);
```

---

### 6. Race Conditions in Async Operations

**Symptom:** search results from an older request appear after results from a newer request, showing stale data. Or duplicate API calls fire for the same operation.

```typescript
// BUG: If user types quickly, older responses can arrive after newer ones
search(term: string): void {
   this.http.get<Product[]>(`/api/products?q=${term}`).subscribe(results => {
       this.results = results; // Might be overwritten by an older in-flight request
   });
}
```

```typescript
// CORRECT: switchMap cancels previous in-flight request
@Component({ /* ... */ })
export class SearchComponent {
   private searchTerm$ = new Subject<string>();

   results$ = this.searchTerm$.pipe(
       debounceTime(300),        // Wait for user to stop typing
       distinctUntilChanged(),    // Don't search if term hasn't changed
       switchMap(term =>          // Cancel previous request, start new one
           this.http.get<Product[]>(`/api/products?q=${term}`).pipe(
               catchError(() => of([]))  // Handle errors without breaking the stream
           )
       )
   );

   onSearch(term: string): void {
       this.searchTerm$.next(term);
   }
}
```

**RxJS operator guide for race condition scenarios:**

| Operator | Behavior | Use When |
|---|---|---|
| `switchMap` | Cancels previous, starts new | Search, navigation, latest-wins scenarios |
| `concatMap` | Queues — waits for previous to complete | Sequential operations where order matters |
| `exhaustMap` | Ignores new while previous is active | Form submission (prevent double-submit) |
| `mergeMap` | Runs all concurrently | Parallel operations where all results needed |

---

### 7. Lifecycle Hook Timing Issues

**Symptom:** accessing a `@ViewChild` or `@ContentChild` in `ngOnInit` returns `undefined`. Data isn't available when you expect it. Child component methods called from parent don't work.

**Angular lifecycle execution order:**

```
constructor()
↓
ngOnChanges()        ← @Input values available
↓
ngOnInit()           ← Component initialized, but child components NOT yet rendered
↓
ngDoCheck()
↓
ngAfterContentInit() ← <ng-content> projected content available
↓
ngAfterContentChecked()
↓
ngAfterViewInit()    ← @ViewChild and @ContentChild now available
↓
ngAfterViewChecked()
↓
ngOnDestroy()        ← Clean up subscriptions here
```

```typescript
@Component({
 template: `<canvas #myChart></canvas>`
})
export class ChartComponent implements OnInit, AfterViewInit {

   @ViewChild('myChart') canvasRef!: ElementRef;

   ngOnInit(): void {
       // WRONG: canvasRef is undefined here — DOM not rendered yet
       this.initChart(this.canvasRef.nativeElement); // Error!
   }

   ngAfterViewInit(): void {
       // CORRECT: canvasRef is available here
       this.initChart(this.canvasRef.nativeElement);
   }
}
```

---

### 8. Circular Dependency Errors

**Symptom:** `NullInjectorError` or `Cannot read property of undefined` when a service is injected. The application fails to bootstrap with a cryptic error about circular dependencies.

**Diagnosis:** Angular's injector cannot instantiate a service if constructing it requires another service that in turn requires the first service.

```
ServiceA constructor requires ServiceB
ServiceB constructor requires ServiceA
Angular cannot resolve either — circular dependency
```

**Detection:**

```bash
# Build with verbose output to expose circular dependencies
ng build --source-map
# Then analyze with:
npx source-map-explorer dist/main.js
# Or use:
npx madge --circular --extensions ts src/
```

**Fix strategies:**

```typescript
// Option 1: Introduce a shared service that both depend on
// ServiceA and ServiceB both depend on SharedDataService
// SharedDataService depends on neither

// Option 2: Use forwardRef() for cases where one service
// genuinely needs to reference the other
@Injectable({ providedIn: 'root' })
export class ServiceA {
   constructor(
       @Inject(forwardRef(() => ServiceB)) private serviceB: ServiceB
   ) {}
}

// Option 3: Restructure — usually the best long-term answer
// Ask: why does ServiceA need ServiceB? Can that responsibility
// be moved to a calling component or a shared third service?
```

---

## Structured Logging: Making Console Output Useful

Most developers use `console.log` as an afterthought. Structured logs make debugging dramatically faster when you or a colleague needs to trace an issue.

```typescript
// BAD: No context, no structure
console.log('test');
console.log(data);
console.log('error:', e);

// GOOD: Structured, contextual, grouped
console.group('UserService.loadUser()');
console.log('Input:', { userId });
console.time('API call');
this.http.get<User>(`/api/users/${userId}`).pipe(
   tap(user => {
       console.timeEnd('API call');
       console.log('Response:', user);
       console.groupEnd();
   }),
   catchError(error => {
       console.timeEnd('API call');
       console.error('Failed:', { userId, status: error.status, message: error.message });
       console.groupEnd();
       return throwError(() => error);
   })
).subscribe(/* ... */);
```

**Create a debug utility that strips out in production:**

```typescript
// core/utils/debug.util.ts
const isDevMode = !environment.production;

export const debug = {
   log: (context: string, data: unknown) => {
       if (isDevMode) console.log(`[${context}]`, data);
   },
   warn: (context: string, data: unknown) => {
       if (isDevMode) console.warn(`[${context}]`, data);
   },
   table: (context: string, data: unknown[]) => {
       if (isDevMode) {
           console.group(context);
           console.table(data);
           console.groupEnd();
       }
   }
};

// Usage
debug.log('UserService.loadUser', { userId, timestamp: Date.now() });
```

---

## Template Debugging Techniques

### The Performance Pipe Trap

Functions called in templates are invoked on every change detection cycle — potentially hundreds of times per second:

```html
<!-- BAD: getFullName() is called on every change detection cycle -->
<div>{{ getFullName(user) }}</div>

<!-- BAD: filter() creates a new array on every cycle -->
<div *ngFor="let item of getActiveItems()">{{ item.name }}</div>
```

```typescript
// DETECT template function calls causing performance issues
// Add this temporarily to find offending functions:
getFullName(user: User): string {
   console.count('getFullName called'); // Will show how many times per second
   return `${user.firstName} ${user.lastName}`;
}
```

**Fix: move computation to component properties or pure pipes:**

```typescript
// Move to a computed signal — calculated only when inputs change
fullName = computed(() => `${this.user().firstName} ${this.user().lastName}`);

// Or use a pure pipe — Angular caches the result when input doesn't change
@Pipe({ name: 'fullName', pure: true })
export class FullNamePipe implements PipeTransform {
   transform(user: User): string {
       return `${user.firstName} ${user.lastName}`;
   }
}
```

### Debugging Binding Errors

```html
<!-- Cannot read property 'name' of undefined -->
<!-- SYMPTOM: user is null/undefined at template evaluation time -->
<div>{{ user.name }}</div>

<!-- FIX: Optional chaining -->
<div>{{ user?.name }}</div>

<!-- FIX: Null guard with @if -->
@if (user) {
 <div>{{ user.name }}</div>
}

<!-- FIX: Default value -->
<div>{{ user?.name ?? 'Loading...' }}</div>
```

---

## The Systematic Debugging Process

Translate the mental model into a concrete checklist:

**Step 1: Characterize precisely**
- Write down expected vs. actual behavior in one sentence each
- Identify when it last worked (check git log)
- Determine if it's reproducible consistently or intermittently

**Step 2: Check the error layer first (2 minutes)**
- Browser console: errors and warnings
- Network tab: failed requests, unexpected status codes, malformed responses
- Angular DevTools: component state

**Step 3: Form a specific hypothesis**
- "I think the problem is [X] because [Y]"
- "If I'm right, I should see [Z] when I check [W]"

**Step 4: Test the hypothesis with minimal intervention**
- Check Angular DevTools before touching code
- Use the `ng` global object to inspect state
- Set a breakpoint rather than adding console.log

**Step 5: Narrow the blast radius**
- Comment out unrelated code
- Create a minimal reproduction — if you can reproduce the bug in a fresh component with minimal code, you understand it

**Step 6: Fix and verify**
- Implement the minimal fix that addresses the root cause
- Test the original scenario
- Test edge cases: empty data, null values, error states, rapid user interaction
- Check that no adjacent behavior regressed

---

## Production Debugging: When You Can't Use DevTools

Some bugs only appear in production. Source maps aren't available. Angular DevTools doesn't run. You're working from error reports and logs.

**Set up structured error logging:**

```typescript
// core/services/error-tracking.service.ts
@Injectable({ providedIn: 'root' })
export class ErrorTrackingService implements ErrorHandler {

   handleError(error: Error): void {
       console.error('Unhandled error:', {
           message: error.message,
           stack: error.stack,
           timestamp: new Date().toISOString(),
           url: window.location.href,
           userAgent: navigator.userAgent
       });

       // Send to your error tracking service (Sentry, Datadog, etc.)
       this.sendToErrorTracker(error);
   }

   private sendToErrorTracker(error: Error): void {
       // Integration with Sentry, Rollbar, Datadog, etc.
   }
}

// Provide as the global error handler
// app.config.ts
export const appConfig: ApplicationConfig = {
   providers: [
       { provide: ErrorHandler, useClass: ErrorTrackingService }
   ]
};
```

**Enable source maps in production (carefully):**

```json
// angular.json — hidden source maps allow you to read them without
// exposing them to end users via browser DevTools
{
 "configurations": {
   "production": {
     "sourceMap": {
       "scripts": true,
       "hidden": true  // Maps exist but aren't linked in the bundle
     }
   }
 }
}
```

Upload the hidden source maps to your error tracking service. Errors reported from production will include readable stack traces that reference your TypeScript source, not the minified bundle.

---

## Summary: The Debugging Mindset

The difference between debugging taking 10 minutes and debugging taking 3 hours is almost never knowledge of the codebase. It's almost always process.

A systematic debugger:
- Characterizes the problem precisely before touching the code
- Forms a falsifiable hypothesis about the cause
- Tests the hypothesis with the minimum intervention
- Uses the right tool for the right problem (Angular DevTools for state, Network tab for HTTP, breakpoints for logic flow)
- Understands the Angular-specific failure categories — change detection, subscription lifecycle, signal reactivity, lifecycle hook timing — and checks them systematically rather than randomly

The bugs that take hours aren't the complex ones. They're the simple ones that were never properly characterized at the start. Most Angular bugs, once you've looked in the right place, are one-line fixes. The skill is knowing where to look.