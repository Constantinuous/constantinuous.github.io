

## Clarification

The databases that participate in replication are referred to as the publication database,
distribution database, and subscription database.

Be aware that a single SQL Server instance can play more than one role in the replication
context. For example, the Publisher and the Distributor are often the same instance.

Publication, distribution, and subscription are logical replication concepts rather than
physical objects; they are not referring to database objects or databases. 

A replication agent is always called an Agent, and a SQL Server Agent job is always called a
job, so the term, Distributor never refers to a Distribution Agent or the SQL Server Agent
job that is used to execute the Distribution Agent. Rather, the term always refers to an
instance of SQL Server or to the machine running that instance.

## Replication Types
SQL Server supports three types of replication: snapshot, transactional, and merge
replication. There is also peer-to-peer replication, which is transactional replication with
a few enhancements that allow for bi-directional updates.

## Setting Up the Distributor
The Distributor lies at the core of transactional replication. It must be available when you
set up the other components, so you need to configure it first. Although the Distributor
can reside on its own SQL Server instance, often the Publisher and the Distributor are set
up on the same machine

On the Snapshot Folder page (shown in Figure 2-5), you specify the location for the
snapshot folder. 

When you set up the snapshot folder, you must also ensure that the appropriate rights
have been granted on that folder. To keep this example simple, simply grant write access
to the Authenticated Users group

In addition, you must ensure that file sharing has been enabled on the snapshot folder
and that read access to that share has been granted to the Everyone group, as shown in
Figure 2-7.

If these folder security settings are not set up correctly, you'll have to deal with silent
failures, that is, failures that don't generate obvious or even easy-to-find error messages,
so double-check that you got them right. 

On the Publishers page, you specify the Publishers that should have access to the
Distributor.


## Important!

* If you use Sql Server Agent for publication, you need to open SQL Server Configuration Manager and check that the service is running!!!
* For the IIS set the execution of scripts must be allowed in handler options
* Security Best Practises: https://technet.microsoft.com/en-us/library/ms151255(v=sql.105).aspx
