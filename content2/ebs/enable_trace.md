Date: 2015-11-09
Title: 启用session级别的trace
Tags: Oracle, PLSQL
sulg: enable_trace

```
DECLARE
  Session_Id binary_integer;
  Serial_Num binary_integer;

BEGIN
  SELECT Sid, Serial#
    INTO Session_Id, Serial_Num
    FROM V$session s
   WHERE s.Sid = Userenv('SID');
  Dbms_Monitor.session_trace_enable(Session_Id => Session_Id,
                                    Serial_Num => Serial_Num,
                                    Waits      => TRUE,
                                    Binds      => false);
  EXECUTE IMMEDIATE ('ALTER Session SET Tracefile_Identifier = ''yihan''');

  FOR Rec IN (SELECT * FROM Cux_Gl_Code_Combinations_b_v Gcc) LOOP
    NULL;
  END LOOP;
  Dbms_Monitor.Session_Trace_Disable;
END;
```
