# When @Input() and @Output() Become a Mess: Cleaner Alternatives


![Angular Input Output Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*NWQvvOC_gKOSfrspzlIWhA.png)

If you’ve been working with Angular for a while, you’ve likely run into this:
1. A parent component sends data down to a child with `@Input()`.
2. The child emits changes back up with `@Output()`.
3. Then the parent passes that data to another sibling, and suddenly your simple app turns into a **spaghetti mess of bindings**.

In small apps, `@Input()` and `@Output()` work fine. But in real-world enterprise Angular apps, these decorators can quickly become unmaintainable. Let’s break down **why this happens**—and explore **cleaner alternatives** that I’ve successfully used in large projects.

## The Problem: Too Many Inputs & Outputs
Imagine a **dashboard with multiple widgets**.

* The `DashboardComponent` passes filters to `ChartComponent`, `TableComponent`, and `StatsComponent`.
* Each child emits changes back to the parent.
* The parent then re-broadcasts to other siblings.

**The result?**
* **Nested Input hell** → multiple bindings in HTML
* **Output chaos** → event bubbling everywhere
* **Tight coupling** → hard to reuse components elsewhere

```html
// dashboard.component.html
<app-filter [filters]="filters" (filtersChanged)="onFiltersChanged($event)"></app-filter>
<app-chart [filters]="filters"></app-chart>
<app-table [filters]="filters"></app-table>
```

```typescript
// dashboard.component.ts
onFiltersChanged(newFilters: Filter) {
  this.filters = newFilters;
}
```

Looks simple? Wait until you have **5+ siblings**, each talking to each other indirectly through the parent. Debugging becomes a nightmare.

---

## Cleaner Alternatives

### 1. Shared Services with RxJS
Instead of chaining inputs/outputs, create a **shared service** that holds state in an RxJS `BehaviorSubject` or `Signal`.

```typescript
@Injectable({ providedIn: 'root' })
export class FilterService {

  private filtersSubject = new BehaviorSubject<Filter>({}); 
 
  filters$ = this.filtersSubject.asObservable();

  updateFilters(newFilters: Filter) {
    this.filtersSubject.next(newFilters);
  }
}
```

Now each component can **subscribe or update** directly:

```typescript
// filter.component.ts
this.filterService.updateFilters(updated);

// chart.component.ts
this.filterService.filters$.subscribe(filters => {
  this.loadData(filters);
});
```

✅ **Pros:**
* Decoupled components
* Cleaner templates
* Easy to scale

⚠️ **Cons:**
* Can lead to too many services if overused
* Requires RxJS knowledge

---

### 2. Signals (Angular 17+)
With Signals, Angular now has **reactivity without subscriptions**. Perfect for replacing `@Input()` + `@Output()`.

```typescript
@Injectable({ providedIn: 'root' })
export class FilterStore {

  filters = signal<Filter>({});

  updateFilters(newFilters: Filter) {
    this.filters.set(newFilters);
  }
}
```

In your components:

```typescript
// filter.component.ts
this.filterStore.updateFilters(updated);

// chart.component.ts
effect(() => {
  const filters = this.filterStore.filters();
  this.loadData(filters);
});
```

✅ **Pros:**
* No need for `subscribe/unsubscribe`
* Automatic reactivity
* Cleaner state management

⚠️ **Cons:**
* Still new → best for Angular 17+ projects

---

### 3. NgRx or Signal-Based Store
For **large-scale apps** where data flows everywhere, adopting a **global store** makes sense.

* Use **NgRx** if you already need advanced features like effects, entity adapters, devtools.
* Use **Signal-based stores** if you want lighter boilerplate but still need shared state.

✅ **Pros:**
* Single source of truth
* Debuggable with devtools
* Great for enterprise-scale apps

⚠️ **Cons:**
* Overkill for small/medium projects
* Learning curve for newcomers

---

### 4. Content Projection & Smart/Dumb Components
Sometimes the issue isn’t the data flow — it’s **bad component design**.

* Use **dumb/presentational components** that only display data.
* Use **smart/container components** to manage state & logic.

Example: Instead of pushing data to every child, project content using `<ng-content>` and let the parent handle logic.

---

## When to Use What
* **Simple app / 1–2 child components** → Stick to `@Input()` & `@Output()`.
* **Medium app with sibling communication** → Shared service with RxJS or Signals.
* **Large app / enterprise scale** → NgRx or Signal-based global store.
* **Complex UIs** → Mix in content projection and smart/dumb separation.

---

## Final Thoughts
`@Input()` and `@Output()` are great tools—but only until your app grows beyond a certain complexity. Once you notice too much “prop drilling” or endless event bubbling, it’s time to rethink.

In my experience:
* **For Angular 16 and below** → RxJS services are the cleanest solution.
* **For Angular 17 and above** → Signals are a game-changer.
* **For enterprise projects** → NgRx or Signal-based stores bring scalability and debugging power.

The takeaway: **Don’t force `@Input()` and `@Output()` everywhere. Angular now gives you better tools—use them!**