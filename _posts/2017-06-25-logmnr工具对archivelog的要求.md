---
layout: post
title: logmnr工具对archivelog的要求
category: oracle
---
有个数据库反映某张表中的部分数据丢失，使用logmnr查询定位数据丢失的原因。Logmnr分析归档日志文件，需要归档日志文件还在数据库中没有被清除。但是该数据库每天做增量备份，同时清除归档日志，所以尝试从rman增量备份中的归档直接分析。  
发生如下报错：  SQL> exec sys.dbms\_logmnr.add\_logfile(logfilename => '/u01/databak/rman/level1/201706140100/L1\_CJDFWEB\_ARCH\_20170614\_7us6o00l\_1\_1',options => dbms\_logmnr.new);
begin sys.dbms\_logmnr.add\_logfile(logfilename => '/u01/databak/rman/level1/201706140100/L1\_CJDFWEB\_ARCH\_20170614\_7us6o00l\_1\_1',options => dbms\_logmnr.new);   
end;  ORA-01284: 文件 /u01/databak/rman/level1/201706140100/L1\_CJDFWEB\_ARCH\_20170614\_7us6o00l\_1\_1 无法打开  ORA-00317: 标头中的文件类型 0 不是日志文件  ORA-00334: 归档日志: '/u01/databak/rman/level1/201706140100/L1\_CJDFWEB\_ARCH\_20170614\_7us6o00l\_1\_1'ORA-06512: 在 "SYS.DBMS\_LOGMNR", line 68  ORA-06512: 在 line 1

如此可见：  
logmnr在分析日志时，需要保证该日志格式数据库识别。  为了能还原日志归档，需要在rman备份中恢复备份的归档日志文件。  恢复到另外路径指定时间段的日志文件：  run {  allocate channel c1 type disk;  SET ARCHIVELOG DESTINATION TO ‘/u01/archivelog/14’;  
(指定archivelog恢复的路径，即将rman备份里面的archivelog恢复到哪里)SQL’ALTER SESSION SET NLS_DATE_FORMAT=”YYYY-MM-DD HH24:MI:SS”’;  restore archivelog time between ‘2017-06-14 00:00:00’ and ‘2017-06-14 23:59:59’;  
(指定恢复时间)release channel c1;  }

恢复archivelog后，使用logmnr定位表数据丢失的语句及详细信息。具体logmnr使用详见：<a href:https://liuahuo.github.io/logmnr工具的使用>logmnr工具的使用。
