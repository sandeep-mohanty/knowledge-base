# 10 Advanced Performance Tips You Might Be Missing
**Unlocking Angular’s Hidden Speed Boosters**

![Angular Performance Tips Header](https://miro.medium.com/v2/resize:fit:720/format:webp/0*1bUaH3wRMuAvJrxQ)  

Angular is a powerful framework — but power alone doesn’t guarantee speed. If your app feels sluggish, it’s not Angular’s fault. It’s yours. And mine. Because most developers (my past self included) stick to the basics: lazy loading, Ahead-of-Time (AOT) compilation, and maybe a few change detection tweaks.

But Angular has layers. And if you dig deeper, you’ll find performance tricks that go beyond the obvious. These are the kinds of optimizations that separate good apps from great ones — apps that feel snappy, responsive, and effortless.

So if you’re ready to level up, here are 10 Angular performance tips you probably didn’t know — but definitely should.

---

### 1. 🧠 Use trackBy in *ngFor Like Your App Depends on It
Angular’s default change detection re-renders entire lists when data changes — even if only one item was updated. That’s a performance killer.

Using `trackBy` tells Angular how to uniquely identify items, so it only updates what’s necessary.

```html
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
```

```typescript
trackById(index: number, item: any): number {
  return item.id;
}
```
This tiny change can make a huge difference in large lists.

---

### 2. 🚀 Avoid ngStyle and ngClass in Hot Paths
Dynamic styling with `ngStyle` and `ngClass` is convenient—but expensive. Angular evaluates them on every change detection cycle.

Instead, use static class bindings or toggle classes manually in your component logic.

```html
<div [class.active]="isActive"></div>
```
Avoid binding entire style objects unless absolutely necessary.

---

### 3. 🧹 Clean Up Subscriptions with takeUntil
Memory leaks don’t just slow down your app — they can crash it. If you’re subscribing to observables, make sure you unsubscribe properly.

Use `takeUntil` with a `Subject` to clean up when your component is destroyed.

```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.getData()
    .pipe(takeUntil(this.destroy$))
    .subscribe(data => this.data = data);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```
It’s cleaner, safer, and scalable.

---

### 4. 🧭 Use ChangeDetectionStrategy.OnPush Everywhere You Can
Angular’s default change detection checks everything. That’s fine for small apps — but a nightmare for performance.

Switching to `OnPush` tells Angular to only check components when their inputs change.



```typescript
@Component({
  selector: 'app-fast',
  templateUrl: './fast.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
```
Combine this with immutable data patterns and you’ll see a noticeable speed boost.

---

### 5. 🧊 Freeze Your Data (Yes, Really)
Immutable data structures play nicely with `OnPush`. But you can go further: freeze your objects to prevent accidental mutations.

```typescript
const frozenData = Object.freeze({ name: 'Angular', version: 17 });
```
This isn’t just about performance — it’s about predictability. Frozen data helps Angular detect changes more efficiently and avoids subtle bugs.

---

### 6. 🧵 Use Web Workers for Heavy Computation
Angular runs in the main thread. If you’re doing heavy calculations — sorting, filtering, parsing — you’re blocking the UI.

Offload that work to a Web Worker.

```typescript
const worker = new Worker('./my-worker.worker', { type: 'module' });
worker.postMessage(data);
worker.onmessage = ({ data }) => {
  this.processedData = data;
};
```
Angular CLI supports Web Workers out of the box. Use them.

---

### 7. 📦 Tree-Shake Your Modules
Angular’s modular architecture is great — but unused modules can sneak into your bundle.

Use `providedIn: 'root'` for services to enable tree-shaking.

```typescript
@Injectable({
  providedIn: 'root'
})
export class MyService {}
```
Also, audit your imports. Don’t import entire libraries when you only need one function.

---

### 8. 🧮 Memoize Expensive Calculations
If you’re doing the same calculation multiple times, cache the result. Use memoization libraries like `memo-decorator` or write your own simple cache.

```typescript
private cache = new Map();

getExpensiveResult(input: string): string {
  if (this.cache.has(input)) return this.cache.get(input);
  const result = computeHeavy(input);
  this.cache.set(input, result);
  return result;
}
```
This is especially useful in pipes and computed properties.

---

### 9. 🧪 Profile with Angular DevTools (Not Just Chrome)
Chrome DevTools is great — but Angular DevTools gives you framework-specific insights. You can inspect change detection cycles, component trees, and performance bottlenecks.

Install it from the Chrome Web Store and start profiling your app like a pro.

---

### 10. 🧭 Preload Strategically, Not Blindly
Lazy loading is great — but sometimes you want to preload routes for better UX. Angular’s `PreloadAllModules` strategy loads everything—which defeats the purpose.

Instead, create a custom preloading strategy:

```typescript
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data && route.data['preload'] ? load() : of(null);
  }
}
```
Then mark routes with `data: { preload: true }`. Smart preloading = faster navigation.

---

### Final Thoughts: Performance Is a Mindset
Optimizing Angular isn’t about one magic trick — it’s about adopting a mindset. Every line of code, every component, every service should be written with performance in mind.

The tips above aren’t just hacks — they’re habits. And once you internalize them, you’ll start building apps that feel fast by default.

So go beyond the basics. Dig into the framework. Challenge your assumptions. And make Angular fly.