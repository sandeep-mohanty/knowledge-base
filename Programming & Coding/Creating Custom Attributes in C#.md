# Creating Custom Attributes in C#

**Attributes** provide a way to add metadata to your code. In this blog post we'll cover the basics of what attributes are, how to set attribute properties, and how to configure where attributes can be applied. Finally, we'll dive into a practical example to demonstrate how custom attributes can be used in the applications.

[](#what-is-an-attribute)

## What is an Attribute?

**Attributes** in C# are a way to add declarative information to your code. They provide metadata that can be used to control various aspects of the behavior of your program at runtime or compile-time.

Attributes can be applied to various program elements like:

- assemblies, modules, classes, interfaces, structs, enums
- methods, constructors, delegates, events
- properties, fields, parameters, generic parameters, return values
- or all of the mentioned elements

You can apply one or multiple attributes to these program elements.

You're using attributes every day when creating applications in NET. For example, the following attributes you have seen and used a lot:

```csharp
[Serializable]
public class User { }

[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase { }
```

[](#how-to-create-a-custom-attribute)

## How To Create a Custom Attribute

Attribute in C# is a class that inherits from the base `Attribute` class.

Let's have a look at how to create a custom attribute:

```csharp
[AttributeUsage(AttributeTargets.Class, Inherited = false)]
public class CustomAttribute : Attribute { }

[Custom]
public class MyClass { }
```

The name of the attribute class should have an `Attribute` suffix. When applied to any element, this suffix is omitted.

When creating an attribute class, you're using a built-in attribute to specify where the attribute can be applied:

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = true)]
public class CustomAttribute : Attribute { }
```

You can combine multiple targets by "|" operator to set where an attribute can be applied.

`Inherited` parameter indicates whether the custom attribute can be inherited by derived classes. The default value is `true`.

```csharp
[AttributeUsage(AttributeTargets.Class, Inherited = true)]
public class CustomInheritedAttribute : Attribute { }

[AttributeUsage(AttributeTargets.Class, Inherited = false)]
public class CustomNonInheritedAttribute : Attribute { }
```

When applying **inherited** attribute to the `ParentA` class, it is inherited by a `ChildA` class:

```csharp
[CustomInherited]
public class ParentA { }

public class ChildA { }
```

Here, the `ChildA` class has a `CustomInherited` attribute.

While in the following example, when using a **non-inherited** attribute for the `ParentB`, it is not applied to a `ChildB` class:

```csharp
[CustomNonInherited]
public class ParentB { }

public class ChildB { }
```

[](#attribute-properties)

## Attribute Properties

Attribute properties can be either required (mandatory parameters) or non-required (optional parameters). Attributes can accept arguments in the same way as methods and properties.

```csharp
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, Inherited = false)]
public class CustomWithParametersAttribute : Attribute
{
    public string RequiredProperty { get; }
    public int OptionalProperty { get; set; }

    public CustomWithParametersAttribute(string requiredProperty)
    {
        RequiredProperty = requiredProperty;
    }
}
```

The required properties should be defined as attribute constructor parameters. While non-required should not be defined in the constructor.

When applying such an attribute to a class, you need to set attribute values in the order they are defined in the constructor. Optional properties should be specified by their name:

```csharp
[CustomWithParameters("some text here", OptionalProperty = 5)]
public class ExampleClass { }
```

[](#practical-example-of-using-attributes)

## Practical Example of Using Attributes

Let's create a custom attribute to specify the roles allowed to access a controller method:

```csharp
[AttributeUsage(AttributeTargets.Method, Inherited = false)]
public class AuthorizeRolesAttribute : Attribute
{
    public string[] Roles { get; }

    public AuthorizeRolesAttribute(params string[] roles)
    {
        Roles = roles;
    }
}
```

Next, we apply the `AuthorizeRolesAttribute` to methods in a class, specifying the roles that are allowed to access each method:

```csharp
public class AccountController
{
    [AuthorizeRoles("Admin", "Manager")]
    public void AdminOnlyAction()
    {
        Console.WriteLine("Admin or Manager can access this method.");
    }

    [AuthorizeRoles("User")]
    public void UserOnlyAction()
    {
        Console.WriteLine("Only users can access this method.");
    }

    public void PublicAction()
    {
        Console.WriteLine("Everyone can access this method.");
    }
}
```

To make use of this attribute, we can use reflection to get info about what attributes are applied to a method and what properties they have:

```csharp
public class RoleBasedAccessControl
{
    public void ExecuteAction(object controller, string methodName, string userRole)
    {
        var method = controller.GetType().GetMethod(methodName);
        var attribute = method?.GetCustomAttribute<AuthorizeRolesAttribute>();

        if (attribute is null || attribute.Roles.Contains(userRole))
        {
            method?.Invoke(controller, null);
        }
        else
        {
            Console.WriteLine("Access denied. User does not have the required role.");
        }
    }
}
```

Here we get a method by name from a controller class and look if `AuthorizeRolesAttribute` attribute is applied to the method. If applied, we check if a user can have access to the given method.

Finally, we can test the role-based access control logic with different user roles:

```csharp
var controller = new AccountController();
var accessControl = new RoleBasedAccessControl();

Console.WriteLine("Testing with Admin role:");
accessControl.ExecuteAction(controller, nameof(AccountController.AdminOnlyAction), "Admin");

Console.WriteLine("\nTesting with User role:");
accessControl.ExecuteAction(controller, nameof(AccountController.UserOnlyAction), "User");

Console.WriteLine("\nTesting with Guest role:");
accessControl.ExecuteAction(controller, nameof(AccountController.AdminOnlyAction), "Guest");

Console.WriteLine("\nTesting public method with Guest role:");
accessControl.ExecuteAction(controller, nameof(AccountController.PublicAction), "Guest");
```

Output:

```
Testing with Admin role:
Admin or Manager can access this method.
Testing with User role:
Only users can access this method.
Testing with Guest role:
Access denied. User does not have the required role.
Testing public method with Guest role:
Everyone can access this method.
```

In this example, the `AuthorizeRolesAttribute` is used to specify the roles allowed to access each method in the `AccountController`. The `RoleBasedAccessControl` class enforces these restrictions by checking the user's role against the roles defined in the attribute. This demonstrates how custom attributes can be leveraged in a practical and useful way in a real-world scenario.
