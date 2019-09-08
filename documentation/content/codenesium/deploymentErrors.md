+++
title = "Deployment Errors"
description = ""
weight = 1
alwaysopen = true
+++

There are lots of reasons a project may not generate or deploy. This is a list of the most common issues. We're working to improve feedback
to help you determine what the issue is. You can always sync the code after the generation step completes but the playground
server won't be updated unless our system can build and test the API and React applications. 


##### Primary keys not defined as auto-incrementing integers.
For Codenesium to work you have to have a non-composite auto-incrementing primary key.

##### Reference tables that don't have an auto-incrementing integer primary key.
We require an auto-incrementing primary key on reference tables. This does smell a bit but it's vastly more complicated
to generate code without the primary key. It's hard to reason about reference tables in a REST API in general.

##### Tables missing primary keys.
Add a primary key.

##### Issues with our infrastructure.
Our infrastructure has a lot of moving pieces and they're not allways working. We try our best to keep everything chugging along.
Ususally if there is a problem with the infrastructure a deploy will fail immediately vs in the generation step. 
