# Creating a Slim Select Angular Component: Comprehensive Tutorial

## Introduction

This tutorial guides you through creating a reusable Angular component wrapper for Slim Select, a lightweight JavaScript select dropdown library. You'll learn how to properly integrate third-party JavaScript libraries into Angular using lifecycle hooks, ControlValueAccessor for reactive forms integration, and proper cleanup patterns.[1][2][3][4]

### Why Wrap Slim Select in an Angular Component?

**Benefits**:[4]

- **Reusability**: Use the component throughout your application with consistent behavior
- **Type Safety**: Leverage TypeScript for configuration and data
- **Angular Forms Integration**: Work seamlessly with Reactive Forms and Template-driven Forms
- **Change Detection**: Proper integration with Angular's change detection
- **Lifecycle Management**: Automatic initialization and cleanup
- **Encapsulation**: Hide implementation details from consuming components

### Prerequisites

- Angular 15+ (examples use standalone components, but can be adapted for module-based)
- TypeScript knowledge
- Understanding of Angular lifecycle hooks
- Familiarity with Angular Forms (Reactive/Template-driven)

## Project Setup

### Step 1: Install Slim Select

```bash
npm install slim-select --save
```

### Step 2: Verify Installation

Check your `package.json`:

```json
{
  "dependencies": {
    "slim-select": "^2.8.0"
  }
}
```

### Step 3: Configure TypeScript (if needed)

If you encounter type issues, ensure your `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "esModuleInterop": true
  }
}
```

## Creating the Basic Component

### Step 1: Generate the Component

```bash
ng generate component components/slim-select --standalone
```

Or create manually:

**File: `slim-select.component.ts`**

```typescript
import { 
  Component, 
  ElementRef, 
  ViewChild, 
  AfterViewInit,
  OnDestroy,
  Input,
  Output,
  EventEmitter,
  forwardRef
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';
import SlimSelect from 'slim-select';

@Component({
  selector: 'app-slim-select',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './slim-select.component.html',
  styleUrls: ['./slim-select.component.css']
})
export class SlimSelectComponent implements AfterViewInit, OnDestroy {
  @ViewChild('selectElement', { static: false }) selectElement!: ElementRef<HTMLSelectElement>;
  
  private slimSelectInstance?: SlimSelect;

  ngAfterViewInit(): void {
    // Initialize Slim Select after view is ready
    this.initSlimSelect();
  }

  ngOnDestroy(): void {
    // Clean up Slim Select instance
    this.destroySlimSelect();
  }

  private initSlimSelect(): void {
    if (this.selectElement) {
      this.slimSelectInstance = new SlimSelect({
        select: this.selectElement.nativeElement
      });
    }
  }

  private destroySlimSelect(): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.destroy();
      this.slimSelectInstance = undefined;
    }
  }
}
```

**File: `slim-select.component.html`**

```html
<select #selectElement>
  <ng-content></ng-content>
</select>
```

**File: `slim-select.component.css`**

```css
@import 'slim-select/styles';

/* Additional custom styles can go here */
```

### Understanding the Lifecycle Hooks

**ngAfterViewInit**:[3][5]

This hook is called after Angular initializes the component's view and child views. It's crucial for third-party library integration because:

- The DOM is fully rendered and accessible
- `@ViewChild` references are available
- Safe to manipulate native elements

**Why not ngOnInit?**: In `ngOnInit`, the view hasn't been initialized yet, so `@ViewChild` references would be undefined.[3]

**ngOnDestroy**:[3]

Called just before Angular destroys the component. Essential for:

- Cleaning up Slim Select instance
- Preventing memory leaks
- Removing event listeners
- Unsubscribing from observables

## Adding Configuration Options

### Step 2: Configure Inputs for Slim Select Options

```typescript
import { 
  Component, 
  ElementRef, 
  ViewChild, 
  AfterViewInit,
  OnDestroy,
  Input,
  Output,
  EventEmitter,
  OnChanges,
  SimpleChanges
} from '@angular/core';
import { CommonModule } from '@angular/common';
import SlimSelect, { DataArray, Option } from 'slim-select';

export interface SlimSelectConfig {
  placeholder?: string;
  showSearch?: boolean;
  searchText?: string;
  searchPlaceholder?: string;
  searchHighlight?: boolean;
  closeOnSelect?: boolean;
  allowDeselect?: boolean;
  hideSelected?: boolean;
  showOptionTooltips?: boolean;
}

@Component({
  selector: 'app-slim-select',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './slim-select.component.html',
  styleUrls: ['./slim-select.component.css']
})
export class SlimSelectComponent implements AfterViewInit, OnDestroy, OnChanges {
  @ViewChild('selectElement', { static: false }) selectElement!: ElementRef<HTMLSelectElement>;
  
  @Input() config: SlimSelectConfig = {};
  @Input() data?: DataArray;
  @Input() multiple: boolean = false;
  @Input() disabled: boolean = false;
  
  @Output() change = new EventEmitter<Option | Option[]>();
  @Output() open = new EventEmitter<void>();
  @Output() close = new EventEmitter<void>();
  @Output() search = new EventEmitter<string>();
  
  private slimSelectInstance?: SlimSelect;
  private isInitialized = false;

  ngAfterViewInit(): void {
    this.initSlimSelect();
    this.isInitialized = true;
  }

  ngOnChanges(changes: SimpleChanges): void {
    // Handle changes after initialization
    if (this.isInitialized) {
      if (changes['data'] && this.slimSelectInstance) {
        this.slimSelectInstance.setData(this.data || []);
      }
      
      if (changes['disabled'] && this.slimSelectInstance) {
        if (this.disabled) {
          this.slimSelectInstance.disable();
        } else {
          this.slimSelectInstance.enable();
        }
      }
    }
  }

  ngOnDestroy(): void {
    this.destroySlimSelect();
  }

  private initSlimSelect(): void {
    if (!this.selectElement) {
      return;
    }

    this.slimSelectInstance = new SlimSelect({
      select: this.selectElement.nativeElement,
      settings: {
        placeholderText: this.config.placeholder || 'Select Value',
        searchText: this.config.searchText || 'No results',
        searchPlaceholder: this.config.searchPlaceholder || 'Search',
        searchHighlight: this.config.searchHighlight ?? true,
        closeOnSelect: this.config.closeOnSelect ?? true,
        showSearch: this.config.showSearch ?? true,
        allowDeselect: this.config.allowDeselect ?? false,
        hideSelected: this.config.hideSelected ?? false,
        showOptionTooltips: this.config.showOptionTooltips ?? false
      },
      data: this.data,
      events: {
        afterChange: (newVal) => {
          this.change.emit(newVal);
        },
        afterOpen: () => {
          this.open.emit();
        },
        afterClose: () => {
          this.close.emit();
        },
        search: (search, currentData) => {
          this.search.emit(search);
          return currentData;
        }
      }
    });

    if (this.disabled) {
      this.slimSelectInstance.disable();
    }
  }

  private destroySlimSelect(): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.destroy();
      this.slimSelectInstance = undefined;
    }
  }

  // Public API methods
  public setData(data: DataArray): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.setData(data);
    }
  }

  public getData(): DataArray {
    return this.slimSelectInstance?.getData() || [];
  }

  public setSelected(value: string | string[]): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.setSelected(value);
    }
  }

  public getSelected(): string | string[] {
    return this.slimSelectInstance?.getSelected() || '';
  }

  public open(): void {
    this.slimSelectInstance?.open();
  }

  public close(): void {
    this.slimSelectInstance?.close();
  }

  public enable(): void {
    this.slimSelectInstance?.enable();
  }

  public disable(): void {
    this.slimSelectInstance?.disable();
  }
}
```

**Update Template**:

```html
<select 
  #selectElement 
  [multiple]="multiple"
  [disabled]="disabled">
  <ng-content></ng-content>
</select>
```

## Integrating with Angular Forms (ControlValueAccessor)

To make the component work seamlessly with Angular Reactive Forms and Template-driven Forms, implement `ControlValueAccessor`:[4]

### Step 3: Complete Form Integration

```typescript
import { 
  Component, 
  ElementRef, 
  ViewChild, 
  AfterViewInit,
  OnDestroy,
  Input,
  Output,
  EventEmitter,
  OnChanges,
  SimpleChanges,
  forwardRef
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';
import SlimSelect, { DataArray, Option } from 'slim-select';

export interface SlimSelectConfig {
  placeholder?: string;
  showSearch?: boolean;
  searchText?: string;
  searchPlaceholder?: string;
  searchHighlight?: boolean;
  closeOnSelect?: boolean;
  allowDeselect?: boolean;
  hideSelected?: boolean;
  showOptionTooltips?: boolean;
}

@Component({
  selector: 'app-slim-select',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './slim-select.component.html',
  styleUrls: ['./slim-select.component.css'],
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => SlimSelectComponent),
      multi: true
    }
  ]
})
export class SlimSelectComponent 
  implements AfterViewInit, OnDestroy, OnChanges, ControlValueAccessor {
  
  @ViewChild('selectElement', { static: false }) 
  selectElement!: ElementRef<HTMLSelectElement>;
  
  @Input() config: SlimSelectConfig = {};
  @Input() data?: DataArray;
  @Input() multiple: boolean = false;
  @Input() disabled: boolean = false;
  
  @Output() valueChange = new EventEmitter<string | string[]>();
  @Output() selectOpen = new EventEmitter<void>();
  @Output() selectClose = new EventEmitter<void>();
  @Output() searchChange = new EventEmitter<string>();
  
  private slimSelectInstance?: SlimSelect;
  private isInitialized = false;
  private _value: string | string[] = '';
  
  // ControlValueAccessor callbacks
  private onChange: (value: string | string[]) => void = () => {};
  private onTouched: () => void = () => {};

  ngAfterViewInit(): void {
    this.initSlimSelect();
    this.isInitialized = true;
  }

  ngOnChanges(changes: SimpleChanges): void {
    if (this.isInitialized) {
      if (changes['data'] && this.slimSelectInstance) {
        this.slimSelectInstance.setData(this.data || []);
      }
      
      if (changes['disabled'] && this.slimSelectInstance) {
        this.setDisabledState(this.disabled);
      }
    }
  }

  ngOnDestroy(): void {
    this.destroySlimSelect();
  }

  // ControlValueAccessor implementation
  writeValue(value: string | string[]): void {
    this._value = value || (this.multiple ? [] : '');
    
    if (this.slimSelectInstance) {
      this.slimSelectInstance.setSelected(this._value);
    }
  }

  registerOnChange(fn: (value: string | string[]) => void): void {
    this.onChange = fn;
  }

  registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }

  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
    if (this.slimSelectInstance) {
      if (isDisabled) {
        this.slimSelectInstance.disable();
      } else {
        this.slimSelectInstance.enable();
      }
    }
  }

  private initSlimSelect(): void {
    if (!this.selectElement) {
      return;
    }

    this.slimSelectInstance = new SlimSelect({
      select: this.selectElement.nativeElement,
      settings: {
        placeholderText: this.config.placeholder || 'Select Value',
        searchText: this.config.searchText || 'No results',
        searchPlaceholder: this.config.searchPlaceholder || 'Search',
        searchHighlight: this.config.searchHighlight ?? true,
        closeOnSelect: this.config.closeOnSelect ?? true,
        showSearch: this.config.showSearch ?? true,
        allowDeselect: this.config.allowDeselect ?? false,
        hideSelected: this.config.hideSelected ?? false,
        showOptionTooltips: this.config.showOptionTooltips ?? false
      },
      data: this.data,
      events: {
        afterChange: (newVal: Option | Option[]) => {
          // Extract value(s) from Option object(s)
          let value: string | string[];
          
          if (Array.isArray(newVal)) {
            value = newVal.map(opt => opt.value);
          } else {
            value = newVal.value;
          }
          
          this._value = value;
          
          // Notify Angular Forms
          this.onChange(value);
          this.onTouched();
          
          // Emit custom event
          this.valueChange.emit(value);
        },
        afterOpen: () => {
          this.onTouched();
          this.selectOpen.emit();
        },
        afterClose: () => {
          this.selectClose.emit();
        },
        search: (search, currentData) => {
          this.searchChange.emit(search);
          return currentData;
        }
      }
    });

    // Set initial value if provided
    if (this._value) {
      this.slimSelectInstance.setSelected(this._value);
    }

    if (this.disabled) {
      this.slimSelectInstance.disable();
    }
  }

  private destroySlimSelect(): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.destroy();
      this.slimSelectInstance = undefined;
    }
  }

  // Public API methods
  public setData(data: DataArray): void {
    if (this.slimSelectInstance) {
      this.slimSelectInstance.setData(data);
    }
  }

  public getData(): DataArray {
    return this.slimSelectInstance?.getData() || [];
  }

  public getSelected(): string | string[] {
    return this._value;
  }

  public open(): void {
    this.slimSelectInstance?.open();
  }

  public close(): void {
    this.slimSelectInstance?.close();
  }
}
```

### Understanding ControlValueAccessor

**What is ControlValueAccessor?**

It's an interface that bridges the gap between Angular Forms and custom form controls. It allows your component to:

- Work with `[(ngModel)]` (Template-driven Forms)
- Work with `formControlName` (Reactive Forms)
- Participate in form validation
- Respond to programmatic value changes

**Key Methods**:

1. **writeValue(value)**: Called when the form model changes (external → component)
2. **registerOnChange(fn)**: Receives callback to notify form model of changes (component → external)
3. **registerOnTouched(fn)**: Receives callback to mark control as touched
4. **setDisabledState(isDisabled)**: Called when form control's disabled state changes

## Usage Examples

### Example 1: Basic Usage with Static Options

```typescript
import { Component } from '@angular/core';
import { SlimSelectComponent } from './components/slim-select/slim-select.component';

@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [SlimSelectComponent],
  template: `
    <div class="demo-container">
      <h2>Basic Select</h2>
      
      <app-slim-select 
        [config]="{ placeholder: 'Select a framework' }"
        (valueChange)="onSelectionChange($event)">
        <option value="">Please select</option>
        <option value="angular">Angular</option>
        <option value="react">React</option>
        <option value="vue">Vue</option>
        <option value="svelte">Svelte</option>
      </app-slim-select>
      
      <p>Selected: {{ selectedValue }}</p>
    </div>
  `,
  styles: [`
    .demo-container {
      padding: 20px;
      max-width: 400px;
    }
  `]
})
export class DemoComponent {
  selectedValue: string = '';

  onSelectionChange(value: string | string[]): void {
    this.selectedValue = value as string;
    console.log('Selection changed:', value);
  }
}
```

### Example 2: With Option Groups

```typescript
@Component({
  selector: 'app-demo-groups',
  standalone: true,
  imports: [SlimSelectComponent],
  template: `
    <div class="demo-container">
      <h2>Select with Groups</h2>
      
      <app-slim-select 
        [config]="selectConfig"
        (valueChange)="onSelectionChange($event)">
        <optgroup label="Frontend Frameworks">
          <option value="angular">Angular</option>
          <option value="react">React</option>
          <option value="vue">Vue</option>
        </optgroup>
        <optgroup label="CSS Frameworks">
          <option value="bootstrap">Bootstrap</option>
          <option value="tailwind">Tailwind CSS</option>
          <option value="material">Material UI</option>
        </optgroup>
      </app-slim-select>
      
      <p>Selected: {{ selectedValue }}</p>
    </div>
  `
})
export class DemoGroupsComponent {
  selectConfig = {
    placeholder: 'Choose a framework',
    showSearch: true,
    searchPlaceholder: 'Search frameworks...'
  };

  selectedValue: string = '';

  onSelectionChange(value: string | string[]): void {
    this.selectedValue = value as string;
  }
}
```

### Example 3: Multiple Select

```typescript
@Component({
  selector: 'app-demo-multiple',
  standalone: true,
  imports: [SlimSelectComponent, CommonModule],
  template: `
    <div class="demo-container">
      <h2>Multiple Select</h2>
      
      <app-slim-select 
        [multiple]="true"
        [config]="multiSelectConfig"
        (valueChange)="onSelectionChange($event)">
        <option value="javascript">JavaScript</option>
        <option value="typescript">TypeScript</option>
        <option value="python">Python</option>
        <option value="java">Java</option>
        <option value="csharp">C#</option>
        <option value="go">Go</option>
      </app-slim-select>
      
      <div *ngIf="selectedValues.length > 0">
        <h3>Selected Languages:</h3>
        <ul>
          <li *ngFor="let lang of selectedValues">{{ lang }}</li>
        </ul>
      </div>
    </div>
  `
})
export class DemoMultipleComponent {
  multiSelectConfig = {
    placeholder: 'Select programming languages',
    showSearch: true,
    closeOnSelect: false,
    hideSelected: true
  };

  selectedValues: string[] = [];

  onSelectionChange(values: string | string[]): void {
    this.selectedValues = values as string[];
    console.log('Selected languages:', this.selectedValues);
  }
}
```

### Example 4: Dynamic Data with Data Array

```typescript
import { Component, OnInit } from '@angular/core';
import { SlimSelectComponent } from './components/slim-select/slim-select.component';
import { DataArray } from 'slim-select';

interface Country {
  id: number;
  name: string;
  code: string;
}

@Component({
  selector: 'app-demo-dynamic',
  standalone: true,
  imports: [SlimSelectComponent],
  template: `
    <div class="demo-container">
      <h2>Dynamic Data</h2>
      
      <app-slim-select 
        [data]="countryData"
        [config]="config"
        (valueChange)="onCountryChange($event)">
      </app-slim-select>
      
      <p>Selected Country: {{ selectedCountry }}</p>
      
      <button (click)="loadMoreCountries()">Load More Countries</button>
    </div>
  `
})
export class DemoDynamicComponent implements OnInit {
  countryData: DataArray = [];
  selectedCountry: string = '';
  
  config = {
    placeholder: 'Select a country',
    showSearch: true,
    searchPlaceholder: 'Search countries...'
  };

  private countries: Country[] = [
    { id: 1, name: 'United States', code: 'US' },
    { id: 2, name: 'United Kingdom', code: 'UK' },
    { id: 3, name: 'Canada', code: 'CA' },
    { id: 4, name: 'Australia', code: 'AU' },
    { id: 5, name: 'Germany', code: 'DE' }
  ];

  ngOnInit(): void {
    this.loadCountryData();
  }

  loadCountryData(): void {
    this.countryData = this.countries.map(country => ({
      text: country.name,
      value: country.code,
      data: { id: country.id }
    }));
  }

  loadMoreCountries(): void {
    const newCountries: Country[] = [
      { id: 6, name: 'France', code: 'FR' },
      { id: 7, name: 'Japan', code: 'JP' },
      { id: 8, name: 'Brazil', code: 'BR' }
    ];
    
    this.countries = [...this.countries, ...newCountries];
    this.loadCountryData();
  }

  onCountryChange(value: string | string[]): void {
    this.selectedCountry = value as string;
  }
}
```

### Example 5: Reactive Forms Integration

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { SlimSelectComponent } from './components/slim-select/slim-select.component';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-demo-reactive-form',
  standalone: true,
  imports: [SlimSelectComponent, ReactiveFormsModule, CommonModule],
  template: `
    <div class="demo-container">
      <h2>Reactive Form Integration</h2>
      
      <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
        <div class="form-group">
          <label>Name:</label>
          <input type="text" formControlName="name" class="form-control" />
          <div *ngIf="userForm.get('name')?.invalid && userForm.get('name')?.touched" 
               class="error">
            Name is required
          </div>
        </div>

        <div class="form-group">
          <label>Country:</label>
          <app-slim-select 
            formControlName="country"
            [config]="countryConfig">
            <option value="">Select country</option>
            <option value="us">United States</option>
            <option value="uk">United Kingdom</option>
            <option value="ca">Canada</option>
            <option value="au">Australia</option>
          </app-slim-select>
          <div *ngIf="userForm.get('country')?.invalid && userForm.get('country')?.touched" 
               class="error">
            Country is required
          </div>
        </div>

        <div class="form-group">
          <label>Skills (Multiple):</label>
          <app-slim-select 
            formControlName="skills"
            [multiple]="true"
            [config]="skillsConfig">
            <option value="angular">Angular</option>
            <option value="react">React</option>
            <option value="vue">Vue</option>
            <option value="node">Node.js</option>
            <option value="python">Python</option>
          </app-slim-select>
          <div *ngIf="userForm.get('skills')?.invalid && userForm.get('skills')?.touched" 
               class="error">
            Select at least one skill
          </div>
        </div>

        <button type="submit" [disabled]="userForm.invalid">Submit</button>
        <button type="button" (click)="resetForm()">Reset</button>
      </form>

      <div *ngIf="submittedData" class="result">
        <h3>Submitted Data:</h3>
        <pre>{{ submittedData | json }}</pre>
      </div>
    </div>
  `,
  styles: [`
    .demo-container {
      padding: 20px;
      max-width: 600px;
    }
    
    .form-group {
      margin-bottom: 20px;
    }
    
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    
    .form-control {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    
    .error {
      color: red;
      font-size: 12px;
      margin-top: 5px;
    }
    
    button {
      margin-right: 10px;
      padding: 10px 20px;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    button:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }
    
    button[type="button"] {
      background-color: #6c757d;
    }
    
    .result {
      margin-top: 20px;
      padding: 15px;
      background-color: #f8f9fa;
      border-radius: 4px;
    }
  `]
})
export class DemoReactiveFormComponent implements OnInit {
  userForm!: FormGroup;
  submittedData: any = null;

  countryConfig = {
    placeholder: 'Select your country',
    showSearch: true
  };

  skillsConfig = {
    placeholder: 'Select your skills',
    showSearch: true,
    closeOnSelect: false
  };

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      country: ['', Validators.required],
      skills: [[], Validators.required]
    });
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      this.submittedData = this.userForm.value;
      console.log('Form submitted:', this.submittedData);
    }
  }

  resetForm(): void {
    this.userForm.reset();
    this.submittedData = null;
  }
}
```

### Example 6: Template-Driven Forms

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { SlimSelectComponent } from './components/slim-select/slim-select.component';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-demo-template-form',
  standalone: true,
  imports: [SlimSelectComponent, FormsModule, CommonModule],
  template: `
    <div class="demo-container">
      <h2>Template-Driven Form</h2>
      
      <form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
        <div class="form-group">
          <label>Favorite Framework:</label>
          <app-slim-select 
            name="framework"
            [(ngModel)]="selectedFramework"
            [config]="{ placeholder: 'Select framework' }"
            required>
            <option value="">Choose...</option>
            <option value="angular">Angular</option>
            <option value="react">React</option>
            <option value="vue">Vue</option>
          </app-slim-select>
        </div>

        <div class="form-group">
          <label>Technologies:</label>
          <app-slim-select 
            name="technologies"
            [(ngModel)]="selectedTechnologies"
            [multiple]="true"
            [config]="techConfig"
            required>
            <option value="typescript">TypeScript</option>
            <option value="javascript">JavaScript</option>
            <option value="html">HTML</option>
            <option value="css">CSS</option>
          </app-slim-select>
        </div>

        <button type="submit" [disabled]="!userForm.valid">Submit</button>
      </form>

      <div class="result">
        <p>Selected Framework: {{ selectedFramework }}</p>
        <p>Selected Technologies: {{ selectedTechnologies | json }}</p>
      </div>
    </div>
  `
})
export class DemoTemplateFormComponent {
  selectedFramework: string = '';
  selectedTechnologies: string[] = [];

  techConfig = {
    placeholder: 'Select technologies',
    closeOnSelect: false
  };

  onSubmit(form: any): void {
    console.log('Form submitted:', form.value);
  }
}
```

### Example 7: Async Data Loading

```typescript
import { Component, OnInit } from '@angular/core';
import { SlimSelectComponent } from './components/slim-select/slim-select.component';
import { DataArray } from 'slim-select';
import { CommonModule } from '@angular/common';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  selector: 'app-demo-async',
  standalone: true,
  imports: [SlimSelectComponent, CommonModule],
  template: `
    <div class="demo-container">
      <h2>Async Data Loading</h2>
      
      <app-slim-select 
        [data]="userData"
        [config]="config"
        [disabled]="isLoading"
        (valueChange)="onUserSelect($event)">
      </app-slim-select>
      
      <p *ngIf="isLoading">Loading users...</p>
      <p *ngIf="selectedUser">Selected User: {{ selectedUser }}</p>
    </div>
  `
})
export class DemoAsyncComponent implements OnInit {
  userData: DataArray = [];
  selectedUser: string = '';
  isLoading: boolean = false;

  config = {
    placeholder: 'Select a user',
    showSearch: true,
    searchPlaceholder: 'Search users...'
  };

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading = true;

    // Simulate API call
    setTimeout(() => {
      const users: User[] = [
        { id: 1, name: 'John Doe', email: 'john@example.com' },
        { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
        { id: 3, name: 'Bob Johnson', email: 'bob@example.com' },
        { id: 4, name: 'Alice Williams', email: 'alice@example.com' }
      ];

      this.userData = users.map(user => ({
        text: `${user.name} (${user.email})`,
        value: user.id.toString(),
        data: user
      }));

      this.isLoading = false;
    }, 1500);
  }

  onUserSelect(value: string | string[]): void {
    this.selectedUser = value as string;
    const user = this.userData.find(u => u.value === value);
    console.log('Selected user data:', user?.data);
  }
}
```

## Advanced Features

### Adding Custom Search Logic

```typescript
private initSlimSelect(): void {
  if (!this.selectElement) {
    return;
  }

  this.slimSelectInstance = new SlimSelect({
    select: this.selectElement.nativeElement,
    settings: {
      // ... other settings
    },
    events: {
      search: (search: string, currentData: DataArray) => {
        // Custom search logic
        this.searchChange.emit(search);
        
        // Filter data based on custom logic
        return currentData.filter(item => {
          if ('label' in item) {
            // Handle optgroup
            return item.options.some(opt => 
              opt.text.toLowerCase().includes(search.toLowerCase())
            );
          } else {
            // Handle option
            return item.text.toLowerCase().includes(search.toLowerCase());
          }
        });
      }
    }
  });
}
```

### Handling Async Search

```typescript
@Input() searchFn?: (search: string) => Promise<DataArray>;

private initSlimSelect(): void {
  if (!this.selectElement) {
    return;
  }

  this.slimSelectInstance = new SlimSelect({
    select: this.selectElement.nativeElement,
    settings: {
      // ... other settings
    },
    events: {
      search: async (search: string, currentData: DataArray) => {
        if (this.searchFn) {
          // Call custom async search function
          const results = await this.searchFn(search);
          return results;
        }
        
        // Default search behavior
        return currentData.filter(item => {
          if ('label' in item) {
            return item.options.some(opt => 
              opt.text.toLowerCase().includes(search.toLowerCase())
            );
          } else {
            return item.text.toLowerCase().includes(search.toLowerCase());
          }
        });
      }
    }
  });
}
```

**Usage**:

```typescript
@Component({
  template: `
    <app-slim-select 
      [searchFn]="customSearch"
      [config]="{ placeholder: 'Search...' }">
    </app-slim-select>
  `
})
export class ParentComponent {
  async customSearch(search: string): Promise<DataArray> {
    // Simulate API call
    const response = await fetch(`/api/search?q=${search}`);
    const data = await response.json();
    
    return data.map((item: any) => ({
      text: item.name,
      value: item.id
    }));
  }
}
```

## Best Practices

### 1. Proper Cleanup

Always destroy the Slim Select instance in `ngOnDestroy` to prevent memory leaks:[3]

```typescript
ngOnDestroy(): void {
  this.destroySlimSelect();
}

private destroySlimSelect(): void {
  if (this.slimSelectInstance) {
    this.slimSelectInstance.destroy();
    this.slimSelectInstance = undefined;
  }
}
```

### 2. Handle Change Detection

For complex scenarios, trigger change detection manually:

```typescript
import { ChangeDetectorRef } from '@angular/core';

constructor(private cdr: ChangeDetectorRef) {}

private initSlimSelect(): void {
  this.slimSelectInstance = new SlimSelect({
    // ...
    events: {
      afterChange: (newVal) => {
        this.onChange(newVal);
        this.cdr.detectChanges(); // Trigger change detection
      }
    }
  });
}
```

### 3. Type Safety

Define proper TypeScript interfaces for your data:

```typescript
export interface SelectOption {
  text: string;
  value: string;
  selected?: boolean;
  disabled?: boolean;
  placeholder?: boolean;
  class?: string;
  style?: string;
  data?: Record<string, any>;
}

export interface SelectOptGroup {
  label: string;
  selectAll?: boolean;
  closable?: 'off' | 'open' | 'close';
  options: SelectOption[];
}

export type SelectData = (SelectOption | SelectOptGroup)[];
```

### 4. Error Handling

Add error handling for edge cases:

```typescript
private initSlimSelect(): void {
  try {
    if (!this.selectElement?.nativeElement) {
      console.error('SlimSelect: Select element not found');
      return;
    }

    this.slimSelectInstance = new SlimSelect({
      select: this.selectElement.nativeElement,
      // ... configuration
    });
  } catch (error) {
    console.error('SlimSelect initialization failed:', error);
  }
}
```

### 5. Accessibility

Ensure the component is accessible:

```html
<label [for]="selectId">{{ label }}</label>
<select 
  #selectElement 
  [id]="selectId"
  [attr.aria-label]="ariaLabel"
  [attr.aria-required]="required"
  [multiple]="multiple"
  [disabled]="disabled">
  <ng-content></ng-content>
</select>
```

```typescript
@Input() label?: string;
@Input() selectId: string = `slim-select-${Math.random().toString(36).substr(2, 9)}`;
@Input() ariaLabel?: string;
@Input() required: boolean = false;
```

## Styling and Customization

### Global Styles

In your `styles.css` or `styles.scss`:

```css
/* Import Slim Select base styles */
@import 'slim-select/styles';

/* Custom theme */
.ss-main {
  border-radius: 8px;
  border: 2px solid #e0e0e0;
}

.ss-main:focus {
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.ss-content {
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.ss-option {
  padding: 12px 16px;
}

.ss-option:hover {
  background-color: #f0f0f0;
}

.ss-option.ss-selected {
  background-color: #007bff;
  color: white;
}
```

### Component-Specific Styles

```css
/* slim-select.component.css */
:host {
  display: block;
  width: 100%;
}

/* Custom styling for disabled state */
:host(.disabled) ::ng-deep .ss-main {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Custom styling for error state */
:host(.error) ::ng-deep .ss-main {
  border-color: #dc3545;
}
```

## Testing

### Unit Test Example

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { SlimSelectComponent } from './slim-select.component';

describe('SlimSelectComponent', () => {
  let component: SlimSelectComponent;
  let fixture: ComponentFixture<SlimSelectComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [SlimSelectComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(SlimSelectComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should initialize Slim Select after view init', () => {
    fixture.detectChanges();
    expect(component['slimSelectInstance']).toBeDefined();
  });

  it('should destroy Slim Select on component destroy', () => {
    fixture.detectChanges();
    const destroySpy = jasmine.createSpy('destroy');
    if (component['slimSelectInstance']) {
      component['slimSelectInstance'].destroy = destroySpy;
    }
    
    component.ngOnDestroy();
    expect(destroySpy).toHaveHaveBeenCalled();
  });

  it('should emit valueChange event on selection', (done) => {
    fixture.detectChanges();
    
    component.valueChange.subscribe((value: string | string[]) => {
      expect(value).toBe('test');
      done();
    });

    // Simulate selection change
    if (component['slimSelectInstance']) {
      component['slimSelectInstance'].setSelected('test');
    }
  });
});
```

## Complete Working Example: User Management Form

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { SlimSelectComponent, SlimSelectConfig } from './components/slim-select/slim-select.component';
import { CommonModule } from '@angular/common';
import { DataArray } from 'slim-select';

interface Department {
  id: string;
  name: string;
  description: string;
}

interface Role {
  id: string;
  name: string;
}

@Component({
  selector: 'app-user-management',
  standalone: true,
  imports: [SlimSelectComponent, ReactiveFormsModule, CommonModule],
  template: `
    <div class="user-management">
      <h1>User Management</h1>
      
      <form [formGroup]="userForm" (ngSubmit)="onSubmit()" class="user-form">
        <div class="form-row">
          <div class="form-group">
            <label for="firstName">First Name *</label>
            <input 
              type="text" 
              id="firstName"
              formControlName="firstName" 
              class="form-control"
              [class.is-invalid]="isFieldInvalid('firstName')" />
            <div class="invalid-feedback" *ngIf="isFieldInvalid('firstName')">
              First name is required
            </div>
          </div>

          <div class="form-group">
            <label for="lastName">Last Name *</label>
            <input 
              type="text" 
              id="lastName"
              formControlName="lastName" 
              class="form-control"
              [class.is-invalid]="isFieldInvalid('lastName')" />
            <div class="invalid-feedback" *ngIf="isFieldInvalid('lastName')">
              Last name is required
            </div>
          </div>
        </div>

        <div class="form-group">
          <label for="email">Email *</label>
          <input 
            type="email" 
            id="email"
            formControlName="email" 
            class="form-control"
            [class.is-invalid]="isFieldInvalid('email')" />
          <div class="invalid-feedback" *ngIf="isFieldInvalid('email')">
            <span *ngIf="userForm.get('email')?.errors?.['required']">Email is required</span>
            <span *ngIf="userForm.get('email')?.errors?.['email']">Invalid email format</span>
          </div>
        </div>

        <div class="form-group">
          <label for="department">Department *</label>
          <app-slim-select 
            formControlName="department"
            [data]="departmentData"
            [config]="departmentConfig">
          </app-slim-select>
          <div class="invalid-feedback d-block" *ngIf="isFieldInvalid('department')">
            Department is required
          </div>
        </div>

        <div class="form-group">
          <label for="roles">Roles *</label>
          <app-slim-select 
            formControlName="roles"
            [data]="roleData"
            [multiple]="true"
            [config]="roleConfig">
          </app-slim-select>
          <div class="invalid-feedback d-block" *ngIf="isFieldInvalid('roles')">
            At least one role is required
          </div>
        </div>

        <div class="form-group">
          <label for="status">Status *</label>
          <app-slim-select 
            formControlName="status"
            [config]="{ placeholder: 'Select status' }">
            <option value="">Choose status...</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
            <option value="suspended">Suspended</option>
          </app-slim-select>
          <div class="invalid-feedback d-block" *ngIf="isFieldInvalid('status')">
            Status is required
          </div>
        </div>

        <div class="form-actions">
          <button type="submit" class="btn btn-primary" [disabled]="isSubmitting">
            {{ isSubmitting ? 'Saving...' : 'Save User' }}
          </button>
          <button type="button" class="btn btn-secondary" (click)="resetForm()">
            Reset
          </button>
        </div>
      </form>

      <div *ngIf="savedUser" class="result-card">
        <h2>User Saved Successfully!</h2>
        <pre>{{ savedUser | json }}</pre>
      </div>
    </div>
  `,
  styles: [`
    .user-management {
      max-width: 800px;
      margin: 0 auto;
      padding: 30px;
    }

    h1 {
      margin-bottom: 30px;
      color: #333;
    }

    .user-form {
      background: white;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    .form-row {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      margin-bottom: 20px;
    }

    .form-group {
      margin-bottom: 20px;
    }

    label {
      display: block;
      margin-bottom: 8px;
      font-weight: 600;
      color: #555;
    }

    .form-control {
      width: 100%;
      padding: 10px 12px;
      border: 2px solid #e0e0e0;
      border-radius: 6px;
      font-size: 14px;
      transition: border-color 0.2s;
    }

    .form-control:focus {
      outline: none;
      border-color: #007bff;
    }

    .form-control.is-invalid {
      border-color: #dc3545;
    }

    .invalid-feedback {
      display: none;
      color: #dc3545;
      font-size: 12px;
      margin-top: 5px;
    }

    .invalid-feedback.d-block {
      display: block;
    }

    .form-actions {
      display: flex;
      gap: 10px;
      margin-top: 30px;
    }

    .btn {
      padding: 12px 24px;
      border: none;
      border-radius: 6px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.2s;
    }

    .btn-primary {
      background-color: #007bff;
      color: white;
    }

    .btn-primary:hover:not(:disabled) {
      background-color: #0056b3;
    }

    .btn-primary:disabled {
      background-color: #ccc;
      cursor: not-allowed;
    }

    .btn-secondary {
      background-color: #6c757d;
      color: white;
    }

    .btn-secondary:hover {
      background-color: #545b62;
    }

    .result-card {
      margin-top: 30px;
      padding: 20px;
      background: #d4edda;
      border: 1px solid #c3e6cb;
      border-radius: 6px;
    }

    .result-card h2 {
      color: #155724;
      margin-bottom: 15px;
      font-size: 18px;
    }

    .result-card pre {
      background: white;
      padding: 15px;
      border-radius: 4px;
      overflow-x: auto;
    }
  `]
})
export class UserManagementComponent implements OnInit {
  userForm!: FormGroup;
  departmentData: DataArray = [];
  roleData: DataArray = [];
  savedUser: any = null;
  isSubmitting: boolean = false;

  departmentConfig: SlimSelectConfig = {
    placeholder: 'Select department',
    showSearch: true,
    searchPlaceholder: 'Search departments...'
  };

  roleConfig: SlimSelectConfig = {
    placeholder: 'Select roles',
    showSearch: true,
    closeOnSelect: false,
    searchPlaceholder: 'Search roles...'
  };

  private departments: Department[] = [
    { id: 'eng', name: 'Engineering', description: 'Software development' },
    { id: 'hr', name: 'Human Resources', description: 'People management' },
    { id: 'sales', name: 'Sales', description: 'Revenue generation' },
    { id: 'marketing', name: 'Marketing', description: 'Brand promotion' },
    { id: 'finance', name: 'Finance', description: 'Financial operations' }
  ];

  private roles: Role[] = [
    { id: 'admin', name: 'Administrator' },
    { id: 'manager', name: 'Manager' },
    { id: 'developer', name: 'Developer' },
    { id: 'analyst', name: 'Analyst' },
    { id: 'support', name: 'Support Staff' }
  ];

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.initForm();
    this.loadDepartments();
    this.loadRoles();
  }

  private initForm(): void {
    this.userForm = this.fb.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      department: ['', Validators.required],
      roles: [[], Validators.required],
      status: ['', Validators.required]
    });
  }

  private loadDepartments(): void {
    this.departmentData = this.departments.map(dept => ({
      text: dept.name,
      value: dept.id,
      data: { description: dept.description }
    }));
  }

  private loadRoles(): void {
    this.roleData = this.roles.map(role => ({
      text: role.name,
      value: role.id
    }));
  }

  isFieldInvalid(fieldName: string): boolean {
    const field = this.userForm.get(fieldName);
    return !!(field && field.invalid && (field.dirty || field.touched));
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      this.isSubmitting = true;

      // Simulate API call
      setTimeout(() => {
        this.savedUser = this.userForm.value;
        this.isSubmitting = false;
        console.log('User saved:', this.savedUser);
      }, 1000);
    } else {
      // Mark all fields as touched to show validation errors
      Object.keys(this.userForm.controls).forEach(key => {
        this.userForm.get(key)?.markAsTouched();
      });
    }
  }

  resetForm(): void {
    this.userForm.reset();
    this.savedUser = null;
  }
}
```

## Summary

You've now created a fully functional, reusable Angular component wrapper for Slim Select. Key achievements:[1][4]

### What We've Built

**Core Features**:
- ✅ Proper Angular lifecycle integration (`ngAfterViewInit`, `ngOnDestroy`)
- ✅ Full ControlValueAccessor implementation for forms
- ✅ Support for both Reactive and Template-driven Forms
- ✅ Single and multiple select modes
- ✅ Dynamic data loading
- ✅ Type-safe configuration
- ✅ Event emissions for change detection
- ✅ Public API methods for programmatic control

### Key Takeaways

**Lifecycle Management**:[5][3]
- Use `ngAfterViewInit` to initialize third-party libraries after DOM is ready
- Always clean up in `ngOnDestroy` to prevent memory leaks
- Handle `ngOnChanges` for dynamic data updates

**Forms Integration**:
- Implement `ControlValueAccessor` for seamless Angular Forms integration
- Provide proper validation state feedback
- Support both programmatic and user-driven value changes

**Best Practices**:
- Type safety with TypeScript interfaces
- Proper error handling
- Accessibility considerations
- Clean API surface
- Comprehensive testing

This component can now be used throughout your Angular application just like any native form control, with full support for validation, form states, and reactive updates.[2][1]
