# How to Build Multi-Language Angular Apps Using ngx-translate

Are you developing a product for a global audience that requires support for multiple languages? If your project uses Angular, this article will guide you to complete this task more easily than ever before.

Building applications that cater to international users is crucial for expanding your reach worldwide. While Angular has built-in **internationalization (i18n)** capabilities, **ngx-translate** is a widely-used third-party library that provides a more flexible and developer-friendly way to handle translations.

---

## What is ngx-translate?

**ngx-translate** is one of the most popular libraries for adding internationalization (i18n) to Angular applications. It’s widely used because it’s flexible, easy to set up, and allows you to switch languages **dynamically at runtime** — without reloading the app. 



### Key features of ngx-translate:
* **Dynamic language switching** at runtime (no page reload).
* **Lightweight** and easy to integrate.
* Supports loading translations from **JSON files**.
* Features plugins for **HTTP loading** and message formatting.

---

## Setup and Configuration of ngx-translate

### Step 1: Install libraries
Open your terminal and run:
```bash
npm install @ngx-translate/core @ngx-translate/http-loader
```

### Step 2: Configure in AppModule
Set up a factory function to load translation JSON files dynamically from `./assets/i18n/`.

```typescript
import { HttpClientModule, HttpClient } from '@angular/common/http';
import { TranslateLoader, TranslateModule } from '@ngx-translate/core';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';

export function HttpLoaderFactory(http: HttpClient) {
  return new TranslateHttpLoader(http, './assets/i18n/', '.json');
}

@NgModule({
  imports: [
    BrowserModule,
    HttpClientModule,
    TranslateModule.forRoot({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient],
      },
    }),
  ],
})
export class AppModule {}
```

### Step 3: Set Preferred Language on Load
In your `AppComponent`, check for a saved preference or detect the browser language.

```typescript
export class AppComponent implements OnInit {
  constructor(private translate: TranslateService) {}

  ngOnInit() {
    const selectedLang = sessionStorage.getItem('selectedLang') || 'English';
    this.translate.setDefaultLang('English');
    this.translate.use(selectedLang);
    sessionStorage.setItem('selectedLang', selectedLang);
  }
}
```

### Step 4: Add Translation JSON Files
Create a folder at `assets/i18n` and add JSON files for each language.

**assets/i18n/English.json**
```json
{
  "title": "Welcome to My App",
  "description": "This is a multi-language Angular application.",
  "something_went_wrong":"Oops! Something went wrong, please try again later"
}
```

**assets/i18n/Hindi.json**
```json
{
  "title": "मेरे ऐप में आपका स्वागत है",
  "description": "यह एक बहुभाषी Angular एप्लिकेशन है।",
  "something_went_wrong": "ओह! कुछ गलत हो गया, कृपया बाद में फिर से प्रयास करें।"
}
```

---

## Implementing Translations in Components

### Step 1: Create a Language Switcher Component
This handles language selection and persists the user’s choice.

```typescript
@Component({
  selector: 'app-language-switcher',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './language-switcher.component.html'
})
export class LanguageSwitcherComponent implements OnInit {
  selectedLanguage = '';
  languageList: any[] = [];

  constructor(private translate: TranslateService) {}

  changeLang(langName: string): void {
    if (langName) {
      this.translate.use(langName);
      sessionStorage.setItem('selectedLang', langName);
    }
  }
}
```

### Step 2: Component Template
Render a dropdown for users to select their language.

```html
<select class="form-select" [(ngModel)]="selectedLanguage" (change)="changeLang(selectedLanguage)">
  <option *ngFor="let l of languageList" [value]="l.name">{{ l.name }}</option>
</select>
```

### Step 3: Use the Translate Pipe
Use the `translate` pipe to replace keys with text based on the active language.

```html
<div class="container">
  <h2>{{ 'title' | translate }}</h2>
  <p>{{ 'description' | translate }}</p>
</div>
```

---

## Advanced Features

### 1. Using Parameters
Pass dynamic values into your translation strings using parameters as placeholders.

**JSON:**
```json
{ "WELCOME_MESSAGE": "Welcome, {{username}}!" }
```

**Template:**
```html
<p>{{ 'WELCOME_MESSAGE' | translate:{ username: 'Sunil' } }}</p>
```

### 2. Lazy Loading
For feature modules, use `.forChild()` to load translations only when the module is accessed.



```typescript
@NgModule({
  imports: [
    TranslateModule.forChild({
      loader: {
        provide: TranslateLoader,
        useFactory: HttpLoaderFactory,
        deps: [HttpClient],
      },
    }),
  ],
})
export class FeatureModule {}
```

ngx-translate offers an intuitive structure that feels familiar to developers coming from Android or Windows ecosystems. It’s a robust choice for any modern multilingual Angular app.
