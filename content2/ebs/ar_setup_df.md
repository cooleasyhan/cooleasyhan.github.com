date: 2015-12-01
title: AR 设置数据data fix
tags: ebs
sulg: ar_setup_df

## 通知单行：
```
--1.备份数据
CREATE TABLE Ar_Memo_Lines_All_Tl_0529 AS
  SELECT * FROM Ar_Memo_Lines_All_Tl;

--2. 创建临时表
CREATE TABLE Cux_Ar_Memo_Lines_Df_0529(Memo_Line_Id NUMBER, Ou_Name VARCHAR2(240), NAME VARCHAR2(50), Description VARCHAR2(80), New_Name VARCHAR2(52), New_Description VARCHAR2(90))

--3. 修正临时表中数据
UPDATE Cux_Ar_Memo_Lines_Df_0529 d SET(Memo_Line_Id, New_Name, New_Description) = (
  SELECT Tl.Memo_Line_Id, d.Name || '_O', d.Description || '-历史'
    FROM Ar_Memo_Lines_All_Tl Tl, Hr_Operating_Units Ou
   WHERE Tl.Org_Id = Ou.Organization_Id
     AND Ou.Name = d.Ou_Name
     AND Tl.Name = d.Name
     AND Tl.Description = d.Description
     AND Tl.Language = 'ZHS');

--4手工将cux_ar_memo_lines_df_0529中 合作方分成中转-第三方平台-联运手游 改为 合作方分成中转-第三方平台-联运手_O

--5 do udpate
UPDATE Ar_Memo_Lines_All_Tl Tl
   SET (NAME, Description) =
       (SELECT New_Name, New_Description
          FROM Cux_Ar_Memo_Lines_Df_0529 d
         WHERE d.Memo_Line_Id = Tl.Memo_Line_Id)
WHERE EXISTS (SELECT 1
          FROM Cux_Ar_Memo_Lines_Df_0529 d
         WHERE d.Memo_Line_Id = Tl.Memo_Line_Id);

--6 check
SELECT *
  FROM Ar_Memo_Lines_All_Tl t, Cux_Ar_Memo_Lines_Df_0529 d
WHERE t.Memo_Line_Id = d.Memo_Line_Id
   AND (t.Name <> d.New_Name OR t.Description <> d.New_Description);

```
## 事务处理类型:

```
--1.备份数据
CREATE TABLE Ra_Cust_Trx_Types_All_0528 AS
  SELECT * FROM Ra_Cust_Trx_Types_All;
--2. 创建临时表
CREATE TABLE Cux_Ra_Cust_Trx_Df_0528(Cust_Trx_Type_Id NUMBER,
                                     Ou_Name VARCHAR2(240),
                                     NAME VARCHAR2(20),
                                     New_Name VARCHAR2(20),
                                     New_Description VARCHAR2(80));
--3. 修正临时表中数据
UPDATE Cux_Ra_Cust_Trx_Df_0528 d
   SET Cust_Trx_Type_Id =
       (SELECT Cust_Trx_Type_Id
          FROM Hr_Operating_Units Ou, Ra_Cust_Trx_Types_All t
         WHERE Ou.Organization_Id = t.Org_Id
           AND Ou.Name = d.Ou_Name
           AND t.Name = d.Name);

--4 do udpate
UPDATE Ra_Cust_Trx_Types_All t
   SET (t.Name, t.Description) =
       (SELECT New_Name, New_Description
          FROM Cux_Ra_Cust_Trx_Df_0528 d, Hr_Operating_Units Ou
         WHERE d.Ou_Name = Ou.Name
           AND d.Cust_Trx_Type_Id = t.Cust_Trx_Type_Id
           AND d.Name = t.Name
           AND Ou.Organization_Id = t.Org_Id)
WHERE EXISTS (SELECT 1
          FROM Cux_Ra_Cust_Trx_Df_0528 d, Hr_Operating_Units Ou
         WHERE d.Ou_Name = Ou.Name
           AND d.Cust_Trx_Type_Id = t.Cust_Trx_Type_Id
           AND d.Name = t.Name
           AND Ou.Organization_Id = t.Org_Id);
--5 check
SELECT a.Name, a.Description, d.New_Name, d.New_Description
  FROM Ra_Cust_Trx_Types_All a, Cux_Ra_Cust_Trx_Df_0528 d
WHERE a.Cust_Trx_Type_Id = d.Cust_Trx_Type_Id
   AND (a.Name <> d.New_Name OR a.Description <> d.New_Description)



```
