+++
title = "Database Migrations"
description = ""
weight = 1
alwaysopen = true
+++

I think database migrations can be one of the most frustrating things to deal with. There are several way to handle migrations. My preferred
way is to use migration scripts with [Fluent Migrator](https://github.com/fluentmigrator/fluentmigrator). I just really like having data and schema scripts in the same script so I can handle any situation. There are lots of options though and you should pick the one that works for you.

Our generated APIs start with a single entity framework migration in the data access project. This is just the create database script from
the database you submitted. 

We do not recommend using this script in a production project. It's really just to get you going if you want to run the api against a database and you don't have your migration strategy worked out. .NET core has excellent migration [management tools](https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-2.1) built in. There are also other options
like [Fluent Migrator](https://github.com/fluentmigrator/fluentmigrator) and also Microsoft's [DACPAC](https://msdn.microsoft.com/en-us/library/jj860455%28v=vs.103%29.aspx?f=255&MSPPError=-2147217396) system.  

There is an option in the appSettings.json called MigrateDatabase. If you set this to true when the app starts it will apply any migrations to your database. 