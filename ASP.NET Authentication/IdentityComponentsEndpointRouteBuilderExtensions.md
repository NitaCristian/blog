
`IdentityComponentsEndpointRouteBuilderExtensions.cs` is a file containing a static class with the same name used for the definition of additional identity endpoints required by several identity components.

Those endpoints are defined in a public static method `MapAdditionalIdentityEndpoints` which represents an extension method for an `IEndpointRouteBuilder`, which in our case is the `app` used in `Program.cs` after we build our `WebApplication`.
```csharp
var app = builder.Build();
// ...
app.MapAdditionalIdentityEndpoints();  
app.Run();
```

An alternative to this would've been to define them in the `Program.cs` file with `app.MapPost("/route", () => {});`, but it would've only cluttered the file.

## What do they do?
Well, there are 4 endpoints in total, 2 for `/Account` and 2 for `Manage`.
## Why are they even defined?
Those endpoints are actions that can be done in our app but aren't standalone components.

For example `/Account/Logout`. There is not Logout component, just the action that can be taken when requesting that URL.
Same as `/Account/PerformExternalLogin`, which handles logging in a user with an external application such as Facebook or Google. There can't be a separate component, we just need to define an action that can be taken when a request is made to that endpoint.

They way they are set up is clever and interesting.
First it sets a group called `Account` and the endpoints in it.

Then upon that same group, it creates another group called `Manage` with additional requirements such as authorization. The final endpoints defined in the `Manage` group will have the URL like this
`/Account/Manage/...`.

The actions defined here are `LinkExternalLogin` and `DownloadPersonalData`. 
The last method simply returns a file that contains all personal data.
The purpose of the first method however is to initiate the process of linking an external login provider (like Google, Facebook, or Microsoft) to the current user's account.


### **What is `QueryString.Create`?**

`QueryString.Create` is an ASP.NET Core utility method for constructing query strings in a **type-safe way**. It takes key-value pairs and formats them into a properly encoded query string.

```csharp
QueryString.Create("key", "value")
// Output: ?key=value
```

### **How External Providers Handle Redirects**

1. **External Providers Expect a Callback URL**:
    
    - When initiating an authentication challenge (e.g., via `Google` or `Facebook`), you provide a "redirect URI" where the provider will send the user back after login.
    - Example:
        

1. - `https://yourapp.com/Account/ExternalLogin?Action=LoginCallbackAction`
        
    - The provider doesn’t inspect or care about the query parameters like `Action`. It just redirects the user to the specified URL after authentication.
2. **Your App Handles the Redirect**:
    
    - After the external provider sends the user back, your app uses the redirect URL (and its query parameters) to determine **what happens next**. For instance:
        - `Action=LoginCallbackAction`: Signals that the user is returning from a sign-in attempt.
        - `Action=LinkLoginCallbackAction`: Signals that the user is linking a new external login provider.

---

### **Why Care About `LoginCallbackAction` or `LinkLoginCallbackAction`?**

While the external provider doesn’t use these components, your application **does care**. These parameters are essential for:

1. **Differentiating Authentication Flows**:
    
    - The app can handle a login (`LoginCallbackAction`) differently from linking an account (`LinkLoginCallbackAction`).
2. **Routing Logic**:
    
    - Your app’s controllers or handlers use these actions to determine how to process the response from the external provider.
    - Example:
        

- - `if (action == ExternalLogin.LoginCallbackAction) {     // Handle login flow } else if (action == ExternalLogins.LinkLoginCallbackAction) {     // Handle account linking }`
        
- **Security and State Management**:
    
    - These query parameters help ensure that the app processes the correct flow and doesn’t mix up the responses.