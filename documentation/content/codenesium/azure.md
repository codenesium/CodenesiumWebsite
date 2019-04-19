+++
title = "Azure"
description = ""
weight = 1
alwaysopen = true
+++

It's very easy to deploy an app to Azure.

You can follow the official documentation with some modification https://docs.microsoft.com/en-us/aspnet/core/tutorials/publish-to-azure-webapp-using-vs?view=aspnetcore-2.2

You can skip to the publish step.

The project you are publishing is the one that ends with .Web

Change the DefaultConnection setting when you set up the SQL Server to be ApplicationDbContext.

Uncheck the "Apply this migrations on publish" checkbox. The migrations in a Codenesium app are enabled with the MigrateDatabase flag in the app settings
and are enabled by default.

When the webpage for your app opens you will get a 404 error. You need to add /swagger to the end to see your endpoint.

That's it!