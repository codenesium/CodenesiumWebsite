+++
title = "Authentication"
description = ""
weight = 1
alwaysopen = true
+++

[.NET Core Authentication MSDN](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-2.1)


The summary of how we do authentication is we use bearer tokens as JWTs.

Authentication is complicated. There are so many options and ways it can be done. For a generated API we took the easiest and most generic route which is
a bearer token. This makes sense for most APIs and web applications. 

The generated APIs contain a simple auth solution for users to register and manage their accounts. You don't have to use this solution.

You mau want to look at [Identity Server](http://docs.identityserver.io/en/release/) for your suth solution.



In ConfigureServices we add the authorize attribute to all controllers
```
services.AddMvcCore(config =>
{
	 var policy = new AuthorizationPolicyBuilder()
				.RequireAuthenticatedUser()
				.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme)
				.Build();

	 // add a policy requiring a an authenticated user to hit any controller
	 config.Filters.Add(new AuthorizeFilter(policy));

	 // add a total request time item to the response header collection
	 config.Filters.Add(new BenchmarkAttribute());

	 // reject requests with a null model
	 config.Filters.Add(new NullModelValidaterAttribute());
}).AddVersionedApiExplorer(
o =>
{
	o.GroupNameFormat = "'v'VVV";

	// note: this option is only necessary when versioning by url segment. the SubstitutionFormat
	// can also be used to control the format of the API version in route templates
	o.SubstituteApiVersionInUrl = true;
});
```

Then we configure Identity and bearer token auth
```
 public virtual void SetupAuthentication(IServiceCollection services)
{
	byte[] key = Encoding.UTF8.GetBytes(this.Configuration["JwtSettings:SigningKey"]);
	if (key.Length <= 16)
	{
		throw new Exception("JWT key mut be longer than 16 characters");
	}

	services.AddIdentity<AuthUser, IdentityRole>()
		.AddEntityFrameworkStores<ApplicationDbContext>()
		.AddDefaultTokenProviders();

	services.Configure<IdentityOptions>(options =>
	{
		// Password settings.
		options.Password.RequiredLength = 8;

		// options.Password.RequireDigit = true;
		// options.Password.RequireLowercase = true;
		// options.Password.RequireNonAlphanumeric = true;
		// options.Password.RequireUppercase = true;
		// options.Password.RequiredUniqueChars = 1;

		// Lockout settings.
		// options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
		// options.Lockout.MaxFailedAccessAttempts = 5;
		// options.Lockout.AllowedForNewUsers = true;
		// options.SignIn.RequireConfirmedEmail = true;

		// User settings.
		options.User.AllowedUserNameCharacters =
		"abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._@+";
		options.User.RequireUniqueEmail = true;
	});

	services.AddAuthentication(cfg =>
	{
		cfg.DefaultScheme = IdentityConstants.ApplicationScheme;
		cfg.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
	})
	.AddJwtBearer(jwtBearerOptions =>
	{
		jwtBearerOptions.TokenValidationParameters = new TokenValidationParameters()
		{
			ClockSkew = TimeSpan.FromMinutes(5),
			ValidateAudience = true,
			ValidateIssuer = true,
			ValidateLifetime = true,
			RequireSignedTokens = true,
			RequireExpirationTime = true,
			ValidAudience = this.Configuration["JwtSettings:Audience"],
			ValidIssuer = this.Configuration["JwtSettings:Issuer"],
			IssuerSigningKey = new SymmetricSecurityKey(key)
		};
	});
}
```


A token looks like this 
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1laWQiOiJzdXBwb3J0QGNvZGVuZXNpdW0uY29tIiwicm9sZSI6IlVzZXIiLCJuYmYiOjE1Mjc5NjQ3MTgsImV4cCI6MTUyODU2OTUxOCwiaWF0IjoxNTI3ODc4MzE4LCJpc3MiOiJodHRwczovL3d3dy5jb2RlbmVzaXVtLmNvbSIsImF1ZCI6Imh0dHBzOi8vd3d3LmNvZGVuZXNpdW0uY29tIn0.ITVgY435wLnFF4FlGbZApSUhI31fbpKcEWrjfXa5XE8
```

You can decrypt a token using a website like https://jwt.io/

```
{
  "alg": "HS256",
  "typ": "JWT"
}

{
  "nameid": "support@codenesium.com",
  "role": "User",
  "nbf": 1527964718,
  "exp": 1528569518,
  "iat": 1527878318,
  "iss": "https://www.codenesium.com",
  "aud": "https://www.codenesium.com"
}
```

For more explanation on JSON Web Tokens see https://jwt.io/introduction/ 


