# Avoid These 3 Entity Design Mistakes in .NET

Refactor bloated C# entities with Domain Services and Policy Objects for clean, testable .NET domain models. Explore real-world DDD examples to simplify your architecture and boost maintainability.

## The Problem with Bloated Entities
You’ve built a Loan entity that started simple—until credit checks and loan history rules turned it into a bloated, untestable mess. This creates maintenance nightmares in .NET projects. Discover how Domain Services and Policy Objects streamline your C# domain models for testability and scalability.

## Why Entities Should Stay Simple
Bloated entities hurt your codebase by:

- **Reducing testability:** You need mocks for external services.
- **Increasing coupling:** Entities depend on unrelated aggregates.
- **Breaking DDD principles:** Entities should only enforce their own invariants.

⚠️ Gotcha: Mixing external logic in entities makes unit testing a nightmare — avoid it by isolating rules in services or policies.

## The 3 Entity Design Mistakes to Avoid
- **Mixing External Logic:** Entities calling external services (e.g., credit checks) increase coupling.
- **Overloading Invariants:** Entities handling cross-aggregate rules violate DDD principles.
- **Neglecting Testability:** Bloated entities require complex mocks, making tests brittle.

## Real-World Example: The Loan Entity
```csharp
public class Loan
{
    public decimal Amount { get; private set; }
    public decimal InterestRate { get; private set; }
    public DateTime DueDate { get; private set; }

    public void Approve(decimal requestedAmount, Customer customer)
    {
        if (requestedAmount <= 0)
            throw new InvalidOperationException("Amount must be greater than zero");
        if (!customer.HasGoodCreditScore())
            throw new InvalidOperationException("Customer credit score too low");
        if (customer.HasOutstandingLoans())
            throw new InvalidOperationException("Customer has outstanding loans");
        Amount = requestedAmount;
    }
}
```

## Fixing It with Domain Services

### What Is a Domain Service?
A Domain Service is a stateless class that coordinates logic spanning multiple entities.

### Refactoring the Loan Example
```csharp
public interface ILoanApprovalService
{
    bool CanApproveLoan(Customer customer, decimal requestedAmount);
}
public class LoanApprovalService : ILoanApprovalService
{
    private readonly ICreditService _creditService;
    public LoanApprovalService(ICreditService creditService)
    {
        _creditService = creditService;
    }
    public bool CanApproveLoan(Customer customer, decimal requestedAmount)
    {
        // Ensure amount is valid
        if (requestedAmount <= 0) return false;
        // Check external credit service
        if (!_creditService.HasGoodCredit(customer)) return false;
        // Validate customer loan history
        if (customer.HasOutstandingLoans()) return false;
        return true;
    }
}
```

And the entity becomes simple:
```csharp
public class Loan
{
    public decimal Amount { get; private set; }
    public bool IsApproved { get; private set; }
    
    internal void Approve(decimal requestedAmount)
    {
        // Only mutate internal state, keeping entity pure
        Amount = requestedAmount;
        IsApproved = true;
    }
}
```

👏 Clap if this Domain Service approach just saved you from another bloated entity!

## How Loan Uses LoanApprovalService (via Application Layer)
```csharp
public class LoanApplicationService
{
    private readonly ILoanApprovalService _approvalService;
    
    public LoanApplicationService(ILoanApprovalService approvalService)
    {
        _approvalService = approvalService;
    }
    public Loan RequestLoan(Customer customer, decimal requestedAmount)
    {
        if (!_approvalService.CanApproveLoan(customer, requestedAmount))
            throw new InvalidOperationException("Loan cannot be approved.");
        var loan = new Loan();
        loan.Approve(requestedAmount);
        return loan;
    }
}
```

## Levelling Up with Policy Objects

### When to Use Policies
Policy Objects are best when rules are many, composable, and likely to change.

### Building Policy Objects
```csharp
public interface ILoanApprovalPolicy
{
    bool IsSatisfiedBy(Customer customer, decimal requestedAmount);
}

public class PositiveAmountPolicy : ILoanApprovalPolicy
{
    public bool IsSatisfiedBy(Customer customer, decimal requestedAmount) 
        => requestedAmount > 0;
}
public class GoodCreditPolicy : ILoanApprovalPolicy
{
    private readonly ICreditService _creditService;
    public GoodCreditPolicy(ICreditService creditService) => _creditService = creditService;
    public bool IsSatisfiedBy(Customer customer, decimal requestedAmount) 
        => _creditService.HasGoodCredit(customer);
}
public class NoOutstandingLoansPolicy : ILoanApprovalPolicy
{
    public bool IsSatisfiedBy(Customer customer, decimal requestedAmount) 
        => !customer.HasOutstandingLoans();
}
```

### Combining Policies with Composition
```csharp
public class CompositeLoanApprovalPolicy : ILoanApprovalPolicy
{
    private readonly IEnumerable<ILoanApprovalPolicy> _policies;
    
    public CompositeLoanApprovalPolicy(IEnumerable<ILoanApprovalPolicy> policies)
    {
        _policies = policies;
    }
    
    public bool IsSatisfiedBy(Customer customer, decimal requestedAmount) =>
        _policies.All(p => p.IsSatisfiedBy(customer, requestedAmount));
}
```

Usage:
```csharp
var policies = new List<ILoanApprovalPolicy>
{
    new PositiveAmountPolicy(),
    new GoodCreditPolicy(creditService),
    new NoOutstandingLoansPolicy()
};

var approvalPolicy = new CompositeLoanApprovalPolicy(policies);
if (approvalPolicy.IsSatisfiedBy(customer, requestedAmount))
{
    loan.Approve(requestedAmount);
}
```

💡 Pro Tip: Use Policy Objects when rules change frequently, as they’re easier to swap or extend without modifying core logic.

## Domain Services vs. Policy Objects

| Aspect | Domain Service | Policy Object |
|---|---|---|
| Scope | Coordinates multiple entities or services | Encapsulates a single rule |
| Granularity | Coarse-grained | Fine-grained |
| Reusability | Medium | High (plug in/out rules easily) |
| Complexity | Centralized but can grow large | More classes, but highly modular |

## Practical Tools and Next Steps

### Libraries to Simplify DDD
- **Scrutor** — Auto-register services and policies via assembly scanning.
- **Moq** — Test services in isolation.
- **FluentAssertions** — Assert policies with readable specs.
- **Microsoft’s DDD documentation** — Great background reading.

## Key Takeaways for .NET Developers
- Entities should only protect their invariants.
- Cross-aggregate or external rules → move to Domain Services.
- Fine-grained, evolving rules → model as Policy Objects.
- The application layer orchestrates the workflow.
- Result = clean, testable, extensible code.

💬 Discussion Prompt: What’s your go-to strategy for keeping .NET entities clean?

Do you lean on Domain Services, Policy Objects, or another approach? Share in the comments!

### Follow Me on LinkedIn for C# and .NET insights!

## Keywords
- Clean code in .NET
- Testable .NET architecture
- Domain-Driven Design (DDD)
- Software architecture
