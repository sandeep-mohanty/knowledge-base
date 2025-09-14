# ✅ Using NgRx Partially in an Angular Application

This tutorial will guide you through integrating **NgRx selectively** into an Angular app — applying it only to some features (e.g., `auth`, `products`), while using local state or services for others (e.g., `settings`, `dashboard`).

---

## 🧱 Project Setup

Start with a basic Angular CLI project, or use an existing one.

**Install NgRx packages:**

// Put this in backticks  
ng add @ngrx/store  
ng add @ngrx/effects  
ng add @ngrx/store-devtools  
ng add @ngrx/entity

---

## 🗂️ Folder Structure (Example)

We'll use NgRx only for `auth` and `products`.

```
src/  
├── app/  
│   ├── core/               // Shared services  
│   ├── state/              // Only NgRx-enabled features  
│   │   ├── auth/  
│   │   ├── products/  
│   ├── features/  
│   │   ├── dashboard/      // Uses local/component state  
│   │   ├── settings/       // Uses service-based state
```

---

## 🧩 Step-by-Step: Using NgRx for `products`

We'll wire up NgRx only for the **Products** feature.

---

### 1. Create the `products` state folder

src/app/state/products/

Inside it, create:

- products.actions.ts  
- products.reducer.ts  
- products.effects.ts  
- products.selectors.ts  
- products.models.ts

---

### 2. Define Model

**products.models.ts**

```ts
export interface Product {  
  id: number;  
  name: string;  
  price: number;  
}
```
---

### 3. Define Actions

**products.actions.ts**

```ts  
import { createAction, props } from '@ngrx/store';  
import { Product } from './products.models';

export const loadProducts = createAction('[Products] Load Products');  
export const loadProductsSuccess = createAction('[Products] Load Products Success', props<{ products: Product[] }>());  
export const loadProductsFailure = createAction('[Products] Load Products Failure', props<{ error: any }>());
```

---

### 4. Create Reducer

**products.reducer.ts**

```ts  
import { createReducer, on } from '@ngrx/store';  
import * as ProductActions from './products.actions';  
import { Product } from './products.models';

export interface ProductsState {  
  products: Product[];  
  loading: boolean;  
  error: any;  
}

export const initialState: ProductsState = {  
  products: [],  
  loading: false,  
  error: null  
};

export const productsReducer = createReducer(  
  initialState,  
  on(ProductActions.loadProducts, state => ({ ...state, loading: true })),  
  on(ProductActions.loadProductsSuccess, (state, { products }) => ({  
    ...state,  
    loading: false,  
    products  
  })),  
  on(ProductActions.loadProductsFailure, (state, { error }) => ({  
    ...state,  
    loading: false,  
    error  
  }))  
);
```
---

### 5. Define Selectors

**products.selectors.ts**

```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';  
import { ProductsState } from './products.reducer';

export const selectProductsState = createFeatureSelector<ProductsState>('products');

export const selectAllProducts = createSelector(  
  selectProductsState,  
  (state) => state.products  
);

export const selectLoading = createSelector(  
  selectProductsState,  
  (state) => state.loading  
);

```
---

### 6. Create Effects

**products.effects.ts**

```ts
import { Injectable } from '@angular/core';  
import { Actions, createEffect, ofType } from '@ngrx/effects';  
import { ProductService } from '../../core/product.service';  
import * as ProductActions from './products.actions';  
import { catchError, map, mergeMap, of } from 'rxjs';

@Injectable()  
export class ProductsEffects {  
  loadProducts$ = createEffect(() =>  
    this.actions$.pipe(  
      ofType(ProductActions.loadProducts),  
      mergeMap(() =>  
        this.productService.getProducts().pipe(  
          map(products => ProductActions.loadProductsSuccess({ products })),  
          catchError(error => of(ProductActions.loadProductsFailure({ error })))  
        )  
      )  
    )  
  );

  constructor(  
    private actions$: Actions,  
    private productService: ProductService  
  ) {}  
}
```
---

## ⚙️ Register the Feature in App Module

**app.module.ts**

```ts
import { StoreModule } from '@ngrx/store';  
import { EffectsModule } from '@ngrx/effects';  
import { productsReducer } from './state/products/products.reducer';  
import { ProductsEffects } from './state/products/products.effects';

@NgModule({  
  imports: [  
    StoreModule.forRoot({}),  
    EffectsModule.forRoot([]),  
    StoreModule.forFeature('products', productsReducer),  
    EffectsModule.forFeature([ProductsEffects])  
  ]  
})  
export class AppModule {}
```
---

## 🧪 Use in Product List Component

**product-list.component.ts**

```ts
import { Component, OnInit } from '@angular/core';  
import { Store } from '@ngrx/store';  
import { loadProducts } from '../../state/products/products.actions';  
import { selectAllProducts, selectLoading } from '../../state/products/products.selectors';

@Component({  
  selector: 'app-product-list',  
  templateUrl: './product-list.component.html'  
})  
export class ProductListComponent implements OnInit {  
  products$ = this.store.select(selectAllProducts);  
  loading$ = this.store.select(selectLoading);

  constructor(private store: Store) {}

  ngOnInit() {  
    this.store.dispatch(loadProducts());  
  }  
}
```

---

## 🧼 For Other Features (Like Settings or Dashboard)

No NgRx is needed. Just use services or component state.

**settings.service.ts**

```ts
@Injectable({ providedIn: 'root' })  
export class SettingsService {  
  private theme = new BehaviorSubject<'light' | 'dark'>('light');  
  theme$ = this.theme.asObservable();

  setTheme(theme: 'light' | 'dark') {  
    this.theme.next(theme);  
  }  
}
```

**dashboard.component.ts**

```ts  
export class DashboardComponent {  
  count = 0;  
  increment() { this.count++; }  
}
```

---

## 📌 Summary

- ✅ Use NgRx only where it adds value (e.g., shared state, side effects, complex flows).  
- 🧠 Use services or component state for simple or isolated features.  
- 🧩 NgRx modules can be registered per feature using StoreModule.forFeature.  
- 🧼 This hybrid approach keeps your app clean, performant, and scalable.

---

## 📁 Optional: Facade Pattern (For Abstraction)

To hide NgRx from components, you can create a **ProductsFacade**:

**products.facade.ts**

```ts
@Injectable({ providedIn: 'root' })  
export class ProductsFacade {  
  products$ = this.store.select(selectAllProducts);  
  loading$ = this.store.select(selectLoading);

  constructor(private store: Store) {}

  loadProducts() {  
    this.store.dispatch(loadProducts());  
  }  
}
```

Then in your component:

```ts 
constructor(private productsFacade: ProductsFacade) {}  
ngOnInit() {  
  this.productsFacade.loadProducts();  
}
```