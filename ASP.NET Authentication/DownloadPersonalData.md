This simple method is called whenever a user wants to download all their personal data.
It accesses a user personal data and saves it into a dictionary.
This dictionary is serialized to bytes and returned as an http json file request.


This method `DownloadPersonalData` is designed to allow a user to download their personal data in JSON format.

#### **1. Fetch the User Data**
```csharp
var user = await userManager.GetUserAsync(context.User);
```

- The `UserManager<ApplicationUser>` is used to fetch the current authenticated user based on the `HttpContext`'s `User` property.
- If the user is not found, a `404 Not Found` response is returned
#### **2. Log the User's Request**

```csharp
var downloadLogger = loggerFactory.CreateLogger("DownloadPersonalData");
downloadLogger.LogInformation("User with ID '{UserId}' asked for their personal data.", userId);
```

#### **3. Identify Properties Marked for Personal Data**

```csharp
var personalDataProps = typeof(ApplicationUser).GetProperties().Where(
    prop => Attribute.IsDefined(prop, typeof(PersonalDataAttribute)));
```

- This step dynamically inspects the `ApplicationUser` class (a user model typically derived from `IdentityUser`) and finds all properties marked with the `[PersonalData]` attribute.
- The `PersonalDataAttribute` is a marker used to designate which user data should be included in a "data download" feature.

#### **4. Collect Personal Data**
```csharp
var personalData = personalDataProps.ToDictionary(p => p.Name, p => p.GetValue(user)?.ToString() ?? "null");
```


#### **5. Include External Logins and Authenticator Key**


```csharp
var logins = await userManager.GetLoginsAsync(user);
foreach (var l in logins)
{
    personalData.Add($"{l.LoginProvider} external login provider key", l.ProviderKey);
}

personalData.Add("Authenticator Key", (await userManager.GetAuthenticatorKeyAsync(user))!);
```
- If the user has any external logins (e.g., Google or Facebook), their external login provider and provider key are added to the data.
- The `Authenticator Key`, used for two-factor authentication, is also included

#### **6. Serialize Data and Prepare for Download**
- The collected data dictionary is serialized into JSON format as a byte array using `JsonSerializer.SerializeToUtf8Bytes`.
- The `Content-Disposition` header ensures the browser prompts the user to download the file with the name `PersonalData.json`.
- The data is returned as a downloadable file response