+++
title = "Database Migrations"
description = ""
weight = 1
alwaysopen = true
+++

There are several ways to handle database migrations in a .NET Core project. Codenesium APIs are set up to use the built in Entity Framework migrations and the migrations are configured to run when the service starts. This can be disabled in the appSettings.json.

Our generated APIs start with two entity framework migrations in the data access project. The first migration is for the database schema you generated.
The second migration is for the Identity tables. By default we generate a migration for SQL Server and PostgreSQL and we check the database provider to determine which migration to run. 

We do not recommend using this script in a production project. It's really just to get you going if you want to run the API against a database and you don't have your migration strategy worked out. .NET core has excellent migration [management tools](https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-2.1) built in. There are also other options
like [Fluent Migrator](https://github.com/fluentmigrator/fluentmigrator) and also Microsoft's [DACPAC](https://msdn.microsoft.com/en-us/library/jj860455%28v=vs.103%29.aspx?f=255&MSPPError=-2147217396) system.  

There is an option in the appSettings.json called MigrateDatabase. If you set this to true when the app starts it will apply any migrations to your database. 