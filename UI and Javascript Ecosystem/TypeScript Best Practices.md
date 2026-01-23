# TypeScript Best Practices You’re Probably Not Following (But Should)
**These small habits quietly separate clean codebases from painful ones**

![TypeScript Best Practices Header](https://miro.medium.com/v2/resize:fit:720/format:webp/1*HL2ZflOZOYOP8oo0E59C3g.png)

I’ve been using TypeScript for years. And for a long time, I thought that was enough. Types were there. Errors were fewer. The compiler was happy. So I assumed I was “doing TypeScript right.”

Turns out, I was mostly just **using TypeScript**, not **using it well**.

TypeScript doesn’t magically improve your code just because it’s present. The real benefits come from how you structure types, where you enforce boundaries, and how much you let the compiler help you instead of fighting it.

Here are the TypeScript best practices I wish I had followed earlier.

---

### 1. Stop Using `any` Just to Make Errors Go Away
// 2019 was 6 years ago, bro
```typescript
function processData(data: any) {
  return data.map(item => item.name.toUpperCase());
}
```

**What seniors write:**
// Proper generics + constraints
```typescript
function processData<T extends { name: string }>(data: T[]): Uppercase<T["name"]>[] {
  return data.map(item => item.name.toUpperCase() as Uppercase<T["name"]>);
}
```

**Or even better — just use the right type:**
```typescript
interface User { name: string; }
function processUsers(users: User[]): string[] {
  return users.map(u => u.name.toUpperCase());
}
```
**Rule:** If you write `any`, you’ve already lost.

---

### 2. You’re Using `as unknown as Type` (TypeScript Laundering)
// The infamous triple assertion
```typescript
const user = apiResponse as unknown as User;
```
This is literally saying: “I don’t trust TypeScript, so I’m bypassing it.”

**What seniors do:**
// Proper type guard
```typescript
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'name' in obj && 'email' in obj;
}

if (isUser(apiResponse)) {
  console.log(apiResponse.email); // Fully typed!
}
```



---

### 3. Your Interfaces Are Bleeding Everywhere
// One giant interface used by 17 components
```typescript
interface UserProfile {
  id: number;
  name: string;
  email: string;
  password: string;  // Why is this here??
  preferences: UserPreferences;
  notifications: NotificationSettings;
}
```

**Seniors split them:**
```typescript
interface UserAuth { id: number; email: string; password: string; }
interface UserPublic { id: number; name: string; email: string; }
interface UserPreferences { theme: 'dark' | 'light'; }
interface UserNotifications { email: boolean; push: boolean; }

type UserProfile = UserPublic & { preferences: UserPreferences; notifications: UserNotifications };
```
**Rule:** One interface = one responsibility.

---

### 4. You’re Not Using `satisfies`
// Old way — lose type information
```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
} as const;
```

**New hotness - keep the type AND satisfy the interface**
```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
} satisfies Record<string, string | number>;
```
Now you get autocomplete AND type safety.

---

### 5. You’re Still Using optional chaining Wrong
// This lies to you
```typescript
user.profile?.address?.street?.length
```
// What if address is an empty string? // This returns 0, not undefined!

**Seniors use explicit checks:**
```typescript
if (user.profile?.address && typeof user.profile.address === 'object') {
  // Now safe
}
```
**Or better — proper types:**
```typescript
type Address = { street: string; city: string };
type Profile = { address?: Address | null };
```

---

### 6. Your Enums Are Silent Killers
// Runtime bloat + no type safety
```typescript
enum UserRole { Admin = "admin", User = "user" }
```

**Seniors use unions**
```typescript
type UserRole = "admin" | "user" | "moderator";
```

**Or const assertions for objects**
```typescript
const UserRoles = {
  Admin: "admin",
  User: "user",
  Moderator: "moderator"
} as const;

type UserRole = typeof UserRoles[keyof typeof UserRoles];
```
**Bonus:** No more `UserRole[userRole]` nonsense.

---

### 7. You’re Not Using Mapped Types (You’re Missing Magic)
// Manual work
```typescript
type UserResponse = {
  userId: number;
  userName: string;
  userEmail: string;
}
```

**2025 way**
```typescript
type User = { id: number; name: string; email: string };
type UserResponse = { [K in keyof User as `user${Capitalize<K>}`]: User[K] };
// → { userId: number; userName: string; userEmail: string }
```

---

### 8. Your Utility Types Are Basic
// You use Partial<T>, Pick<T>, Omit<T>  
**Seniors use these:**

```typescript
type DeepPartial<T> = T extends object ? { [P in keyof T]?: DeepPartial<T[P]> } : T;
type DeepRequired<T> = T extends object ? { [P in keyof T]-?: DeepRequired<T[P]> } : T;
type DeepReadonly<T> = T extends object ? { readonly [P in keyof T]: DeepReadonly<T[P]> } : T;

type PropsOf<T> = T extends React.ComponentType<infer P> ? P : never;
```

---

### 9. You’re Not Using Template Literal Types
// Boring
```typescript
type EventType = "click" | "hover" | "submit";
```

**flex**
```typescript
type EventType = `${"on"}${Capitalize<"click" | "hover" | "submit">}`;
// → "onClick" | "onHover" | "onSubmit"
```
Perfect for API routes, CSS classes, etc.

---

### 10. Your Error Handling Types Suck
// Weak
```typescript
function apiCall(): Promise<User | Error>
```

**Strong**
```typescript
type ApiResponse<T> = { success: true; data: T } | { success: false; error: string };

function apiCall(): Promise<ApiResponse<User>>
```
Now you’re forced to handle both cases.

---

### 11. You’re Not Using Branded Types
// These look the same but shouldn't be
```typescript
type UserId = string;
type PostId = string;
```

**Seniors prevent bugs at compile time**
```typescript
type UserId = string & { __brand: "UserId" };
type PostId = string & { __brand: "PostId" };

function getUser(id: UserId) { /* ... */ }

// Compile error if you pass PostId!
getUser("123" as PostId); // Type error
```

---

### 12. Your `tsconfig.json` Is From 2020
// Your current config
```json
{
  "strict": true
}
```

**2025 senior config**
```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "strictFunctionTypes": true,
  "strictBindCallApply": true,
  "noImplicitThis": true,
  "useUnknownInCatchVariables": true,
  "noUnusedLocals": true,
  "noUnusedParameters": true,
  "exactOptionalPropertyTypes": true,
  "noPropertyAccessFromIndexSignature": true,
  "noUncheckedIndexedAccess": true,
  "noImplicitOverride": true
}
```



Yes, it will hurt at first. Then it will save you.

If your TypeScript feels noisy, restrictive, or annoying, that’s usually a sign that it’s being bypassed instead of used. Slow down. Lean into the compiler. Let it do its job.

That’s when TypeScript stops feeling like overhead and starts feeling like a superpower.

If this helped you rethink how you use TypeScript, give it a clap. A lot of teams are still fighting the type system instead of working with it.