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

You will need a way to generate tokens. This could be a static token you generate with a long expiration time or it could be by using [Identity Server](http://docs.identityserver.io/en/release/)


Security can be enabled/disabled by setting the SecurityEnabled setting in appSettings.json.

In ConfigureServices we add the authorize attribute to all controllers
```
services.AddMvcCore(config =>
{
	if(this.Configuration.GetValue<bool>("SecurityEnabled"))
	{
		 var policy = new AuthorizationPolicyBuilder()
					  .RequireAuthenticatedUser()
					  .Build();
		 config.Filters.Add(new AuthorizeFilter(policy));
	 }
	 config.Filters.Add(new BenchmarkAttribute());
	 config.Filters.Add(new NullModelValidaterAttribute());
	 
});
```

Then we configure the bearer token auth
```if(this.Configuration.GetValue<bool>("SecurityEnabled"))
{
	var key = Encoding.UTF8.GetBytes(this.Configuration["JwtSigningKey"]);
	services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
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
			ValidAudience = this.Configuration["JwtAudience"],
			ValidIssuer = this.Configuration["JwtIssuer"],
			IssuerSigningKey = new SymmetricSecurityKey(key)
		};
	});
}
```

Then in Configure we enable authentication.
```
if(this.Configuration.GetValue<bool>("SecurityEnabled"))
{
	 app.UseAuthentication();
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


