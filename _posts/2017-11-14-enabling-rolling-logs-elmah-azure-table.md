---
layout: post
title: Enabling Rolling Logs with ELMAH Azure Table Storage
author: Chandermani
comments: true
cover: /assets/images/2017-11-14-enabling-rolling-logs-elmah-azure-table.png
tags:
- ELMAH
- azure
- logs
- rolling logs
- library
- table storage
---

ELMAH (Error Logging Modules and Handlers) is an excellent library to track application wide errors in an ASP.Net web app. It also supports several storage sinks including **Azure table** for storing error logs. 

The nuget package [Elmah.AzureTableStorage](https://www.nuget.org/packages/Elmah.AzureTableStorage/) has one such implementation that stores the log in Azure table storage. The implementation creates a table `Elmah` in the configured storage account and pushes logs to the specific storage.

Out team too uses `Elmah.AzureTableStorage` for multiple web applications. But recently we had a problem with the library as the log table kept growing due to lack of a maintenance process that can trim the old errors. 

As I pored over the [source code of Elmah.AzureTableStorage](https://github.com/MisinformedDNA/Elmah.AzureTableStorage) I realized implementing a **rolling log** for ELMAH errors will fix our current problem and work much better in future.

I went ahead and implemented ELMAH rolling logs on top of Azure table storage using `Elmah.AzureTableStorage` as the base implementation. I will not dwell into the implementation as it is not very complex. 

Look at [my fork](https://github.com/chandermani/Elmah.AzureTableStorage) for the implementation details. There is a new class `AzureTableStorageRollingErrorLog` that can be used for rolling logs. The configuration remains similar to the original library except the reference chances to the above class in config file.

```
 <errorLog
type="Elmah.AzureTableStorage.AzureTableStorageRollingErrorLog,     Elmah.AzureTableStorage" 
connectionStringName="ElmahAzureTableStorage" />
```
Once configured correctly, an error will be simultaneously recorded in two table:
- `ElmahYYYYMM (Elmah201711)`
- `ElmahYYYYMM+1 (Elmah201712)`

The use of next month table (`ELMAHYYYYMM+1`) allows us to query last month data from a single table when transitioning into a new month.

With rolling logs in place, past data (tables) can be easily archived!

Another important extension to the above library is the use of compression to store the `AllXml` data. *Azure table row has limit of 1MB*, and I have seen this limit being breached in past when storing logs. With compression enable it will become a bit harder to breach the limit.

If you need rolling logs for ELMAH backed by Azure Table storage, head over to my [github fork](https://github.com/chandermani/Elmah.AzureTableStorage).


