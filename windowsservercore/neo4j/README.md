### neo4j on Windows Servercore Container base image

### Usage

```docker run -d --name neo1 --hostname neo1 -p 7474:7474 -p 7687:7687 -v C:\docker_volume\neo4j_data:C:\neo4j\data --restart=always coderobin/neo4j:10.0.14393.2312-3.4.5```

Adjust parameters per your needs.

### Caution

For version 2.1.8 and 2.3.12, neo4j running as a Windows Service has fatal fault for any Production use, __DO NOT__ use for production.

#### Clean shutdown problem

neo4j is a Java process, in order to run as a Windows Service, it has another Wrapper which was registered as a Windows Service, but when you do normal operations on Windows e.g. ```Stop-Service Neo4j-Server``` or ```Restart-Service Neo4j-Server```, although the Wrapper could get the Stop signal, it doesn't communicate that to the neo4j Java process, therefore the database engine has no chance to run a shutdown process as shown below:

##### Clean Shutdown logs by neo4j v2.3.12

```
2018-08-15 10:19:21.748+0000 INFO  [o.n.k.i.f.GraphDatabaseFacade] Shutdown started
2018-08-15 10:19:21.749+0000 INFO  [o.n.k.i.f.CommunityFacadeFactory] Database is now unavailable
2018-08-15 10:19:21.808+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Starting check pointing...
2018-08-15 10:19:21.808+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Starting store flush...
2018-08-15 10:19:21.819+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Store flush completed
2018-08-15 10:19:21.819+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Starting appending check point entry into the tx log...
2018-08-15 10:19:21.862+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Appending check point entry into the tx log completed
2018-08-15 10:19:21.862+0000 INFO  [o.n.k.i.t.l.c.CheckPointerImpl] Check Pointing triggered by database shutdown [17684899]:  Check pointing completed
2018-08-15 10:19:21.863+0000 INFO  [o.n.k.i.t.l.p.LogPruningImpl] Log Rotation [0]:  Starting log pruning.
2018-08-15 10:19:21.863+0000 INFO  [o.n.k.i.t.l.p.LogPruningImpl] Log Rotation [0]:  Log pruning complete.
2018-08-15 10:19:23.040+0000 INFO  [o.n.k.NeoStoreDataSource] NeoStores closed
2018-08-15 10:19:23.041+0000 INFO  [o.n.k.i.DiagnosticsManager] --- STOPPING diagnostics START ---
2018-08-15 10:19:23.041+0000 INFO  [o.n.k.i.DiagnosticsManager] --- STOPPING diagnostics END ---
```

Without a Clean shutdown, even it won't break the database integrity (a recovery process will run the next time you start the neo4j), there is chance to lose data.

#### The workaround

We need to send SIGINT to the neo4j Java process.

Relevant references:
https://stackoverflow.com/questions/813086/can-i-send-a-ctrl-c-sigint-to-an-application-on-windows  
https://stackoverflow.com/questions/39486647/sending-keys-to-an-open-windows-in-powershell

We made a simple script to do this which you can download here: https://coderobin.blob.core.windows.net/public/tools/Kill-Process.ps1

Running this powershell script should allow you to Clean shutdown the neo4j.
