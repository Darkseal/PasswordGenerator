# PasswordGenerator
A simple C# helper class for ASP.NET Core to generate a random password with custom strength requirements: min length, uppercase, lowercase, digits &amp; more

# Introduction
Some time ago I had to implement a C# method that creates a random generated password in C#. Before committing into it I spent some minutes surfing the web, trying to find something I could use. I stumbled upon [this 2006 post from Mads Kristensen](https://madskristensen.net/post/generate-random-password-in-c), which is a guy I seriously love for all the great work he did with some incredibly useful Visual Studio extensions such as [Web Essentials](http://vswebessentials.com/), [Web Compiler](https://www.nuget.org/packages/BuildWebCompiler/), [ASP.NET Core Web Templates](https://www.nuget.org/packages/MadsKristensen.AspNetCore.Web.Templates/) - and [a bunch of other great stuff](https://www.nuget.org/profiles/madskristensen).

However, the function I found in that post didn't help me much, because it had no way to ensure any strong-password requisite other than the minimum required length: more specifically, I need to generate password with at least one uppercase & lowercase letter, digit and non-alphanumeric character - and also a certain amount of unique characters. The random password generated against the Mads function could have them or not, depending on the randomness: that simply won't do in my scenario, since I had to deal with the UserManager.CreateUserAsync(username, password) method of the Microsoft.AspNetCore.Identity namespace, which utterly crashes whenever the password isn't strong enough.

Eventually, I ended up coding my own helper class - just like Mads Kristensen more than 11 years ago.

# Usage
As you can see by looking at the source code, the class takes a `PasswordOptions` object as parameter, which is shipped by the `Microsoft.AspNetCore.Identity` assembly, but you can easily replace it with a two int - four bool parameter group or POCO class if you don't have that package installed. In the likely case you have it in your ASP.NET Core project, you can use the exact same object used in the `ConfigureService` method of the `Startup` class when defining the password requirements:

```csharp
// Add ASP.NET Identity support
services.AddIdentity<ApplicationUser, IdentityRole>(
    opts =>
{
    opts.Password.RequireDigit = true;
    opts.Password.RequireLowercase = true;
    opts.Password.RequireUppercase = true;
    opts.Password.RequireNonAlphanumeric = false;
    opts.Password.RequiredLength = 8;
})
.AddEntityFrameworkStores<ApplicationDbContext>();
```

**UPDATE**: as of July 2018, the `PasswordOptions` native support has been removed to avoid the required dependency to the `Microsoft.AspNetCore.Identity` class: now the class has standard parameters (two *int*, four *boolean*) having the same name of the corresponding `PasswordOptions` properties.

That's it for now: hope you'll like it!

## Security considerations
The password randomness is calculated using a `CryptoRandom` class that mimics the standard Random class in the .NET Framework, replacing its standard (non-secure) behaviour with a cryptographic random number generator. The `CryptoRandom` class has been taken from [IdentityModel](https://github.com/IdentityModel/IdentityModel/).

In the GitHub project you'll also find an alternative `CryptoRandom` implementation (`CryptoRandom2`), which has been taken from [here](https://gist.github.com/niik/1017834) (credits to Stephen Toub, Shawn Farkas and Markus Olsson). If you want to use the alternative implementation, just rename the `CryptoRandom2.cs` file and class to `CryptoRandom`, replacing the previous one - or just instantiate a `CryptoRandom2` object in the `PasswordGenerator.cs` file.

## Online Resources
* Official Site: https://www.ryadel.com/
* Class explanation and usage samples: https://www.ryadel.com/en/c-sharp-random-password-generator-asp-net-core-mvc/

## Additional References
If you need a C# helper function to check for strong passwords, don't forget to also [read this post](https://www.ryadel.com/en/passwordcheck-c-sharp-password-class-calculate-password-strength-policy-aspnet/).
