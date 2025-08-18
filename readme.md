# Workload tools demo

Code for the tool is here: [github.com/spaghettidba/WorkloadTools](https://github.com/spaghettidba/WorkloadTools)

## Helpful Blog posts

- Intro https://spaghettidba.com/2019/02/15/benchmarking-with-workloadtools/

## Setup

1. We need some containers - let's use one of Jess' that has AdventureWorks on

```powershell
docker run -p 2500:1433 --volume shared:/shared:z --name mssql1 --hostname mssql1 --network localnet -d jpomfret7/dbatools1:latest
docker run -p 2600:1433 --volume shared:/shared:z --name mssql2 --hostname mssql2 --network localnet -d jpomfret7/dbatools1:latest
```

1. Set up the [baseline.json](.\demo\baseline.json) to collect the workload

1. Start the collection

```powershell
& ("{0}\workloadtools\sqlworkload.exe" -f $env:ProgramFiles) --File ".\demo\baseline.json"
```

1. Run a workload\some bad queries - it's good to have a few mins so you get perf details

```sql
    SELECT SalesOrderID, ProductId, UnitPrice
    FROM Sales.SalesOrderDetail 
    WHERE UnitPrice > 100;
    go 25

    SELECT soh.CustomerID 
    FROM Sales.SalesOrderDetail sod
    JOIN Sales.SalesOrderHeader soh 
        ON sod.SalesOrderID = soh.SalesOrderID
    WHERE soh.CustomerID = 29825;
    go 25
```

![workloadtools is collecting stuff](images/collecting.png)

1. Ctrl+c to get out

2. run the workload viewer

```powershell
& ("{0}\workloadtools\workloadviewer.exe" -f $env:ProgramFiles) -S mssql1 -D workloadtools -M baseline -U sqladmin -P dbatools.IO
```

![WorkloadViewer - Workload page](images/workload.png)
![WorkloadViewer - Queries page](images/queries.png)

1. Fix the issues - add some indexes

```sql
 CREATE NONCLUSTERED INDEX IX_SalesOrderDetail_UnitPrice ON Sales.SalesOrderDetail(UnitPrice) include (productid);
 CREATE NONCLUSTERED INDEX IX_SalesOrderDetail_SalesOrderID ON Sales.SalesOrderDetail(SalesOrderID);
 CREATE NONCLUSTERED INDEX IX_SalesOrderHeader_CustomerID ON [Sales].[SalesOrderHeader] ([CustomerID])
```

1. Start the [perftest.json](.\demo\perftest.json) collection this time

```powershell
& ("{0}\workloadtools\sqlworkload.exe" -f $env:ProgramFiles) --File ".\demo\perftest.json"
```

1. Run the workload viewer to view `baseline` and `perftest` results

```powershell
& ("{0}\workloadtools\workloadviewer.exe" -f $env:ProgramFiles) -S mssql1 -D workloadtools -M baseline -U sqladmin -P dbatools.IO -T mssql1 -E workloadtools -N perftest -V sqladmin -P dbatools.IO
```
