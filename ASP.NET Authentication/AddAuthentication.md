The `AddAuthentication()` method in ASP.NET Core is used to set up and configure the authentication system for your application. It’s a critical part of how ASP.NET Core handles user identity and access control.

## **What is Authentication?**

Authentication is the process of identifying who a user is. In ASP.NET Core, this involves verifying the user's credentials (like a username and password, a JWT token, or an OAuth token). Once verified, the user is represented in the app as a `ClaimsPrincipal`, which contains claims about the user (e.g., their roles, permissions, or identity).

## **What Does `AddAuthentication()` Do?**

The `AddAuthentication()` method configures how the application will handle authentication. It defines:

1. **Default Authentication Scheme**: Which strategy (scheme) to use by default for authenticating users.
2. **Default Challenge Scheme**: Which strategy to use when an unauthenticated user tries to access a restricted resource.
3. **Default Forbid Scheme**: Which strategy to use when an authenticated user is denied access to a resource.
4. **Default Sign-In/Sign-Out Scheme**: Which strategy to use for signing in or out users.

These defaults ensure that the framework knows how to handle authentication without requiring explicit configuration for every scenario.

### **Key Authentication Concepts**

1. **Authentication Scheme**:
    
    - A "name" for an authentication mechanism (e.g., cookies, JWT, OAuth).
    - Schemes tell the app which system to use for authentication in different scenarios.
    - Examples:
        - `IdentityConstants.ApplicationScheme`: Used for cookie-based authentication in ASP.NET Identity.
        - `Bearer`: Used for JWT authentication.
2. **Challenge**:
    
    - Happens when an unauthenticated user tries to access a protected resource.
    - Example: Redirecting to a login page for cookie-based authentication, or returning a 401 response for APIs using JWT.
3. **Authentication Handlers**:
    
    - Implement the actual authentication logic for a scheme.
    - Examples: `CookieAuthenticationHandler` for cookies, `JwtBearerHandler` for JWTs
```csharp
builder.Services.AddAuthentication(options =>
{
    // Default schemes
    options.DefaultAuthenticateScheme = IdentityConstants.ApplicationScheme; // Cookie-based auth
    options.DefaultChallengeScheme = IdentityConstants.ApplicationScheme;   // Redirect to login on challenge
    options.DefaultScheme = IdentityConstants.ApplicationScheme;            // Default scheme for auth

    // Alternatively, if using JWT:
    // options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    // options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddCookie(IdentityConstants.ApplicationScheme, options =>
{
    options.LoginPath = "/Account/Login";   // Path to redirect unauthenticated users
    options.AccessDeniedPath = "/Account/AccessDenied"; // Path for access denied
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = "YourIssuer",
        ValidAudience = "YourAudience",
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YourSecretKey"))
    };
});
```