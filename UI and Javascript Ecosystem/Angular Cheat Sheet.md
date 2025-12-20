# Angular Cheat Sheet

## **Core Concepts**

**Angular:** Angular is a TypeScript-based open-source web application framework developed by Google for building scalable single-page applications with a component-based architecture and powerful dependency injection system.

**Component Architecture:** Angular applications are built as trees of components, where each component encapsulates its template, styles, and logic, promoting reusability and maintainability through hierarchical composition and clear separation of concerns.

**TypeScript Integration:** Angular is built with TypeScript, providing strong typing, better IDE support, compile-time error checking, and modern JavaScript features like decorators, interfaces, and generics.

**Modules (NgModules):** NgModules are containers that group related components, directives, pipes, and services together, providing compilation context and organizing the application into cohesive blocks of functionality.

**Dependency Injection:** Angular’s hierarchical dependency injection system provides services and dependencies to components and other services, promoting loose coupling and testability through constructor injection and providers.

## Component Fundamentals

**Component Definition:** Components are the building blocks of Angular applications, defined using the@Component decorator with template, styles, and selector properties, encapsulating both view and behavior logic.

**Component Lifecycle:** Angular components follow a defined lifecycle from creation to destruction, with hooks like ngOnInit, ngOnChanges, ngAfterViewInit, and ngOnDestroy providing precise control over component behavior at different stages.

**Data Binding:** Angular supports four types of data binding — interpolation ({{}}), property binding(\[property\]), event binding ((event)), and two-way binding (\[(ngModel)\]) — enabling dynamic interaction between component class and template.

**Input and Output:** Components communicate through @Input() for passing data from parent to child and @Output() with EventEmitter for child-to-parent communication, maintaining unidirectional dataflow.

## Templates and Data Binding

**Template Syntax:** Angular templates use HTML enhanced with Angular-specific syntax including structural directives (\*ngIf, \*ngFor), attribute directives (\[ngClass\], \[ngStyle\]), and binding expressions for dynamic content rendering.

**Interpolation:** Text interpolation using double curly braces {{}} evaluates expressions in the component context and renders the result as text, with automatic escaping for security.

**Property Binding:** Square bracket syntax \[property\] binds component properties to element attributes, enabling dynamic updates of DOM properties, CSS classes, and styles.

**Event Binding:** Parentheses syntax (event) binds DOM events to component methods, allowing user interactions to trigger component logic with automatic change detection.

**Two-Way Binding:** Banana-in-a-box syntax \[(property)\] combines property and event binding for form controls, most commonly used with \[(ngModel)\] for form inputs.

**Template Reference Variables:** Hash symbol (#variable) creates references to DOM elements or components in templates, enabling access to element properties and methods within the template.

## Directives

**Structural Directives:** Directives that change the DOM structure by adding, removing, or manipulating elements, including \*ngIf for conditional rendering, \*ngFor for lists, and \*ngSwitch for multiple conditions.

**Attribute Directives:** Directives that modify the appearance or behavior of existing elements, such as\[ngClass\] for dynamic CSS classes, \[ngStyle\] for inline styles, and custom attribute directives.

**Custom Directives:** User-defined directives created with @Directive decorator for reusable DOM manipulation, event handling, or element enhancement across the application.

## Services and Dependency Injection

**Services:** Services are singleton objects that provide specific functionality across the application, typically for data access, business logic, logging, or shared utilities, decorated with @Injectable.

**Dependency Injection Hierarchy:** Angular’s DI system creates hierarchical injectors at different levels(platform, application, module, component), with services resolved from the closest injector up the hierarchy.

**Injectable Decorator:** @Injectable() decorator marks classes as available for dependency injection, with providedIn property controlling service scope and lazy loading behavior.

**Provider Configuration:** Providers tell injectors how to create service instances, with options for useClass, useValue, useFactory, and useExisting to control instantiation behavior.

**Injection Tokens:** InjectionToken creates unique tokens for non-class dependencies, enabling injection of primitive values, configuration objects, or third-party libraries.

## Reactive Programming with RxJS

**Observables:** Observables represent streams of data over time, providing powerful operators for data transformation, error handling, and asynchronous operations with built-in subscription management.

**HTTP Client:** Angular’s HttpClient service returns Observables for HTTP requests, supporting interceptors, request/response transformation, and error handling through RxJS operators.

**RxJS Operators:** Transform, filter, and combine data streams using operators like map, filter, switchMap, mergeMap, debounceTime, distinctUntilChanged, and combineLatest for complex async operations.

**Subject Types:** BehaviorSubject stores current value, ReplaySubject caches multiple values, AsyncSubject emits only the last value, and regular Subject provides basic multicasting for component communication.

## Subject vs Behavior Subject Deep Dive

**Subject:** Basic multicast observable that acts as both observer and observable. Subjects don’t store values- subscribers only receive values emitted after subscription, making them ideal for event-like communications.

**BehaviorSubject:** Stores the current value and immediately emits it to new subscribers, ensuring all subscribers have access to the latest state. Requires an initial value and always has a current value available.

**Key Differences:**

1.  **Initial Value:** BehaviorSubject requires initial value, Subject doesn’t
2.  **Late Subscribers:** BehaviorSubject gives latest value to late subscribers, Subject doesn’t
3.  **Current Value:** BehaviorSubject provides .value property for synchronous access
4.  **Use Cases:** BehaviorSubject for state management, Subject for events

**Subscription Management:** Proper subscription cleanup prevents memory leaks using unsubscribe(), takeUntil pattern with destroy subjects, or async pipe for automatic subscription handling.

**Async Pipe:** Angular’s async pipe automatically handles subscription and unsubscription, eliminating manual memory management and reducing boilerplate code.

## Forms

**Template-Driven Forms:** Forms built in templates using ngModel, ngForm, and validation directives, suitable for simple forms with straightforward validation requirements and minimal dynamic behavior.

**Reactive Forms:** Forms built programmatically with FormControl, FormGroup, and FormArray, providing better control over validation, dynamic form generation, and complex form interactions.

**Form Validation:** Built-in validators (required, email, minLength, pattern) and custom validators for complex validation logic, with real-time validation feedback and error message display.

**Dynamic Forms:** Programmatically create form controls and validation based on configuration objects, enabling forms that adapt to changing requirements or user permissions.

## Routing

**Router Configuration:** Define routes using RouterModule.forRoot() with path, component, and optional parameters like guards, resolvers, and data for navigation configuration.

**Route Parameters:** Access route parameters using ActivatedRoute service with params observable for dynamic parameters and queryParams for URL query strings.

**Route Guards:** Implement CanActivate, CanDeactivate, CanLoad, and Resolve interfaces to control access to routes, handle unsaved changes, and pre-load data.

**Lazy Loading:** Load feature modules on-demand using dynamic imports in route configuration, reducing initial bundle size and improving application startup performance.

## Module-wise Code Splitting

**Why Module Splitting is Essential:** Module-wise splitting divides your application into logical, feature-based chunks that are loaded only when needed, dramatically improving initial load times, reducing memory consumption, and enabling better caching strategies for unchanged features.

**Performance Benefits of Module Splitting:**

1.  **Reduced Initial Bundle Size:** Only core modules load initially, with feature modules loading on-demand
2.  **Faster Time to Interactive (TTI):** Smaller initial JavaScript bundles execute faster
3.  **Better Browser Caching:** Individual modules can be cached separately, so unchanged features don’t need re-downloading
4.  **Improved Core Web Vitals:** Reduced Largest Contentful Paint (LCP) and First Input Delay (FID)
5.  **Memory Efficiency:** Unused features don’t consume memory until accessed

**Security Benefits of Module Splitting:**

1.  **Reduced Attack Surface:** Users only download code for features they access
2.  **Code Isolation:** Sensitive admin features can be completely separated from public features
3.  **Permission-based Loading:** Modules load only if user has appropriate permissions
4.  **Audit Trail:** Easier to track which code modules users actually access
5.  **Vulnerability Containment:** Security issues in one module don’t affect others immediately

**Navigation:** Programmatic navigation using Router service with navigate() and navigateByUrl() methods, supporting relative navigation and navigation extras like query parameters.

## State Management

**Component State:** Local component state using class properties for simple scenarios, with change detection automatically updating the view when properties change.

**Service State:** Shared state through services with BehaviorSubject or state management patterns, providing centralized data access across multiple components.

**NgRx:** Redux-inspired state management with actions, reducers, effects, and selectors for complex applications requiring predictable state updates and time-travel debugging.

## Change Detection

**Zone.js:** Angular’s change detection mechanism that patches asynchronous operations (setTimeout, Promise, events) to automatically trigger change detection cycles.

**Change Detection Strategy:** OnPush strategy improves performance by checking components only when inputs change or events occur, reducing unnecessary checks in large component trees.

**Manual Change Detection:** Use ChangeDetectorRef to manually control change detection withdetectChanges(), markForCheck(), and detach() for fine-grained performance optimization.

**Immutable Data:** Using immutable data structures with OnPush strategy enables efficient change detection by reference comparison instead of deep object inspection.

## Testing

**Testing Framework:** Angular uses Jasmine for test syntax and Karma for test execution, with Angular Testing Utilities providing specialized tools for component and service testing.

**Component Testing:** TestBed configures testing modules, Component Fixture provides component instances and DOM access, and DebugElement enables DOM querying and event triggering.

**Service Testing:** Test services in isolation using TestBed.inject() for dependency injection and HttpClientTestingModule for HTTP request mocking and verification.

**E2E Testing:** Protractor (deprecated) or Cypress for end-to-end testing, validating complete user workflows and cross-browser compatibility.

**Test Doubles:** Use spies (jasmine.createSpy), stubs, and mock objects to isolate units under test and control external dependencies for reliable testing.

## Testing Best Practices

**Unit Testing Strategy:** Test components in isolation using shallow rendering, mock dependencies, and focus on component logic rather than implementation details.

**Integration Testing:** Test component interactions, service integration, and data flow between related components using TestBed configuration.

**HTTP Testing:** Use HttpClientTestingModule to mock HTTP requests and verify correct API calls with proper error handling and data transformation.

**Async Testing:** Handle asynchronous operations using fakeAsync, tick, flush, and async/await patterns for reliable test execution.

## Performance Optimization

**OnPush Change Detection:** Optimize performance by using OnPush change detection strategy to reduce checking frequency and improve rendering performance in large applications.

**Lazy Loading:** Implement feature module lazy loading to reduce initial bundle size and improve application startup time through route-based code splitting.

**Track By Functions:** Use trackBy functions in \*ngFor to help Angular identify list items, reducing DOM manipulation when arrays change and improving rendering performance.

**Pure Pipes (Default Behavior):** Pure pipes are executed only when Angular detects a pure change to the input value — when a primitive input value changes or when an object reference changes. Angular caches the pipe results and returns cached values for identical inputs, providing excellent performance optimization.

**Impure Pipes:** Impure pipes execute on every change detection cycle, regardless of whether inputs have changed. They detect changes within objects or external state changes but have significant performance implications.

**When to Use Each:**

**Pure Pipes:** Data transformation, formatting, filtering with immutable data (95% of use cases)

**Impure Pipes:** Real-time data that changes frequently, external state dependencies, complex object filtering

**Virtual Scrolling:** Use Angular CDK’s virtual scrolling for large lists to render only visible items, dramatically improving performance with large datasets.

**Bundle Analysis:** Analyze bundle size using webpack-bundle-analyzer and Angular CLI’s build analyzer to identify optimization opportunities and reduce application size.

## Security

**XSS Prevention:** Angular automatically sanitizes HTML content to prevent cross-site scripting attacks, with DomSanitizer service for bypassing sanitization when necessary.

**CSRF Protection:** Implement CSRF tokens in HTTP requests using Angular’s HttpClient with proper interceptors for state-changing operations.

**Content Security Policy:** Configure CSP headers to prevent XSS attacks and control resource loading, with Angular supporting CSP-compliant inline styles and scripts.

**Trusted Types:** Use Trusted Types API with Angular’s sanitization system to prevent DOM-based XSS vulnerabilities in modern browsers.

**Authentication:** Implement JWT-based authentication with HTTP interceptors for automatic token inclusion and route guards for access control.

## HTTP Interceptors Deep Dive

**What are Interceptors:** HTTP interceptors are middleware functions that intercept HTTP requests and responses, allowing you to transform requests, add headers, handle errors, implement caching, logging, and authentication tokens automatically across your entire application.

**Request Interceptors:** Modify outgoing requests by adding authentication tokens, setting content types, adding correlation IDs, or transforming request data before sending to the server.

**Response Interceptors:** Handle responses globally for error handling, token refresh, response transformation, or logging.

## Advanced Patterns

**Dynamic Component Loading:** Load components at runtime using ComponentFactory and ViewContainerRef for flexible UI composition and plugin architectures.

**Content Projection:** Use ng-content for flexible component composition, supporting single slot, multi-slot, and conditional projection patterns.

**Custom Elements:** Angular Elements package enables creating custom elements from Angular components for use in non-Angular applications or micro-frontend architectures.

**Micro-Frontends:** Module Federation with Angular enables building scalable micro-frontend architectures with independent deployments and technology diversity.

**Internationalization (i18n):** Angular i18n package provides comprehensive internationalization support with extraction, translation, and build-time locale-specific bundling.

## Standalone vs Module-Based Architecture

**Traditional Module-Based Architecture:** NgModules organize related components, directives, pipes, and services into cohesive units with declarations, imports, providers, and exports arrays, providing compilation context and dependency management.

**Standalone Component Architecture:** Components import their dependencies directly without NgModules, reducing boilerplate and providing more granular control over component dependencies.

## Architecture Patterns

**Feature Modules:** Organize related components, services, and routes into feature modules for better code organization and lazy loading capabilities.

**Shared Modules:** Create shared modules for common components, directives, and pipes used across multiple features, avoiding code duplication.

**Core Module:** Singleton services and application-wide providers belong in core module, imported only once in AppModule with guard against multiple imports.

**Barrel Exports:** Use index.ts files to create clean import paths and encapsulate module internals, improving code organization and refactoring capabilities.

**Smart/Dumb Components:** Separate presentation components (dumb) from container components(smart) for better reusability, testing, and maintenance.

## Performance Monitoring

**Angular DevTools:** Browser extension for debugging Angular applications, inspecting component tree, and analyzing performance bottlenecks.

**Bundle Analysis:** Analyze application bundles using source-map-explorer or webpack-bundle-analyzer to identify large dependencies and optimization opportunities.

**Runtime Performance:** Monitor application performance using browser developer tools, Angular profiler, and Web Vitals metrics.

**Memory Management:** Prevent memory leaks by properly unsubscribing from observables, removing event listeners, and cleaning up resources in ngOnDestroy.

**Lazy Loading Optimization:** Implement progressive loading strategies with route-based and feature-based lazy loading for optimal user experience.