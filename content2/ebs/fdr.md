Date: 2015-11-04
Title: Forms Runtime Diagnostics (FRD) 
Tags: Oracle, EBS
sulg: fdr


The Forms Runtime Diagnostic (FRD) is a method for capturing all events that occur in a form session and writing a file with the details of those events. There is overhead in writing this file, so you should only use this for the purposes of development and debugging

When a form is run with FRD enabled, a combination of external user-application interactions and internal Forms processing events are written in chronological order to a log on the file system.

> Note: The Forms runtime diagnostics (FRD)  output is written to an intermediate OS buffer, and only written to the physical file when the buffer is full. The output buffer is finally flushed when the process terminates. Hence one need to take into consideration below points 

> 1. Make sure the forms session has ended, otherwise you may lose the last few lines which are usually the most important. . If the client has terminated abnormally, you may have to wait for the frmweb process to timeout, which is 5 minutes by default (timeout is governed by value of env variable FORMS_TIMEOUT). If you are unsure whether the process has terminated, you can obtain the process id from the header in the FRD log file and check whether the process ended using appropriate OS commands. 

> 2. If you are reproducing FRM-xxx errors. Please ensure you log off (after reproducing the error)



## active FRD

- Method1: Set the profile option 'ICX: Forms Launcher' to:

|In Servlet Mode | Socket Mode
|----------------|-------------------
|https://hostname.domain:port/forms/frmservlet?record=collect |ttps://hostname.domain:port/OA_HTML/frmservlet?record=collect



- Method2: Set the profile option 'Forms Runtime Parameters' to 'record=collect'



## log file location
The FRD log file will be written  in the directory pointed by environment variable **$FORMS_TRACE_DIR**. By default, the trace file with name `collect_<pid>` gets written in $FORMS_TRACE_DIR , where `<pid>` is the process identifier.This is a simple text file and can be viewed directly.


- - -
## Refrences


[oracle support Document 438652.1](https://support.oracle.com/epmos/faces/DocumentDisplay?parent=DOCUMENT&sourceId=1238633.1&id=438652.1&_adf.ctrl-state=hfqxo0l92_53&_afrLoop=345047785108442#Option_1_frd)
