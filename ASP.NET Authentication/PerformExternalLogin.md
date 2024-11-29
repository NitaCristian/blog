# `PerformExternalLogin`

The method triggers the external login flow (e.g., Google or Facebook login) by preparing necessary parameters and redirecting the user to the external provider's login page.
### 1. Gather Query Parameters

```csharp
IEnumerable<KeyValuePair<string, StringValues>> query =
    [new("ReturnUrl", returnUrl), new("Action", ExternalLogin.LoginCallbackAction)];
```

Prepares query parameters to include with the redirection:
- `ReturnUrl`: The URL the user should be returned to after the login process is completed.
- `Action`: Specifies the callback action to handle the login process (`LoginCallbackAction`).
### 2. Build Redirect URL
```csharp
var redirectUrl = UriHelper.BuildRelative(context.Request.PathBase, "/Account/ExternalLogin", QueryString.Create(query));
```
Constructs the URL to which the external provider should redirect the user after authentication:
- `/Account/ExternalLogin`: The endpoint in your app to process the callback from the external provider.
- Includes the query parameters (`ReturnUrl` and `Action`).
### 3. Configure Authentication Properties
```csharp
var properties = signInManager.ConfigureExternalAuthenticationProperties(provider, redirectUrl);
```
Sets up the properties needed to initiate the external login:
- `provider`: Specifies the external login provider (e.g., Google, Facebook).
- `redirectUrl`: Tells the external provider where to send the user back after successful login.
### 4. Return Challenge
Issues an authentication challenge for the specified provider:
- Redirects the user to the provider’s login page.
- The challenge includes the configured `properties` to ensure the flow is linked to your app’s callback.
### What is an "Authentication Challenge"?
An **authentication challenge** is a way to initiate the process of authenticating a user. When your application sends an authentication challenge, it's essentially asking an external system (like Google or Facebook) or a built-in authentication system to verify the user's identity.

#### Examples:
- **External Authentication**: Redirecting a user to a Google sign-in page is an authentication challenge.
- **Local Authentication**: Prompting a user to log in with a username and password on your app is a simpler form of authentication challenge.

In the context of ASP.NET Core:
- A **challenge** typically means the user needs to prove their identity via some mechanism (OAuth, cookies, etc.).
- The `ChallengeResult` (or its variants) triggers this process.

### **What are "Redirect URL" and "Return URL"?**

These URLs help manage navigation during and after the authentication process:
#### **Redirect URL**
- This is the **destination URL** where the external authentication provider (e.g., Google) sends the user back after they log in.
#### **Return URL**
- This is the **final URL** the user wants to access after completing the login process.
- Example: A user tries to visit their profile page (`/profile`) but is not logged in. After login, the app should take them back to `/profile`.

#### How They Work Together
1. **Redirect URL**: Specifies where the external provider (e.g., Google) sends the user back.
2. **Return URL**: Specifies where your app redirects the user after handling the external provider's response.

### What Are `[FromServices]` and `[FromForm]`?

These are **parameter binding attributes** in ASP.NET Core. They help the framework understand **where to get values** for parameters in your action methods.

#### `[FromServices]`

- Specifies that the parameter should be resolved from the **Dependency Injection (DI) container**.
- Example:
```csharp
public IActionResult MyAction([FromServices] MyService service) { }
```
- Here, `MyService` is injected from the DI container.
#### `[FromForm]`
- Specifies that the parameter value should come from the **HTTP form data** (e.g., when a user submits a form in a POST request).
- Example:
```csharp
public IActionResult SubmitForm([FromForm] string name) { }
```

- Here, `name` is taken from the submitted form's body.

### **What is `HttpContext`?**

`HttpContext` represents all the details of an individual HTTP request and its associated response in ASP.NET Core. It is essentially a **container** for everything about the current request being handled.

#### **Key Components of `HttpContext`**:

1. **Request**: Information about the incoming HTTP request.
    
    - URL, headers, body, query parameters, etc.
    - Example: `HttpContext.Request.Query["id"]` retrieves the `id` query parameter.
2. **Response**: Information about the HTTP response being sent to the client.
    
    - Example: `HttpContext.Response.StatusCode = 404` sets the status code to `404 Not Found`.
3. **User**: Represents the currently authenticated user (if any).
    
    - Example: `HttpContext.User` gives access to claims and roles.
4. **Session**: Manages user session data.
    
    - Example: `HttpContext.Session.SetString("Key", "Value")`.
5. **Items**: Temporary storage for the lifetime of the request.
    
    - Example: `HttpContext.Items["key"] = "value"`.


### **What is `SignInManager`?**

`SignInManager` is part of ASP.NET Core Identity, a framework for handling authentication and user management in ASP.NET Core applications. It provides high-level APIs for managing sign-in operations.

#### **Key Responsibilities of `SignInManager`**:

1. **User Sign-In**:
    
    - Authenticate a user and create a session or authentication cookie.
    - Example: `SignInManager.PasswordSignInAsync()` validates a username/password and signs in the user.
2. **External Authentication**:
    
    - Works with external providers like Google, Facebook, or Microsoft.
    - Example: `SignInManager.ConfigureExternalAuthenticationProperties()` prepares the properties for external login.
3. **Sign-Out**:
    
    - Invalidates the current user session or authentication cookie.
    - Example: `SignInManager.SignOutAsync()`.
4. **Two-Factor Authentication (2FA)**:
    
    - Supports 2FA with tokens or SMS.

#### How Does It Work?

- `SignInManager` interacts with other components like `UserManager` (to manage user accounts) and ASP.NET Core's authentication middleware to handle sign-in and authentication workflows.