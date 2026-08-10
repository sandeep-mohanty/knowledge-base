# 💼 Tutorial: ASP.NET Core Authorization with Policies, Requirements, and Custom Handlers  
### Business Scenario – Expense Report Approval (Manager vs. CFO)

In this tutorial, we’ll dive deeply into ASP.NET Core’s **policy-based authorization** to implement a real-world scenario:

- **Employees** submit **Expense Reports**.  
- **Managers** can approve expenses up to `$1000`.  
- **CFO** role is required for expenses above `$1000`.  

We’ll use **Policies + Requirements + Handlers** with **clean extension methods** for maintainability.  

---

## 1. The Big Picture Flow

Here’s the mental model of how ASP.NET Core evaluates authorization:

```plaintext
+-----------------+        +-------------------+        +---------------------+        +-----------------------+
|   Controller    |        | Policy "Approve"  |        |  Requirements       |        |   Handlers (logic)    |
|   Action        |  -->   | Def: ApproveExp.  |  -->   |  MinRole, Amount    |  -->   |  Manager vs CFO logic |
+-----------------+        +-------------------+        +---------------------+        +-----------------------+
         |
         V
   ✅ Allow if requirements succeed
   ❌ Deny if any fail
```

---

## 2. The Domain Model

Let’s define a simple `ExpenseReport`:

```csharp
public class ExpenseReport
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public string SubmittedBy { get; set; } // UserId
}
```

Employees submit these. We now define rules to protect **approval actions**.

---

## 3. Defining the Requirements

Requirements are just “data contracts” — no logic.

```csharp
using Microsoft.AspNetCore.Authorization;

public class ExpenseApprovalRequirement : IAuthorizationRequirement
{
    public decimal MaximumAllowed { get; }

    public ExpenseApprovalRequirement(decimal maximumAllowed)
    {
        MaximumAllowed = maximumAllowed;
    }
}
```

This requirement communicates:  
> *The user must have authority to approve expenses up to `$MaximumAllowed`.*

---

## 4. Writing Custom Handlers

We need **two handlers**: one for **Managers**, one for **CFO**.

### 4.1 Manager Handler
```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;

public class ManagerExpenseHandler 
    : AuthorizationHandler<ExpenseApprovalRequirement, ExpenseReport>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ExpenseApprovalRequirement requirement,
        ExpenseReport resource)
    {
        if (context.User.IsInRole("Manager") &&
            resource.Amount <= requirement.MaximumAllowed)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

### 4.2 CFO Handler
```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;

public class CFOExpenseHandler
    : AuthorizationHandler<ExpenseApprovalRequirement, ExpenseReport>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context,
        ExpenseApprovalRequirement requirement,
        ExpenseReport resource)
    {
        if (context.User.IsInRole("CFO") &&
            resource.Amount > requirement.MaximumAllowed)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

---

## 5. Policy Extensions (Cleaner than Lambdas)

Instead of `policy.Requirements.Add(new ...)` all over the place, we centralize:

```csharp
public static class ExpenseAuthorizationPolicies
{
    public static AuthorizationOptions AddExpensePolicies(this AuthorizationOptions options)
    {
        // Policy: Manager can approve ≤ $1000, CFO can approve > $1000
        options.AddPolicy("CanApproveExpenses", policy =>
        {
            // Requirement says: the maximum amount managers can approve
            policy.Requirements.Add(new ExpenseApprovalRequirement(1000));
        });

        return options;
    }
}
```

---

## 6. Register in Program.cs

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddExpensePolicies(); // our clean extension method
});

// Register handlers
builder.Services.AddScoped<IAuthorizationHandler, ManagerExpenseHandler>();
builder.Services.AddScoped<IAuthorizationHandler, CFOExpenseHandler>();
```

---

## 7. Consuming Policies in Controllers

In controller/service code, we **authorize against a resource**: the actual expense report.

```csharp
[ApiController]
[Route("api/[controller]")]
public class ExpensesController : ControllerBase
{
    private readonly IAuthorizationService _auth;

    public ExpensesController(IAuthorizationService auth)
    {
        _auth = auth;
    }

    [HttpPost("{id}/approve")]
    public async Task<IActionResult> ApproveExpense(int id)
    {
        // Example expense pulled from DB
        var expense = new ExpenseReport
        {
            Id = id,
            Amount = 1500,
            SubmittedBy = "user123"
        };

        // Evaluate policy against resource
        var result = await _auth.AuthorizeAsync(User, expense, "CanApproveExpenses");

        if (!result.Succeeded)
            return Forbid();

        return Ok($"Expense {id} approved by {User.Identity!.Name}");
    }
}
```

---

## 8. Walkthrough with Example Users

- **User with Manager role**, approving `$800` expense:  
  - Requirement: ≤ $1000.  
  - Handler: `ManagerExpenseHandler` succeeds. ✅  

- **User with Manager role**, approving `$1500` expense:  
  - Requirement fails (> $1000 for manager). ❌  

- **User with CFO role**, approving `$1500` expense:  
  - `CFOExpenseHandler` succeeds since > $1000. ✅  

---

## 9. Diagram of Manager vs. CFO Flow

```plaintext
+-------------------+
|  Expense Report   |  --> amount = $800
+-------------------+                  \
                                         \
                +-----------------------------+
                | Policy: CanApproveExpenses  |
                | Requirement: max 1000       |
                +-----------------------------+
                           |
           +----------------------------------------+
           |                                        |
ManagerExpenseHandler                   CFOExpenseHandler
 (Role=Manager & ≤1000)                 (Role=CFO & >1000)
           |                                        |
           v                                        v
        SUCCESS (Manager)               NOT APPLICABLE (Amount low)
```

For `$1500` expense: Manager handler fails, CFO handler succeeds.

---

## 10. Summary

- **Requirement** = sets approval rule (max limit).  
- **Handler** = enforces rule (Manager ≤ $1000, CFO > $1000).  
- **Policy** = bundles Requirement, applied by name `"CanApproveExpenses"`.  
- **Resource-based authorization** ensures access control can depend on *resource properties* (like `Amount`).  

This approach:  
✅ Clean separation of rules from logic.  
✅ Centralized policy extensions (no messy lambdas).  
✅ Flexible scaling for real-world business needs.  

---

# 🎯 Wrap-Up

We’ve built a complete **end-to-end authorization system**:  
- **Expense Report domain model**  
- **Custom Requirement & Handlers** (Manager vs. CFO logic)  
- **Policy defined via extension methods**  
- **Resource authorization in controller**  
- **Diagram for visualization**  

This demonstrates **how to model complex business approval workflows** cleanly in ASP.NET Core’s authorization framework.