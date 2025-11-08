# Angular Signals: The Complete Tutorial

## Introduction

Angular Signals represent a revolutionary shift in how Angular handles reactivity and state management. Introduced in Angular 16 and stabilized in Angular 17, Signals provide a **simpler, more performant alternative** to traditional change detection and RxJS for synchronous state management.[1][2][3]

### What Are Signals?

A Signal is a **reactive primitive** that wraps a value and notifies Angular when that value changes. Think of it as a smart container that:[2][4]

- Holds a value (primitive, object, or array)
- Tracks who reads the value
- Notifies consumers when the value changes
- Enables fine-grained, efficient DOM updates

**Traditional Approach (Without Signals)**:

```typescript
@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count }}</h2>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count: number = 0;
  
  increment(): void {
    this.count++;
    // Angular checks entire component tree for changes
  }
}
```

**Modern Approach (With Signals)**:

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count() }}</h2>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);
  
  increment(): void {
    this.count.set(this.count() + 1);
    // Angular knows exactly what changed and updates only that
  }
}
```

### Why Signals Matter

**Performance Benefits**:[1][2]

1. **Fine-Grained Reactivity**: Angular knows exactly what changed and updates only affected parts
2. **No Zone.js Required**: Future Angular versions will support zone-less change detection
3. **Optimized Re-rendering**: Only components using changed signals are re-rendered
4. **Reduced Overhead**: No need to check entire component tree

**Developer Experience Benefits**:[3][1]

1. **Simpler Than RxJS**: Easier mental model for synchronous state
2. **Type-Safe**: Full TypeScript support
3. **Declarative**: Clear dependency relationships
4. **Less Boilerplate**: No manual subscriptions/unsubscriptions
5. **Better Debugging**: Clear signal dependency graph

## Core Concepts

### 1. Writable Signals

Writable signals are mutable containers you can update directly.[2]

#### Creating Writable Signals

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-demo',
  template: `
    <h3>Name: {{ name() }}</h3>
    <h3>Age: {{ age() }}</h3>
    <h3>Is Active: {{ isActive() }}</h3>
    <h3>User: {{ user().name }} - {{ user().email }}</h3>
  `
})
export class DemoComponent {
  // Primitive values
  name = signal<string>('John Doe');
  age = signal<number>(30);
  isActive = signal<boolean>(true);
  
  // Objects
  user = signal({
    id: 1,
    name: 'Alice',
    email: 'alice@example.com'
  });
  
  // Arrays
  items = signal<string[]>(['Item 1', 'Item 2', 'Item 3']);
}
```

#### Reading Signal Values

To read a signal's value, **call it as a function**:[1][2]

```typescript
export class ReadingSignalsComponent {
  count = signal(0);
  
  ngOnInit(): void {
    // Read the signal value
    console.log('Current count:', this.count());  // Output: 0
    
    // Use in logic
    if (this.count() > 10) {
      console.log('Count is high!');
    }
  }
}
```

**Important**: The parentheses `()` are crucial. `this.count` is the signal object, `this.count()` returns the value.

#### Updating Signal Values

**Method 1: Using `set()`**:[2][1]

Replace the entire value:

```typescript
export class UpdateSignalsComponent {
  count = signal(0);
  user = signal({ name: 'John', age: 30 });
  
  updateWithSet(): void {
    // Set primitive value
    this.count.set(5);
    
    // Set object (replaces entire object)
    this.user.set({ name: 'Jane', age: 25 });
  }
}
```

**Method 2: Using `update()`**:[1][2]

Compute new value from current value:

```typescript
export class UpdateSignalsComponent {
  count = signal(0);
  items = signal<string[]>(['Item 1']);
  user = signal({ name: 'John', age: 30 });
  
  updateWithUpdate(): void {
    // Increment based on current value
    this.count.update(current => current + 1);
    
    // Add item to array
    this.items.update(current => [...current, 'New Item']);
    
    // Update object property
    this.user.update(current => ({
      ...current,
      age: current.age + 1
    }));
  }
}
```

**When to Use Each**:

- **`set()`**: When you have a new value independent of the current value
- **`update()`**: When the new value depends on the current value

#### Complete Example: Todo Counter

```typescript
import { Component, signal } from '@angular/core';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

@Component({
  selector: 'app-todo-list',
  standalone: true,
  template: `
    <div class="todo-container">
      <h2>Todo List</h2>
      
      <input 
        #todoInput 
        type="text" 
        placeholder="Add new todo"
        (keyup.enter)="addTodo(todoInput.value); todoInput.value = ''">
      
      <button (click)="addTodo(todoInput.value); todoInput.value = ''">
        Add Todo
      </button>
      
      <ul>
        @for (todo of todos(); track todo.id) {
          <li>
            <input 
              type="checkbox" 
              [checked]="todo.completed"
              (change)="toggleTodo(todo.id)">
            <span [class.completed]="todo.completed">
              {{ todo.title }}
            </span>
            <button (click)="removeTodo(todo.id)">Delete</button>
          </li>
        }
      </ul>
      
      <div class="stats">
        <p>Total: {{ todos().length }}</p>
        <p>Completed: {{ completedCount() }}</p>
        <p>Remaining: {{ remainingCount() }}</p>
      </div>
    </div>
  `,
  styles: [`
    .completed {
      text-decoration: line-through;
      color: #999;
    }
    .todo-container {
      max-width: 600px;
      margin: 20px auto;
      padding: 20px;
    }
    li {
      display: flex;
      gap: 10px;
      padding: 10px;
      border-bottom: 1px solid #eee;
    }
  `]
})
export class TodoListComponent {
  todos = signal<Todo[]>([
    { id: 1, title: 'Learn Angular Signals', completed: false },
    { id: 2, title: 'Build an app', completed: false }
  ]);
  
  private nextId = 3;
  
  addTodo(title: string): void {
    if (!title.trim()) return;
    
    this.todos.update(current => [
      ...current,
      { id: this.nextId++, title: title.trim(), completed: false }
    ]);
  }
  
  toggleTodo(id: number): void {
    this.todos.update(current =>
      current.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }
  
  removeTodo(id: number): void {
    this.todos.update(current => 
      current.filter(todo => todo.id !== id)
    );
  }
  
  // Computed signals (explained next)
  completedCount = computed(() => 
    this.todos().filter(t => t.completed).length
  );
  
  remainingCount = computed(() => 
    this.todos().filter(t => !t.completed).length
  );
}
```

### 2. Computed Signals

Computed signals derive their value from other signals. They automatically update when dependencies change.[4][2][1]

#### Creating Computed Signals

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-computed-demo',
  template: `
    <h3>First Name: {{ firstName() }}</h3>
    <h3>Last Name: {{ lastName() }}</h3>
    <h3>Full Name: {{ fullName() }}</h3>
    
    <h3>Price: ${{ price() }}</h3>
    <h3>Quantity: {{ quantity() }}</h3>
    <h3>Total: ${{ total() }}</h3>
    <h3>With Tax: ${{ totalWithTax() }}</h3>
  `
})
export class ComputedDemoComponent {
  // Source signals
  firstName = signal('John');
  lastName = signal('Doe');
  
  // Computed signal depends on firstName and lastName
  fullName = computed(() => {
    return `${this.firstName()} ${this.lastName()}`;
  });
  
  // Shopping cart example
  price = signal(10);
  quantity = signal(2);
  
  // Computed: total price
  total = computed(() => this.price() * this.quantity());
  
  // Computed from another computed signal
  totalWithTax = computed(() => this.total() * 1.1);
}
```

#### Key Characteristics of Computed Signals

**1. Read-Only**:[2][1]

```typescript
const count = signal(5);
const doubled = computed(() => count() * 2);

// ✅ Reading is allowed
console.log(doubled());  // 10

// ❌ Writing is NOT allowed - compilation error
doubled.set(20);  // Error: Property 'set' does not exist
```

**2. Lazily Evaluated**:[2]

Computed signals only calculate their value when read:

```typescript
const count = signal(0);

// Computation function does NOT run yet
const expensive = computed(() => {
  console.log('Computing...');
  return count() * 1000;
});

// Now it runs and logs "Computing..."
console.log(expensive());

// Reads cached value - does NOT log again
console.log(expensive());

// Update source signal
count.set(1);

// Next read triggers recomputation
console.log(expensive());  // Logs "Computing..." again
```

**3. Memoized (Cached)**:[2]

Computed signals cache their result until dependencies change:

```typescript
const numbers = signal([1, 2, 3, 4, 5]);

const filtered = computed(() => {
  console.log('Filtering array...');
  return numbers().filter(n => n > 2);
});

// First read: computation runs
console.log(filtered());  // Logs "Filtering array..." then [3, 4, 5]

// Second read: uses cached value
console.log(filtered());  // Just returns [3, 4, 5], no log

// Change source
numbers.set([1, 2, 3, 4, 5, 6]);

// Next read recomputes
console.log(filtered());  // Logs "Filtering array..." then [3, 4, 5, 6]
```

**4. Dynamic Dependencies**:[2]

Signal dependencies are determined at runtime:

```typescript
const showDetails = signal(false);
const userName = signal('John');
const userAge = signal(30);

const display = computed(() => {
  if (showDetails()) {
    // When showDetails is true, depends on userName AND userAge
    return `${userName()} is ${userAge()} years old`;
  } else {
    // When showDetails is false, depends ONLY on userName
    return userName();
  }
});

// Initially depends only on userName
console.log(display());  // "John"

// Changing userAge won't trigger recomputation
userAge.set(31);
console.log(display());  // Still "John"

// Enable details
showDetails.set(true);

// Now depends on both userName and userAge
console.log(display());  // "John is 31 years old"

// Now changing userAge WILL trigger recomputation
userAge.set(32);
console.log(display());  // "John is 32 years old"
```

#### Common Computed Signal Patterns

**Pattern 1: Data Transformation**:

```typescript
interface User {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
}

@Component({
  template: `
    <ul>
      @for (display of userDisplays(); track display.id) {
        <li>{{ display.name }} ({{ display.email }})</li>
      }
    </ul>
  `
})
export class UserListComponent {
  users = signal<User[]>([
    { id: 1, firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
    { id: 2, firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' }
  ]);
  
  // Transform data for display
  userDisplays = computed(() => 
    this.users().map(user => ({
      id: user.id,
      name: `${user.firstName} ${user.lastName}`,
      email: user.email
    }))
  );
}
```

**Pattern 2: Filtering and Sorting**:

```typescript
@Component({
  template: `
    <input [(ngModel)]="searchTerm" placeholder="Search...">
    <select [(ngModel)]="sortBy">
      <option value="name">Name</option>
      <option value="price">Price</option>
    </select>
    
    <ul>
      @for (product of filteredAndSortedProducts(); track product.id) {
        <li>{{ product.name }} - ${{ product.price }}</li>
      }
    </ul>
  `
})
export class ProductListComponent {
  products = signal([
    { id: 1, name: 'Laptop', price: 1200 },
    { id: 2, name: 'Mouse', price: 25 },
    { id: 3, name: 'Keyboard', price: 75 }
  ]);
  
  searchTerm = signal('');
  sortBy = signal<'name' | 'price'>('name');
  
  // Filter based on search
  filteredProducts = computed(() => {
    const term = this.searchTerm().toLowerCase();
    if (!term) return this.products();
    
    return this.products().filter(p => 
      p.name.toLowerCase().includes(term)
    );
  });
  
  // Sort filtered results
  filteredAndSortedProducts = computed(() => {
    const products = [...this.filteredProducts()];
    const sortKey = this.sortBy();
    
    return products.sort((a, b) => 
      a[sortKey] > b[sortKey] ? 1 : -1
    );
  });
}
```

**Pattern 3: Aggregations**:

```typescript
interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Component({
  template: `
    <h3>Cart Summary</h3>
    <p>Items: {{ itemCount() }}</p>
    <p>Subtotal: ${{ subtotal() }}</p>
    <p>Tax (10%): ${{ tax() }}</p>
    <p>Total: ${{ total() }}</p>
  `
})
export class CartComponent {
  cartItems = signal<CartItem[]>([
    { id: 1, name: 'Book', price: 20, quantity: 2 },
    { id: 2, name: 'Pen', price: 5, quantity: 5 }
  ]);
  
  itemCount = computed(() => 
    this.cartItems().reduce((sum, item) => sum + item.quantity, 0)
  );
  
  subtotal = computed(() => 
    this.cartItems().reduce((sum, item) => 
      sum + (item.price * item.quantity), 0
    )
  );
  
  tax = computed(() => this.subtotal() * 0.1);
  
  total = computed(() => this.subtotal() + this.tax());
}
```

#### Critical Pitfall: Conditional Signal Reading

**❌ WRONG - Broken Dependencies**:[1]

```typescript
const enabled = signal(false);
const count = signal(0);

// BAD: count() is only read in else branch
const result = computed(() => {
  if (enabled()) {
    return 'Enabled';
  } else {
    return count();  // Only read when enabled is false
  }
});

// Problem: When enabled is true, count is not a dependency
// Changing count won't trigger recomputation!
```

**✅ CORRECT - Always Read Dependencies**:[1]

```typescript
const enabled = signal(false);
const count = signal(0);

// GOOD: count() is always read
const result = computed(() => {
  const currentCount = count();  // Read unconditionally
  
  if (enabled()) {
    return 'Enabled';
  } else {
    return currentCount;
  }
});

// Now changing count always triggers recomputation
```

### 3. Effects

Effects are operations that run whenever signal dependencies change. Use them for **side effects**, not state derivation.[4][2]

#### Creating Effects

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-logger',
  template: `
    <h3>Count: {{ count() }}</h3>
    <button (click)="increment()">Increment</button>
  `
})
export class LoggerComponent {
  count = signal(0);
  
  constructor() {
    // Effect runs whenever count changes
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
    });
    
    // Effect runs at least once on creation
    // Output: "Count changed to: 0"
  }
  
  increment(): void {
    this.count.update(c => c + 1);
    // Effect runs again
    // Output: "Count changed to: 1"
  }
}
```

#### When to Use Effects

**✅ Good Use Cases**:[4][2]

1. **Logging and Analytics**:

```typescript
export class AnalyticsComponent {
  userId = signal<string | null>(null);
  currentPage = signal('/home');
  
  private analyticsEffect = effect(() => {
    const user = this.userId();
    const page = this.currentPage();
    
    if (user) {
      this.analyticsService.track('page_view', { user, page });
    }
  });
  
  constructor(private analyticsService: AnalyticsService) {}
}
```

2. **LocalStorage Synchronization**:

```typescript
export class PreferencesComponent {
  theme = signal<'light' | 'dark'>('light');
  fontSize = signal(16);
  
  constructor() {
    // Load from localStorage
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) {
      this.theme.set(savedTheme as 'light' | 'dark');
    }
    
    // Sync to localStorage
    effect(() => {
      localStorage.setItem('theme', this.theme());
    });
    
    effect(() => {
      localStorage.setItem('fontSize', this.fontSize().toString());
    });
  }
}
```

3. **Custom DOM Manipulation**:

```typescript
export class CustomChartComponent {
  @ViewChild('chartCanvas') canvas!: ElementRef<HTMLCanvasElement>;
  
  chartData = signal([10, 20, 30, 40]);
  
  constructor() {
    effect(() => {
      const data = this.chartData();
      this.renderChart(data);
    });
  }
  
  private renderChart(data: number[]): void {
    // Custom canvas/D3/Chart.js rendering
    const ctx = this.canvas.nativeElement.getContext('2d');
    // ... render logic
  }
}
```

4. **WebSocket/Server Sync**:

```typescript
export class RealtimeComponent {
  documentContent = signal('');
  
  constructor(private websocket: WebSocketService) {
    // Send changes to server
    effect(() => {
      const content = this.documentContent();
      this.websocket.send({ type: 'update', content });
    });
  }
}
```

#### When NOT to Use Effects

**❌ Don't Use for State Derivation**:[2]

```typescript
// ❌ BAD: Using effect for state derivation
firstName = signal('John');
lastName = signal('Doe');
fullName = signal('');

constructor() {
  effect(() => {
    // This can cause circular updates and errors!
    this.fullName.set(`${this.firstName()} ${this.lastName()}`);
  });
}

// ✅ GOOD: Use computed instead
firstName = signal('John');
lastName = signal('Doe');
fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
```

#### Effect Cleanup

Effects can perform cleanup when they run again or are destroyed:[2]

```typescript
export class TimerComponent {
  interval = signal(1000);
  
  constructor() {
    effect((onCleanup) => {
      const ms = this.interval();
      
      console.log(`Starting timer with ${ms}ms interval`);
      const timerId = setInterval(() => {
        console.log('Tick');
      }, ms);
      
      // Cleanup runs before next effect execution or on destroy
      onCleanup(() => {
        console.log('Cleaning up timer');
        clearInterval(timerId);
      });
    });
  }
}
```

**Real-World Example: Auto-Save**:

```typescript
export class DocumentEditorComponent {
  documentContent = signal('');
  private saveInProgress = signal(false);
  
  constructor(private api: ApiService) {
    effect((onCleanup) => {
      const content = this.documentContent();
      
      // Skip if no content
      if (!content) return;
      
      // Debounce auto-save
      const timeoutId = setTimeout(() => {
        this.saveInProgress.set(true);
        
        this.api.saveDocument(content).subscribe({
          next: () => {
            console.log('Document saved');
            this.saveInProgress.set(false);
          },
          error: (err) => {
            console.error('Save failed:', err);
            this.saveInProgress.set(false);
          }
        });
      }, 2000);  // Wait 2 seconds after last change
      
      // Cancel pending save if content changes again
      onCleanup(() => {
        clearTimeout(timeoutId);
      });
    });
  }
}
```

#### Effect Injection Context

Effects must be created in an injection context:[2]

**✅ In Constructor (Recommended)**:

```typescript
@Component({ /* ... */ })
export class MyComponent {
  count = signal(0);
  
  constructor() {
    effect(() => {
      console.log(this.count());
    });
  }
}
```

**✅ As Field Initializer**:

```typescript
@Component({ /* ... */ })
export class MyComponent {
  count = signal(0);
  
  // Named effect as field
  private loggingEffect = effect(() => {
    console.log(this.count());
  });
}
```

**✅ Outside Constructor with Injector**:

```typescript
@Component({ /* ... */ })
export class MyComponent {
  count = signal(0);
  private injector = inject(Injector);
  
  initializeEffects(): void {
    effect(() => {
      console.log(this.count());
    }, { injector: this.injector });
  }
}
```

#### Manual Effect Cleanup

```typescript
@Component({ /* ... */ })
export class MyComponent {
  count = signal(0);
  
  constructor() {
    const effectRef = effect(() => {
      console.log(this.count());
    }, { manualCleanup: true });
    
    // Manually destroy when needed
    setTimeout(() => {
      effectRef.destroy();
    }, 5000);
  }
}
```

## Signals vs Observables: The Complete Comparison

### Fundamental Differences

| Aspect | Signals | Observables |
|--------|---------|-------------|
| **Nature** | Synchronous, always has a value | Asynchronous streams, may not have value yet |
| **Evaluation** | Pull-based (read when needed) | Push-based (emit when ready) |
| **Memory** | Always contains current value | No inherent value, just stream |
| **Time** | Represents current state | Represents values over time |
| **Subscription** | Automatic (framework-managed) | Manual (subscribe/unsubscribe) |
| **Use Case** | Synchronous UI state | Asynchronous operations, events |
| **Learning Curve** | Simple, intuitive | Steeper (operators, subscription management) |
| **Operators** | Limited (computed, effect) | Rich (100+ RxJS operators) |
| **Cancellation** | N/A (synchronous) | Built-in (unsubscribe) |
| **Multiple Values** | Single value at a time | Multiple values over time |

### When to Use Signals

**✅ Use Signals For**:[5][3]

**1. Local Component State**:

```typescript
@Component({
  template: `
    <input [(ngModel)]="searchQuery" />
    <p>Searching for: {{ searchQuery() }}</p>
  `
})
export class SearchComponent {
  // ✅ Perfect for signals
  searchQuery = signal('');
  isLoading = signal(false);
  selectedItems = signal<string[]>([]);
}
```

**2. Derived/Computed State**:

```typescript
@Component({
  template: `
    <p>{{ firstName() }} {{ lastName() }}</p>
    <p>{{ fullName() }}</p>
  `
})
export class UserComponent {
  // ✅ Signals excel here
  firstName = signal('John');
  lastName = signal('Doe');
  fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
}
```

**3. Form State (Not HTTP)**:

```typescript
@Component({
  template: `
    <form>
      <input [value]="username()" (input)="username.set($any($event.target).value)">
      <input type="checkbox" [checked]="agreedToTerms()" 
             (change)="agreedToTerms.set($any($event.target).checked)">
      <button [disabled]="!isFormValid()">Submit</button>
    </form>
  `
})
export class SignupFormComponent {
  // ✅ Great for form state
  username = signal('');
  agreedToTerms = signal(false);
  
  isFormValid = computed(() => 
    this.username().length >= 3 && this.agreedToTerms()
  );
}
```

**4. UI State Flags**:

```typescript
@Component({
  template: `
    <button (click)="toggleSidebar()">Toggle</button>
    <aside [class.open]="sidebarOpen()">...</aside>
    
    <div *ngIf="darkMode()">Dark theme</div>
  `
})
export class LayoutComponent {
  // ✅ Perfect for UI toggles
  sidebarOpen = signal(false);
  darkMode = signal(false);
  
  toggleSidebar(): void {
    this.sidebarOpen.update(open => !open);
  }
}
```

**5. Counts, Counters, Tallies**:

```typescript
@Component({
  template: `
    <p>Cart: {{ itemCount() }} items</p>
    <p>Unread: {{ unreadCount() }}</p>
  `
})
export class HeaderComponent {
  // ✅ Signals are ideal
  itemCount = signal(0);
  unreadCount = signal(5);
}
```

### When to Use Observables

**✅ Use Observables For**:[3][5]

**1. HTTP Requests**:

```typescript
@Component({
  template: `
    <div *ngIf="user$ | async as user">
      {{ user.name }}
    </div>
  `
})
export class UserProfileComponent {
  // ✅ Observables are perfect for async data
  user$ = this.http.get<User>('/api/user/123');
  
  constructor(private http: HttpClient) {}
}
```

**2. Event Streams**:

```typescript
@Component({
  template: `<input (input)="onSearchInput($event)" />`
})
export class SearchComponent implements OnInit {
  private searchSubject = new Subject<string>();
  
  // ✅ Observables handle event streams well
  searchResults$ = this.searchSubject.pipe(
    debounceTime(300),
    distinctUntilChanged(),
    switchMap(query => this.api.search(query))
  );
  
  constructor(private api: SearchService) {}
  
  ngOnInit(): void {
    this.searchResults$.subscribe(results => {
      console.log('Search results:', results);
    });
  }
  
  onSearchInput(event: Event): void {
    const query = (event.target as HTMLInputElement).value;
    this.searchSubject.next(query);
  }
}
```

**3. WebSocket Connections**:

```typescript
@Injectable({ providedIn: 'root' })
export class WebSocketService {
  // ✅ Observables for continuous data streams
  messages$ = webSocket<Message>('ws://api.example.com/socket').pipe(
    retry({ delay: 3000 }),
    share()
  );
}
```

**4. Complex Async Workflows**:

```typescript
@Component({ /* ... */ })
export class DataSyncComponent implements OnInit {
  
  ngOnInit(): void {
    // ✅ Observables for complex async chains
    this.http.get<User>('/api/user').pipe(
      switchMap(user => this.http.get(`/api/user/${user.id}/settings`)),
      switchMap(settings => this.http.post('/api/sync', settings)),
      catchError(error => {
        console.error('Sync failed:', error);
        return of(null);
      }),
      finalize(() => console.log('Sync complete'))
    ).subscribe();
  }
  
  constructor(private http: HttpClient) {}
}
```

**5. Timer/Interval Operations**:

```typescript
@Component({
  template: `<p>Elapsed: {{ elapsed$ | async }} seconds</p>`
})
export class TimerComponent implements OnInit {
  // ✅ Observables for time-based operations
  elapsed$ = interval(1000).pipe(
    take(60),
    map(count => count + 1)
  );
  
  ngOnInit(): void {
    this.elapsed$.subscribe();
  }
}
```

**6. Reactive Form Value Changes**:

```typescript
@Component({ /* ... */ })
export class ReactiveFormComponent implements OnInit {
  form = new FormGroup({
    email: new FormControl(''),
    password: new FormControl('')
  });
  
  ngOnInit(): void {
    // ✅ Observables for form value streams
    this.form.get('email')?.valueChanges.pipe(
      debounceTime(300),
      switchMap(email => this.api.checkEmailAvailability(email))
    ).subscribe(available => {
      console.log('Email available:', available);
    });
  }
}
```

### Combining Signals and Observables

Angular provides interop utilities for seamless integration:[3]

#### Converting Observable to Signal: `toSignal()`

```typescript
import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  template: `
    <div *ngIf="user(); else loading">
      <h2>{{ user()?.name }}</h2>
      <p>{{ user()?.email }}</p>
    </div>
    
    <ng-template #loading>Loading...</ng-template>
  `
})
export class UserComponent {
  private http = inject(HttpClient);
  
  // Convert Observable to Signal
  user = toSignal(
    this.http.get<User>('/api/user/123'),
    { initialValue: null as User | null }
  );
  
  // Now use as signal
  userName = computed(() => this.user()?.name ?? 'Guest');
}
```

**Options for `toSignal()`**:

```typescript
// With initial value (synchronous)
const count = toSignal(interval(1000), { initialValue: 0 });

// Without initial value (might be undefined initially)
const data = toSignal(this.http.get('/api/data'));

// Handle errors
const safeData = toSignal(
  this.http.get('/api/data').pipe(
    catchError(() => of({ default: 'value' }))
  ),
  { initialValue: { default: 'initial' } }
);
```

#### Converting Signal to Observable: `toObservable()`

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, switchMap } from 'rxjs/operators';

@Component({
  template: `
    <input [value]="searchQuery()" 
           (input)="searchQuery.set($any($event.target).value)">
    
    <div *ngIf="searchResults$ | async as results">
      <p *ngFor="let result of results">{{ result.name }}</p>
    </div>
  `
})
export class SearchComponent {
  searchQuery = signal('');
  
  // Convert signal to observable for RxJS processing
  private searchQuery$ = toObservable(this.searchQuery);
  
  // Use RxJS operators
  searchResults$ = this.searchQuery$.pipe(
    debounceTime(300),
    switchMap(query => this.api.search(query))
  );
  
  constructor(private api: SearchService) {}
}
```

### Hybrid Pattern: Best of Both Worlds

```typescript
@Component({
  template: `
    <input [value]="searchQuery()" 
           (input)="updateSearch($any($event.target).value)">
    
    <div *ngIf="isSearching()">Searching...</div>
    
    <ul>
      @for (result of results(); track result.id) {
        <li>{{ result.name }}</li>
      }
    </ul>
    
    <p *ngIf="error()">Error: {{ error() }}</p>
  `
})
export class HybridSearchComponent {
  // Signals for UI state
  searchQuery = signal('');
  isSearching = signal(false);
  results = signal<SearchResult[]>([]);
  error = signal<string | null>(null);
  
  constructor(
    private http: HttpClient,
    private destroyRef = inject(DestroyRef)
  ) {
    // Convert signal to observable for async operations
    toObservable(this.searchQuery)
      .pipe(
        debounceTime(300),
        tap(() => {
          this.isSearching.set(true);
          this.error.set(null);
        }),
        switchMap(query => 
          query ? this.http.get<SearchResult[]>(`/api/search?q=${query}`) : of([])
        ),
        takeUntilDestroyed(this.destroyRef)
      )
      .subscribe({
        next: (results) => {
          this.results.set(results);
          this.isSearching.set(false);
        },
        error: (err) => {
          this.error.set(err.message);
          this.isSearching.set(false);
        }
      });
  }
  
  updateSearch(query: string): void {
    this.searchQuery.set(query);
  }
}
```

## Advanced Patterns and Best Practices

### 1. Signal-Based Services

Create reactive services using signals:[6][1]

```typescript
import { Injectable, signal, computed } from '@angular/core';

export interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

@Injectable({ providedIn: 'root' })
export class CartService {
  // Private writable signal
  private _items = signal<CartItem[]>([]);
  
  // Public read-only signal
  readonly items = this._items.asReadonly();
  
  // Computed signals
  readonly itemCount = computed(() => 
    this._items().reduce((sum, item) => sum + item.quantity, 0)
  );
  
  readonly subtotal = computed(() => 
    this._items().reduce((sum, item) => 
      sum + (item.price * item.quantity), 0
    )
  );
  
  readonly tax = computed(() => this.subtotal() * 0.1);
  
  readonly total = computed(() => this.subtotal() + this.tax());
  
  // Public methods for controlled updates
  addItem(item: Omit<CartItem, 'quantity'>): void {
    this._items.update(current => {
      const existingItem = current.find(i => i.id === item.id);
      
      if (existingItem) {
        return current.map(i => 
          i.id === item.id 
            ? { ...i, quantity: i.quantity + 1 }
            : i
        );
      } else {
        return [...current, { ...item, quantity: 1 }];
      }
    });
  }
  
  removeItem(id: number): void {
    this._items.update(current => current.filter(item => item.id !== id));
  }
  
  updateQuantity(id: number, quantity: number): void {
    if (quantity <= 0) {
      this.removeItem(id);
      return;
    }
    
    this._items.update(current => 
      current.map(item => 
        item.id === id ? { ...item, quantity } : item
      )
    );
  }
  
  clearCart(): void {
    this._items.set([]);
  }
}
```

**Using the Service**:

```typescript
@Component({
  selector: 'app-cart',
  template: `
    <h2>Shopping Cart</h2>
    
    <div *ngIf="cart.items().length === 0">
      <p>Your cart is empty</p>
    </div>
    
    <ul>
      @for (item of cart.items(); track item.id) {
        <li>
          {{ item.name }} - ${{ item.price }}
          <input 
            type="number" 
            [value]="item.quantity"
            (change)="updateQty(item.id, $any($event.target).value)">
          <button (click)="cart.removeItem(item.id)">Remove</button>
        </li>
      }
    </ul>
    
    <div class="summary">
      <p>Items: {{ cart.itemCount() }}</p>
      <p>Subtotal: ${{ cart.subtotal().toFixed(2) }}</p>
      <p>Tax: ${{ cart.tax().toFixed(2) }}</p>
      <p><strong>Total: ${{ cart.total().toFixed(2) }}</strong></p>
    </div>
  `
})
export class CartComponent {
  constructor(public cart: CartService) {}
  
  updateQty(id: number, value: string): void {
    const quantity = parseInt(value, 10);
    if (!isNaN(quantity)) {
      this.cart.updateQuantity(id, quantity);
    }
  }
}
```

### 2. Signal Inputs (Angular 17.1+)

```typescript
import { Component, input, output } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ fullName() }}</h3>
      <p>{{ email() }}</p>
      <p>Age: {{ age() ?? 'N/A' }}</p>
      <button (click)="handleClick()">Contact</button>
    </div>
  `
})
export class UserCardComponent {
  // Signal inputs (read-only)
  firstName = input.required<string>();
  lastName = input.required<string>();
  email = input.required<string>();
  age = input<number>();  // Optional
  
  // Computed from inputs
  fullName = computed(() => 
    `${this.firstName()} ${this.lastName()}`
  );
  
  // Outputs
  contactClicked = output<string>();
  
  handleClick(): void {
    this.contactClicked.emit(this.email());
  }
}
```

**Usage**:

```typescript
@Component({
  template: `
    <app-user-card 
      [firstName]="'John'"
      [lastName]="'Doe'"
      [email]="'john@example.com'"
      [age]="30"
      (contactClicked)="onContact($event)" />
  `
})
export class ParentComponent {
  onContact(email: string): void {
    console.log('Contact:', email);
  }
}
```

### 3. Handling Arrays and Objects Immutably

**❌ Wrong: Direct Mutation**:[1]

```typescript
// DON'T DO THIS
const items = signal([1, 2, 3]);
items().push(4);  // Mutates array directly - signal doesn't know!

const user = signal({ name: 'John', age: 30 });
user().age = 31;  // Mutates object directly - signal doesn't know!
```

**✅ Correct: Immutable Updates**:[1]

```typescript
// DO THIS
const items = signal([1, 2, 3]);
items.update(current => [...current, 4]);  // Creates new array

const user = signal({ name: 'John', age: 30 });
user.update(current => ({ ...current, age: 31 }));  // Creates new object
```

### 4. Custom Equality Functions

For expensive object comparisons:[1][2]

```typescript
import { signal } from '@angular/core';
import { isEqual } from 'lodash-es';

interface ComplexData {
  id: number;
  values: number[];
  nested: { key: string };
}

const data = signal<ComplexData>(
  { id: 1, values: [1, 2, 3], nested: { key: 'value' } },
  {
    // Custom deep equality check
    equal: (a, b) => isEqual(a, b)
  }
);

// This won't trigger updates (deep equal)
data.set({ id: 1, values: [1, 2, 3], nested: { key: 'value' } });

// This will trigger updates (different values)
data.set({ id: 1, values: [1, 2, 3, 4], nested: { key: 'value' } });
```

### 5. Untracked Reads

Read signal without creating dependency:[2]

```typescript
import { signal, computed, untracked } from '@angular/core';

const count = signal(0);
const multiplier = signal(10);

// Only depends on count, not multiplier
const result = computed(() => {
  const currentCount = count();
  const currentMultiplier = untracked(multiplier);  // Not tracked
  
  return currentCount * currentMultiplier;
});

// Changing multiplier won't trigger recomputation
multiplier.set(20);
console.log(result());  // Still uses old multiplier value

// Changing count WILL trigger recomputation
count.set(5);
console.log(result());  // Now uses new count, but still old multiplier
```

## Real-World Example: Task Management App

Complete application showcasing all concepts:

```typescript
// models/task.model.ts
export interface Task {
  id: number;
  title: string;
  description: string;
  completed: boolean;
  priority: 'low' | 'medium' | 'high';
  dueDate: Date;
}

export type TaskFilter = 'all' | 'active' | 'completed';
export type TaskSort = 'dueDate' | 'priority' | 'title';

// services/task.service.ts
import { Injectable, signal, computed } from '@angular/core';
import { Task, TaskFilter, TaskSort } from '../models/task.model';

@Injectable({ providedIn: 'root' })
export class TaskService {
  private _tasks = signal<Task[]>([]);
  private _filter = signal<TaskFilter>('all');
  private _sortBy = signal<TaskSort>('dueDate');
  private _searchQuery = signal('');
  
  // Public read-only signals
  readonly tasks = this._tasks.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly sortBy = this._sortBy.asReadonly();
  readonly searchQuery = this._searchQuery.asReadonly();
  
  // Computed: filtered tasks
  readonly filteredTasks = computed(() => {
    const tasks = this._tasks();
    const filter = this._filter();
    const query = this._searchQuery().toLowerCase();
    
    let filtered = tasks;
    
    // Apply filter
    if (filter === 'active') {
      filtered = filtered.filter(t => !t.completed);
    } else if (filter === 'completed') {
      filtered = filtered.filter(t => t.completed);
    }
    
    // Apply search
    if (query) {
      filtered = filtered.filter(t =>
        t.title.toLowerCase().includes(query) ||
        t.description.toLowerCase().includes(query)
      );
    }
    
    return filtered;
  });
  
  // Computed: sorted tasks
  readonly sortedTasks = computed(() => {
    const tasks = [...this.filteredTasks()];
    const sortBy = this._sortBy();
    
    return tasks.sort((a, b) => {
      if (sortBy === 'dueDate') {
        return a.dueDate.getTime() - b.dueDate.getTime();
      } else if (sortBy === 'priority') {
        const priorityOrder = { high: 3, medium: 2, low: 1 };
        return priorityOrder[b.priority] - priorityOrder[a.priority];
      } else {
        return a.title.localeCompare(b.title);
      }
    });
  });
  
  // Computed: statistics
  readonly totalCount = computed(() => this._tasks().length);
  readonly activeCount = computed(() => 
    this._tasks().filter(t => !t.completed).length
  );
  readonly completedCount = computed(() => 
    this._tasks().filter(t => t.completed).length
  );
  readonly highPriorityCount = computed(() =>
    this._tasks().filter(t => t.priority === 'high' && !t.completed).length
  );
  
  // Methods
  addTask(task: Omit<Task, 'id'>): void {
    const newTask: Task = {
      ...task,
      id: Date.now()
    };
    
    this._tasks.update(tasks => [...tasks, newTask]);
  }
  
  updateTask(id: number, updates: Partial<Task>): void {
    this._tasks.update(tasks =>
      tasks.map(task =>
        task.id === id ? { ...task, ...updates } : task
      )
    );
  }
  
  deleteTask(id: number): void {
    this._tasks.update(tasks => tasks.filter(t => t.id !== id));
  }
  
  toggleComplete(id: number): void {
    this._tasks.update(tasks =>
      tasks.map(task =>
        task.id === id ? { ...task, completed: !task.completed } : task
      )
    );
  }
  
  setFilter(filter: TaskFilter): void {
    this._filter.set(filter);
  }
  
  setSortBy(sortBy: TaskSort): void {
    this._sortBy.set(sortBy);
  }
  
  setSearchQuery(query: string): void {
    this._searchQuery.set(query);
  }
  
  constructor() {
    // Load from localStorage
    this.loadFromStorage();
    
    // Save to localStorage on changes
    effect(() => {
      const tasks = this._tasks();
      localStorage.setItem('tasks', JSON.stringify(tasks));
    });
  }
  
  private loadFromStorage(): void {
    const stored = localStorage.getItem('tasks');
    if (stored) {
      try {
        const tasks = JSON.parse(stored);
        // Convert date strings back to Date objects
        const parsedTasks = tasks.map((t: any) => ({
          ...t,
          dueDate: new Date(t.dueDate)
        }));
        this._tasks.set(parsedTasks);
      } catch (e) {
        console.error('Failed to load tasks from storage', e);
      }
    }
  }
}

// components/task-list.component.ts
@Component({
  selector: 'app-task-list',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="task-app">
      <header>
        <h1>Task Manager</h1>
        
        <div class="stats">
          <span>Total: {{ taskService.totalCount() }}</span>
          <span>Active: {{ taskService.activeCount() }}</span>
          <span>Completed: {{ taskService.completedCount() }}</span>
          <span class="high-priority" *ngIf="taskService.highPriorityCount() > 0">
            High Priority: {{ taskService.highPriorityCount() }}
          </span>
        </div>
      </header>
      
      <div class="controls">
        <input 
          type="text"
          placeholder="Search tasks..."
          [value]="taskService.searchQuery()"
          (input)="onSearch($any($event.target).value)">
        
        <div class="filters">
          <button 
            *ngFor="let f of filters"
            [class.active]="taskService.filter() === f"
            (click)="taskService.setFilter(f)">
            {{ f | titlecase }}
          </button>
        </div>
        
        <select 
          [value]="taskService.sortBy()"
          (change)="taskService.setSortBy($any($event.target).value)">
          <option value="dueDate">Due Date</option>
          <option value="priority">Priority</option>
          <option value="title">Title</option>
        </select>
      </div>
      
      <div class="task-list">
        @for (task of taskService.sortedTasks(); track task.id) {
          <div class="task-card" [class.completed]="task.completed">
            <input 
              type="checkbox"
              [checked]="task.completed"
              (change)="taskService.toggleComplete(task.id)">
            
            <div class="task-content">
              <h3>{{ task.title }}</h3>
              <p>{{ task.description }}</p>
              <div class="task-meta">
                <span class="priority" [class]="'priority-' + task.priority">
                  {{ task.priority }}
                </span>
                <span class="due-date">
                  Due: {{ task.dueDate | date:'short' }}
                </span>
              </div>
            </div>
            
            <button (click)="taskService.deleteTask(task.id)">Delete</button>
          </div>
        } @empty {
          <p class="empty-state">No tasks found</p>
        }
      </div>
      
      <button class="add-task-btn" (click)="showAddTask = true">
        Add New Task
      </button>
    </div>
  `,
  styles: [`
    .task-app {
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }
    
    .stats {
      display: flex;
      gap: 20px;
      margin: 10px 0;
    }
    
    .high-priority {
      color: #dc3545;
      font-weight: bold;
    }
    
    .controls {
      display: flex;
      gap: 10px;
      margin: 20px 0;
      flex-wrap: wrap;
    }
    
    .filters button.active {
      background-color: #007bff;
      color: white;
    }
    
    .task-card {
      display: flex;
      gap: 15px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 8px;
      margin-bottom: 10px;
    }
    
    .task-card.completed {
      opacity: 0.6;
    }
    
    .task-card.completed .task-content h3 {
      text-decoration: line-through;
    }
    
    .priority-high { 
      background-color: #dc3545; 
      color: white;
      padding: 2px 8px;
      border-radius: 4px;
    }
    
    .priority-medium { 
      background-color: #ffc107; 
      padding: 2px 8px;
      border-radius: 4px;
    }
    
    .priority-low { 
      background-color: #28a745; 
      color: white;
      padding: 2px 8px;
      border-radius: 4px;
    }
  `]
})
export class TaskListComponent {
  filters: TaskFilter[] = ['all', 'active', 'completed'];
  showAddTask = false;
  
  constructor(public taskService: TaskService) {}
  
  onSearch(query: string): void {
    this.taskService.setSearchQuery(query);
  }
}
```

## Best Practices Summary

### ✅ DO

1. **Use signals for synchronous UI state**[6][3]
2. **Use computed for derived values**[1][2]
3. **Keep writable signals private in services**[6][1]
4. **Use effects for side effects only**[4][2]
5. **Update arrays/objects immutably**[1]
6. **Use `toSignal()` to convert async data to signals**[3]
7. **Combine signals with observables when needed**[3]
8. **Use signal inputs for component communication**

### ❌ DON'T

1. **Don't use effects for state derivation** (use computed)[2]
2. **Don't mutate signal values directly**[1]
3. **Don't conditionally read signals in computed** (breaks dependencies)[1]
4. **Don't expose writable signals publicly**[6][1]
5. **Don't use signals for async operations** (use observables)[3]
6. **Don't create circular signal dependencies**
7. **Don't overuse custom equality functions**[1]

## Migration Strategy

### Gradual Adoption

You don't need to rewrite everything at once:

**Phase 1: New Components**
- Use signals for all new component state
- Keep existing components as-is

**Phase 2: Services**
- Migrate services to signal-based APIs
- Use `toSignal()` for existing observable APIs

**Phase 3: Refactor Components**
- Gradually convert component state to signals
- Replace `BehaviorSubject` with signals in services

### Quick Wins

Replace these patterns first:

**Before (BehaviorSubject)**:
```typescript
private countSubject = new BehaviorSubject(0);
count$ = this.countSubject.asObservable();

increment(): void {
  this.countSubject.next(this.countSubject.value + 1);
}
```

**After (Signal)**:
```typescript
count = signal(0);

increment(): void {
  this.count.update(c => c + 1);
}
```

## Conclusion

Angular Signals represent the future of reactivity in Angular. They provide:[3][1]

- **Simpler mental model** than RxJS for synchronous state
- **Better performance** through fine-grained reactivity
- **Improved developer experience** with less boilerplate
- **Seamless integration** with existing Angular features

**Golden Rule**: Use **Signals** for synchronous state, **Observables** for asynchronous operations, and combine them when needed.[5][3]

Signals are not replacing Observables—they complement each other, giving you the best tool for each job.[3]

## Additional Resources

- **Official Documentation**: https://angular.dev/guide/signals
- **Angular University Guide**: https://blog.angular-university.io/angular-signals/
- **Google Codelabs**: https://codelabs.developers.google.com/angular-signals
- **GitHub Examples**: https://github.com/sonusindhu/angular-signals-examples
- **Angular Blog**: https://blog.angular.io

Start using Signals today and experience the future of Angular development!
