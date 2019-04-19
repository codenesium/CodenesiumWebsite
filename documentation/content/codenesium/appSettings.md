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
  "ExternalBaseUrl": "http://localhost:8000", -> used to return headers with the correct url when hosted with a proxy
  "MigrateDatabase": false, -> migrate the database with entity framework 
  "SecurityEnabled": false, -> secure all controllers with bearer authentication
  "JwtSigningKey": "", -> used with bearer auth
  "JwtIssuer": "", -> used with bearer auth
  "JwtAudience": "", -> used with bearer auth
  "ConnectionStrings": {
    "ApplicationDbContext": "" -> connection string for the application
  },

  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  }
}
```