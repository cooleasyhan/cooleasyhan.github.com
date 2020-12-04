Date: 2015-10-30
Title: Log file location for Oracle E-Business Suite
Tags: Oracle, EBS
sulg: ebs_logs


## Log file location for Oracle E-Business Suite R12

- Apache, OC4J and OPMN:
>$LOG_HOME/ora/10.1.3/Apache
$LOG_HOME/ora/10.1.3/j2ee
$LOG_HOME/ora/10.1.3/opmn
- Startup/Shutdown Log files:
>$INST_TOP/apps/$CONTEXT_NAME/logs/appl/admin/log
- Patch log:
>$APPL_TOP/admin/$SID/log/
- Autoconfig log file:
Apps:
>$INST_TOP/apps/$CONTEXT_NAME/admin/log/$MMDDHHMM/adconfig.log
Db:
>$ORACLE_HOME/appsutil/log/$CONTEXT_NAME/<MMDDHHMM>/adconfig.log
$ORACLE_HOME/appsutil/log/$CONTEXT_NAME/<MMDDHHMM>/NetServiceHandler.log
- Concurrent log:
>$INST_TOP/apps/$CONTEXT_NAME/logs/appl/conc/log
- Clone log:
Preclone log files in source instance
	- Apps:
>$INST_TOP/apps/$CONTEXT_NAME/admin/log/ (StageAppsTier_MMDDHHMM.log)
	- Db:
>$ORACLE_HOME/appsutil/log/$CONTEXT_NAME/(StageDBTier_MMDDHHMM.log)

- Clone log files in target instance

	- Apps :
>$INST_TOP/apps/$CONTEXT_NAME/admin/log/ApplyAppsTier_<time>.log
	- Db:
>$ORACLE_HOME/appsutil/log/$CONTEXT_NAME/ApplyDBTier_<time>.log
- Alert Log File:
>$ORACLE_HOME/admin/$CONTEXT_NAME/bdump/alert_$SID.log


## Log file location for Oracle E-Business Suite R11


- OPMN log file
>$ORACLE_HOME/opmn/logs/ipm.log
- Start/Stop script log files
>$COMMON_TOP/admin/log/CONTEXT_NAME/
- Apache, Jserv, JVM log files
>$IAS_ORACLE_HOME/Apache/Apache/logs/ssl_engine_log
$IAS_ORACLE_HOME/Apache/Apache/logs/ssl_request_log
$IAS_ORACLE_HOME/Apache/Apache/logs/access_log
$IAS_ORACLE_HOME/Apache/Apache/logs/error_log
$IAS_ORACLE_HOME/Apache/JServ/logs
- Concurrent log file:
>$APPL_TOP/admin/PROD/log or $APPLLOG/$APPLCSF
- Patch log file:
>$APPL_TOP/admin/PROD/log
- Worker Log:
>$APPL_TOP/admin/PROD/log
- Autoconfig log:
	- Appl:
>$APPL_TOP/admin/SID_Hostname/log//DDMMTime/adconfig.log
	- Db:
>$ORACLE_HOME/appsutil/log/SID_Hostname/DDMMTime/adconfig.log
- Error log:
	- Appl:
>$APPL_TOP/admin/PROD/log
	- Db tarafında:
>$ORACLE_HOME/appsutil/log/SID_Hostname
- Alert Log File:
>$ORACLE_HOME/admin/$CONTEXT_NAME/bdump/alert_$SID.log
- Trace file:
>$ORACLE_HOME/admin/SID_Hostname/udump

