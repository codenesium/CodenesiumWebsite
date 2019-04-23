+++
title = "Health Checks"
description = ""
weight = 1
alwaysopen = true
+++
Generated APIs have a health check endpoint that by default checks that the database is accessible and that we can query it.

The path is api/health.

Health checks are enabled in Startup.cs.

Health checks can check databases and also that other services are up. 

[Health Checks[(https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-2.2)
