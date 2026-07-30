---
type: Note
_width: wide
---
About the original idea. I will go area by area, talking about my thoughts and ideas that we need to consider:

1. [The.NET](http://The.NET) framework modernization: this is moving [from.NET](http://from.NET) 4.5 to 4.8.1 and updating the dependencies of the libraries. The original estimate is between three and four months. I find this, without seeing the code and how many dependencies there are, a bit too high. I can say that once we get the code and see how we can deploy the whole thing and all the workarounds that they have to deploy the application, it's something that we should have finished in between one and two months. It's similar to the work I did for the migration somewhere into MAUI, which was even more challenging. Now with AI and having the records or the log changes between the versions of their framework, you get to speed really fast.
2. Front-end architecture and UI modernization: this is rewriting the application to a modern framework. We haven't seen how many features it has. We only have one screenshot and we are talking about five to six months. Maybe if we have two people in there, two front-end developers, depending on how big it is, it is in the range between 800 and 2,000 hours. It's okay but we need to have more information.
3. Regarding business logic refactoring, this is basically seeing what's on the SQL Server. They aim to work only on the SQL Server procedures but there should be much more in there that could be a risk for this migration. Move that into the application layer. The problem with that approach is that as there is no testing across the whole application, we cannot guarantee that we will not break anything. What is now on the business layer behaves as expected and it's a feature parity with what was on the stored procedures or the database code. It can be very dark. It needs to be removed from there but as we haven't seen any of the database structure, tables, stored procedures, functions, linked servers, views, or triggers, it could be anything there. That could be quite challenging. Their estimation is between 10 and 14 months for a single developer. It could be okay again depending on what we have in the database.
4. Regarding the infrastructure modernization, this is migrating from a SQL Server to an Azure SQL Database server. There are some problems there that they may not have considered. For example linked servers or Azure shops or SQL shops are not supported on Azure SQL Databases and there are some changes that you have to do to support them. There's a tool from Microsoft that allows us to see if it's viable or not to migrate the database but we must run it on a live connection so we have to get a database to see if it's feasible or not.
5. That's it. The 700-800 hours, which is between four and five months, looks way too high. Imagine that the easiest approach would be to have a back-up and to restore that back-up into Azure, which would take maybe a week. Backups are not an option because it requires you to have downtime but perhaps they are open to that idea.
6. The source management, the source code management point, consists of migrating from DFS to Azure DevOps and it was estimated between one and two months of work. To me if, assuming that they don't need to carry over the work that they may have done in terms of tickets and management things they may have on DFS and they want just to have the repos in Azure, that would be really simple. It would be a matter of a couple of days to have it on a repo in Azure, ready to start working on day zero.
7. CI/CD could be challenging and I think that this, once you do months, is realistic depending on the tech stack that we have to build and deploy and the strategy that we have to. For example if we were talking about the net core application that can be Dockerized and we deploy to a natural container registry, be running on container apps, for sure this is way too high. Since we may have to have a virtual machine to run the build on the old network version, see it, maybe upload to NSF TP, download it on a machine, replace the files into the IIS, and configure users with permissions enough to perform that action, that may be accurate. If we were on a much more stack, for sure this is way too high.

Up to this point, if we follow all these steps, we will end up with:

- a newer version of the.NET framework and application service layer
- the Azure databases migrated to the cloud with the backups policy and recovery strategies all configured on that
- a new React application

I don't see much value proposition in this because.NET will reach its end of life. There will be new needs. Keeping this monolith as it is, it will not be prepared as we are just migrating the business logic that was on the database into the application area. This will gain us new sorts of challenges, such as:

- unit-of-work dependency injection
- context management
- the usage of an ORM not just to execute the SQL but to prevent SQL injection
- the dependency between the upcoming migrations and changes made to the source code

It's not a clear win in this whole process rather than security issues and having a more modern stack.
