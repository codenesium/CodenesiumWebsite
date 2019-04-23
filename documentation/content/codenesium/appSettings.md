+++
title = "AppSettings"
description = ""
weight = 1
alwaysopen = true
+++
There are two appSettings.json files that are generated.

appSettings.Development.json
appSettings.Production.json

By default .NET Core looks at the environment variable ASPNETCORE_ENVIRONMENT to determine what environment it's using. 

If that variable isn't found it will default to production.

```
{
  "DatabaseProvider": "MSSQL", --> Can be MSSQL or POSTGRESQL
  "ExternalBaseUrl": "http://localhost:8000", --> Th base URL for the API if it's behind a proxy
  "MigrateDatabase": false, --> Migrate the database when the service starts
  "DebugSendAuthEmailsToClient": true, --> Display auth links that would normally be sent by email in the UI for easy testing
  "JwtSettings": {
    "SigningKey": "3dcf63249bacf399226e6e7fc580379420d75f7917b897f29a54984c94c81dbd", --> The password used to sign bearer tokens
    "Issuer": "https://www.codenesium.com", --> the bearer token issuer
    "Audience": "https://www.codenesium.com" --> the bearer token audience
  },
  "StripeSettings": { --> Stripe settings you can use in your app. Not currently implemented.
    "SecretKey": "",
    "PublishableKey": ""
  },
  "ConnectionStrings": { --> database connection strings
    "ApplicationDbContext": "Server=localhost;Persist Security Info=False;User ID=test;Password=Passw0rd;Initial Catalog=IOT;MultipleActiveResultSets=True;Connection Timeout=10;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "System": "Debug",
      "Microsoft": "Information"
    },
    "Console": {
      "IncludeScopes": "true"
    },
    "Debug": {
      "IncludeScopes": "true"
    }
  }
}
```