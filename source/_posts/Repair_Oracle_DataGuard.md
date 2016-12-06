---
title: 修复/重建 Oracle Dataguard 容灾
categories: 
- DataBase
- Oracle
tags: 
- Oracel Dataguard
- Database HA
- Oracle RAC
---

##  参考 ##
[Oracle Database Online Documentation 11g Release 2 (11.2)](http://docs.oracle.com/cd/E11882_01/index.htm)
[Managing Physical and Snapshot Standby Databases](http://docs.oracle.com/cd/E11882_01/server.112/e41134/manage_ps.htm#SBYDB00700)
[Using RMAN to Back Up and Restore Files](http://docs.oracle.com/cd/E11882_01/server.112/e41134/rman.htm#SBYDB04700)
[Using RMAN Incremental Backups to Roll Forward a Physical Standby Database](http://docs.oracle.com/cd/E11882_01/server.112/e41134/rman.htm#SBYDB4878)
[Recovering Standby Database Using Incremental RMAN ](http://www.dba-oracle.com/t_rman_86_recover_standby_incremental.htm)
[Using RMAN Incremental Backups to Refresh Standby Database](http://oracleinaction.com/using-rman-incremental-backups-refresh-standby-database/)
[Roll Foward a Physical Standby on 11gR2 ](https://asanga-pradeep.blogspot.com/2011/10/roll-foward-physical-standby-on-11gr2.html)
[ORA-10458: standby database requires recovery](http://rajiboracle.blogspot.com/2015/07/ora-10458-standby-database-requires.html)
[Syncing Standby Database Using SCN Based Backup](https://saruamit4.wordpress.com/2014/05/03/recovering-standby-database-using-scn-based-backup/)

## 问题描述与分析 ##


1. OEM 无法正常登录
2. 登录系统，发现数据库已经挂掉，包括ASM
3. 以往经验，查看了下磁盘使用， 发现 /u01 使用100%，由于/u01只按抓个oracle database software，查看各级目录使用率，发现 $ORACLE_HOME/diag  特别是 trace与alter目录爆满，应该是由于数据库异常太久，大量警告与错误等系统运行日志产生，占满磁盘，遂删之。由于是ORACLE RAC ，ASM，ORACLE Cluster， Database 自动修复。
4. 查看数据库状态（open_mode) 为 read，write。
5. 普通用户连接  ora-00257
6. asm中archivelog所在 分区/卷爆满，进入rman删除 archivel log，无法删除，原因是 

> RMAN-08120: WARNING: archived log not deleted, not yet applied by standby

即：主库日志未被备库接受并应用，所以无法删除。
7. 查看备库进程与状态

```sql
sql>select process,thread#,sequence#,status from v$managed_standby ; 
```
发现没有MRP

8.  启动redo apply 
```sql
sql> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE USING CURRENT LOGFILE DISCONNECT;

```
人然失败 ，重启standby，
```sql
SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE;
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE
*
ERROR at line 1:

ORA-00283: recovery session canceled due to errors

ORA-00368: checksum error in redo log block

ORA-00353: log corruption near block 330320 change 16534734342 time 10/30/2016

23:55:38

ORA-00334: archived log: '/arch/2_140517_876346982.dbf'
```
日志文件右问题 
9. 由主库传送相应日志
```bash
ASMCMD> cp +ARCHDG/ZJRKK/ARCHIVELOG/2016_10_30/thread_2_seq_140517.14887.926639767  /home/grid/2_140517_876346982.dbf
ASMCMD> exit
$ scp /home/grid/2_140517_876346982.dbf oracle@172.17.178.29:/arch/

```
10. 备库（standby)发现重新启动日志文件报错
11. 至此，可判定日志文件损坏（主库，备库都损坏），需重新修复dataguard。


## 环境 ##
|domain|oracle_sid|database_name|hostname|IP|oradata_storagetype|description|
|:----:|:--------:|:-----------:|:-------:|:-:|:-:|:-:|
|ORACLE RAC|zjrkk2|zjrkk|rkkdbserdw01|172.17.178.5|ASM|数据库集群几点之一|
|ORACLE RAC|zjrkk1|zjrkk|rkkdbserdw02|172.17.178.33|ASM|数据库集群节点之一|
|ORACLE STANDBY|zjrkk||zjrkk|rkkdbserbak02|172.17.178.29|Local FS(ext4)|备库|



## 解决过程 ##



### 解决思路与方案 ###
1. 强制删除已满的archivlog，排除 ora-00257,使主库恢复
2. 同步主备数据库
	2.1. 查看备库当前scn
	2.2. 由备库scn号开始，在主库rman中增量备份
	2.3. 恢复备库到更高当前主库scn
3. 修复dataguard同步，主备修复

### 解决处理过程日志 ###

#### 删除主库已满archivlog，恢复主库正常运行 ####

```bash

$ su - grid
$ asmcmd
ASMCMD> rm -rf +ARCHDG/ZJRKK/ARCHIVELOG/2016_03_19/

ASMCMD> rm -rf +ARCHDG/ZJRKK/ARCHIVELOG/2016_10_30   

ASMCMD> rm -rf +ARCHDG/ZJRKK/ARCHIVELOG/2016_10_31   

ASMCMD>  rm -rf +ARCHDG/ZJRKK/ARCHIVELOG/2016_11_01

ASMCMD>  rm -rf +ARCHDG/ZJRKK/ARCHIVELOG/2016_11_0* 

```

```sql
SQL> conn app/app

ERROR:

ORA-00257: archiver error. Connect internal only, until freed.

```
依然报错00257
```sql

NAME                                                           TOTAL_MB    FREE_MB FREE_MB/TOTAL_MB*100 STATE

------------------------------------------------------------ ---------- ---------- -------------------- ----------------------
  ARCHDG                                                          1024000    1014475           99.0698242 CONNECTED

DATA_FASTDG01                                                   1742848     988840           56.7370189 CONNECTED

DATA_SLOWDG01                                                   2048000    1268401           61.9336426 CONNECTED

OCRDISK                                                           30720      29794           96.9856771 MOUNTED


```

发现 ocrdisk mounted 可能有问题吧。
另外临时想到，在asmcmd中系统级别的删除日志，但rman中未删除，可能也是问题。
在rman中删除日志报错

```bash
$ rman target /
RMAN> delete noprompt archivelog all completed before 'sysdate-5' ;
RMAN-08120: WARNING: archived log not deleted, not yet applied by standby 

```
依然是因为备库没有apply导致无法删除

```bash
$ rman target /
RMAN> crosscheck archivelog all ;
RMAN> delete expired archivelog all ; 
RMAN> report obsolete ;
RMAN> delete obsolete ; 
```
至此，主库集群恢复正常使用，修复DataStage与Oracle Goldengate。


### 同步主库备库 ###

1. stop Redo Apply on the standby database

	```sql
	
	SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
	ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL
	*
	ERROR at line 1:
	ORA-16136: Managed Standby Recovery not active
	
	```

备库不再活动状态

2. On the standby database, compute the FROM SCN for the incremental backup. This is done differently depending on thea situation:

   2.1 On a standby that lags far behind the primary database, query the V$DATABASE view and record the current SCN of the standby database:

   ```sql
   SQL> SELECT CURRENT_SCN FROM V$DATABASE;
   
   CURRENT_SCN
   -----------
   1.6535E+10

   SQL> set numwidth 50 ;
   SQL> SELECT CURRENT_SCN FROM V$DATABASE;
   
   CURRENT_SCN
   --------------------------------------------------
				         16534733512
   
   ```

   2.2 On a standby that has widespread nologging changes, query the V$DATAFILE view to record the lowest FIRST_NONLOGGED_SCN:

   ```sql
   SQL> SELECT MIN(FIRST_NONLOGGED_SCN) FROM V$DATAFILE  WHERE FIRST_NONLOGGED_SCN>0;

        	         MIN(FIRST_NONLOGGED_SCN)
    --------------------------------------------------
   
   ```

   2.3 On a standby that has nologging changes on a subset of datafiles, query the V$DATAFILE view, as follows:

   ```sql
   SQL> SELECT FILE#, FIRST_NONLOGGED_SCN FROM V$DATAFILE  WHERE FIRST_NONLOGGED_SCN > 0;

   no rows selected
   ```
   2.4  Check current log sequence on primary
     - on 172.17.178.33
     
     ```sql
     SQL> archive log list ; 

     Database log mode              Archive Mode

     Automatic archival             Enabled

     Archive destination            USE_DB_RECOVERY_FILE_DEST

     Oldest online log sequence     172963

     Next log sequence to archive   172967

     Current log sequence           172967

     ```

     - on 172.17.178.5
     
     ```sql
     SQL> archive log list ; 

     Database log mode              Archive Mode

     Automatic archival             Enabled

     Archive destination            USE_DB_RECOVERY_FILE_DEST

     Oldest online log sequence     142295

     Next log sequence to archive   142299

     Current log sequence           142299

     ```

     2.5 check that all the archived logs prior to the current log have been sent to standby
     
     ```sql

     SQL> select max(sequence#) from v$archived_log;
     
     MAX(SEQUENCE#)
     --------------
     172966
     
     ```

3. Connect to the primary database as the RMAN target and create an incremental backup from the current SCN (for a standby lagging far behind the primary) or from the lowest FIRST_NONLOGGED_SCN (for a standby with widespread nologging changes) of the standby database that was recorded in step 2:

  ```bash
  RMAN> BACKUP INCREMENTAL FROM SCN 16534733512 DATABASE FORMAT '/RKK/CENTER/RKKDBSER01/BAK/TMP/ForStandby_%U' tag 'FORSTANDBY';
  ```

4. If the backup pieces are not on shared storage, then transfer all the backup pieces created on the primary to the standby:

  ```bash

   $ scp ./* oracle@172.17.178.29:/backup/ForStandby/

   oracle@172.17.178.29's password: 

   ForStandby_nsrlfv3d_1_1                                                                   100%   27GB  97.6MB/s   04:44    

   ForStandby_ntrlg176_1_1                                                                   100%   23GB  90.6MB/s   04:24    

   ForStandby_nurlg3ba_1_1                                                                   100%  392MB 130.8MB/s   00:03  
   
   ```
5. If you had to copy the backup pieces in the previous step, or if you are not connected to the recovery catalog for the entire process, then you must catalog the new backup pieces on the standby (otherwise, go on to the next step):
  
  ```bash
  RMAN> CATALOG START WITH '/backup/ForStandby/ForStandby';
  ```
6. Connect to the standby database as the RMAN target and execute the REPORT SCHEMA statement to ensure that the standby database site is automatically registered and that the files names at the standby site are displayed:

  ```bash
  
  RMAN> REPORT SCHEMA;

  RMAN-06139: WARNING: control file is not current for REPORT SCHEMA

  ```
7. Connect to the standby database as the RMAN target and apply incremental backups by executing the following commands. Note that the RESTORE STANDBY CONTROLFILE FROM TAG command only works if you are connected to the recovery catalog for the entire process. Otherwise, you must use the RESTORE STANDBY CONTROLFILE FROM '<control file backup filename>' command.
  
  ```bash
  RMAN> RESTORE STANDBY CONTROLFILE FROM TAG 'FORSTANDBY';

  Starting restore at 22-NOV-16

  allocated channel: ORA_DISK_1

  channel ORA_DISK_1: SID=2833 device type=DISK

  RMAN-00571: ===========================================================

  RMAN-00569: =============== ERROR MESSAGE STACK FOLLOWS ===============

  RMAN-00571: ===========================================================

  RMAN-03002: failure of restore command at 11/22/2016 10:37:37

  RMAN-06563: control file or SPFILE must be restored using FROM AUTOBACKUP

  RMAN>restore standby controlfile from  '/backup/ForStandby/ForStandby_nurlg3ba_1_1' ;

  Starting restore at 22-NOV-16

  using channel ORA_DISK_1

  channel ORA_DISK_1: restoring control file

  channel ORA_DISK_1: restore complete, elapsed time: 00:00:15

  output file name=/data/controlfile/controlfile01.ctl

  output file name=/data/controlfile/controlfile02.ctl

  output file name=/data/controlfile/controlfile03.ctl

  Finished restore at 22-NOV-16

  RMAN> ALTER DATABASE MOUNT;

  RMAN> RECOVER DATABASE NOREDO;

  ```
8. On standbys that have widespread nologging changes or that have nologging changes on a subset of datafiles, query the V$DATAFILE view to verify there are no datafiles with nologged changes. The following query should return zero rows:
  
  ```sql
  SQL> SELECT FILE#, FIRST_NONLOGGED_SCN FROM V$DATAFILE  WHERE FIRST_NONLOGGED_SCN > 0;

  ```
9. Start Redo Apply on the physical standby database:
  
  ```sql
  SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE  USING CURRENT LOGFILE DISCONNECT FROM SESSION;

  SQL> select process,thread#,sequence#,status from v$managed_standby ;

  PROCESS      THREAD#  SEQUENCE# STATUS
  --------- ---------- ---------- ------------
  ARCH               0          0 CONNECTED
  ARCH               0          0 CONNECTED
  ARCH               1     172968 CLOSING
  ARCH               0          0 CONNECTED
  RFS                0          0 IDLE
  RFS                0          0 IDLE
  RFS                1     172969 IDLE
  RFS                2     142300 IDLE
  RFS                0          0 IDLE
  RFS                0          0 IDLE
  RFS                0          0 IDLE
  MRP0               2     140517 WAIT_FOR_GAP
  
  SQL> select open_mode from v$database ; 

  OPEN_MODE
  --------------------
  MOUNTED
  

  ```
10. Alter Database open and start redo-apply

  ```sql
  SQL> ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;

  Database altered.

  SQL> ALTER DATABASE OPEN;

  ALTER DATABASE OPEN

  *
  ERROR at line 1:

  ORA-10458: standby database requires recovery

  ORA-01152: file 1 was not restored from a sufficiently old backup

  ORA-01110: data file 1: '/data/oradata/system.256.876346867'
  ```
* 我的内心是崩溃的 *

### 重来一遍试试 ###
- Stop redo transport on primary
  ```bash

  DGMGRL> connect sys/system

  Connected.

  DGMGRL> show configuration 

  Error: 

  ORA-16525: the Data Guard broker is not yet available

  DGMGRL> connect sys/system

  Connected.

  DGMGRL> show configuration 

  Error: 

  ORA-16525: the Data Guard broker is not yet available

  ```

 [Oracle data guard broker configurations - step by step ](http://naeemsheeraz.blogspot.com/2010/06/oracle-data-guard-broker-configurations.html)
  
  ~ 我决定重启主库，包括集群 ~
  [Starting and Stopping Instances and Oracle RAC Databases](https://docs.oracle.com/cd/E11882_01/rac.112/e41960/admin.htm#RACAD801
  [How to STOP and START processes in Oracle RAC and Log Directory Structure ](http://www.oracleracexpert.com/2012/01/how-to-stop-and-start-processes-in.html)
  [steps to stop RAC enviroment with ASM ](http://www.dba-village.com/village/dvp_forum.OpenThread?ThreadIdA=43116&DestinationA=RSS)

  ```bash
  $ emctl stop dbconsole
  $ srvctl stop instance -d zjrkk -i zjrkk1 -o immediate 
  $ srvctl stop listener -n rkkdbserdw02
  $ sudo -c  /u01/app/grid/product/11.2.0/bin/crsctl stop crs
  
  CRS-2791: Starting shutdown of Oracle High Availability Services-managed resources on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.crsd' on 'rkkdbserdw02'
  CRS-2790: Starting shutdown of Cluster Ready Services-managed resources on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.cvu' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.oc4j' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.LISTENER_SCAN1.lsnr' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.OCRDISK.dg' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.ARCHDG.dg' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.DATA_FASTDG01.dg' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.DATA_SLOWDG01.dg' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.cvu' on 'rkkdbserdw02' succeeded
  CRS-2672: Attempting to start 'ora.cvu' on 'rkkdbserdw01'
  CRS-2676: Start of 'ora.cvu' on 'rkkdbserdw01' succeeded
  CRS-2677: Stop of 'ora.DATA_SLOWDG01.dg' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.DATA_FASTDG01.dg' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.ARCHDG.dg' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.LISTENER_SCAN1.lsnr' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.scan1.vip' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.scan1.vip' on 'rkkdbserdw02' succeeded
  CRS-2672: Attempting to start 'ora.scan1.vip' on 'rkkdbserdw01'
  CRS-2676: Start of 'ora.scan1.vip' on 'rkkdbserdw01' succeeded
  CRS-2672: Attempting to start 'ora.LISTENER_SCAN1.lsnr' on 'rkkdbserdw01'
  CRS-2676: Start of 'ora.LISTENER_SCAN1.lsnr' on 'rkkdbserdw01' succeeded
  CRS-2677: Stop of 'ora.oc4j' on 'rkkdbserdw02' succeeded
  CRS-2672: Attempting to start 'ora.oc4j' on 'rkkdbserdw01'
  CRS-2676: Start of 'ora.oc4j' on 'rkkdbserdw01' succeeded
  CRS-2677: Stop of 'ora.OCRDISK.dg' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.asm' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.asm' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.net1.network' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.net1.network' on 'rkkdbserdw02' succeeded
  CRS-2792: Shutdown of Cluster Ready Services-managed resources on 'rkkdbserdw02' has completed
  CRS-2677: Stop of 'ora.crsd' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.crf' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.ctssd' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.evmd' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.asm' on 'rkkdbserdw02'
  CRS-2673: Attempting to stop 'ora.mdnsd' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.crf' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.mdnsd' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.evmd' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.asm' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.cluster_interconnect.haip' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.ctssd' on 'rkkdbserdw02' succeeded
  CRS-2677: Stop of 'ora.cluster_interconnect.haip' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.cssd' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.cssd' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.gipcd' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.gipcd' on 'rkkdbserdw02' succeeded
  CRS-2673: Attempting to stop 'ora.gpnpd' on 'rkkdbserdw02'
  CRS-2677: Stop of 'ora.gpnpd' on 'rkkdbserdw02' succeeded
  CRS-2793: Shutdown of Oracle High Availability Services-managed resources on 'rkkdbserdw02'
  
  $ sudo -c init 6
  $ srvctl start instance -d zjrkk -i zjrkk2

  ```
- 增量备份

  ``` bash
  SQL>     SELECT CURRENT_SCN FROM V$DATABASE;

      CURRENT_SCN
  ---------------
      16588045137

  SQL> SELECT MIN(FIRST_NONLOGGED_SCN) FROM V$DATAFILE  WHERE FIRST_NONLOGGED_SCN>0;

  MIN(FIRST_NONLOGGED_SCN)
  ------------------------

  SQL>  SELECT FILE#, FIRST_NONLOGGED_SCN FROM V$DATAFILE  WHERE FIRST_NONLOGGED_SCN > 0;  

  no rows selected

  ```

  ``` bash
  RMAN> BACKUP INCREMENTAL FROM SCN 16588045137 DATABASE FORMAT '/RKK/CENTER/RKKDBSER01/BAK/TMP/ForStandby_%U_%T' tag 'FORSTANDBY';
  RMAN> BACKUP CURRENT CONTROLFILE FOR STANDBY tag standbycrtl format '/RKK/CENTER/RKKDBSER01/BAK/TMP/control_standby.ctl' ;
  ```

  ```bash
  scp /RKK/CENTER/RKKDBSER01/BAK/TMP/* oracle@172.17.178.29:/backup/ForStandby/
  ```

- 恢复recovery
......

---
SQL> alter database open read only;

 alter database open read only
 *
 ERROR at line 1:

 ORA-10458: standby database requires recovery

 ORA-01152: file 1 was not restored from a sufficiently old backup

 ORA-01110: data file 1: '/data/oradata/system.256.876346867'
---

* 最终结果，无济于事 *

- 停主库，在同步期间，不产生新的数据及归档。
  
