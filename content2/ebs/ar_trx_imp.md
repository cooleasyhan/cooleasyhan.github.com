Date: 2015-11-14
Title: AR 的事务处理接口
Tags: Oracle, EBS
sulg: ar_trx_imp

最近由于会有大量的接口开发，重构了现有AR 的事务处理接口，将其作为后续所有事务处理接口的统一入口。在后期的项目中看看效果，如果效果OK， 考虑把收款接口，核销接口也用类似方法重构下，从测试脚本来看，使用还是比较简单的，就是感觉方法的调用会不会多了点，不过个人挺习惯和喜欢这种无脑的调用方法，后面会贴出来一个精简版的测试脚本


结构如下：

- 使用一个第三方的表作为借口表，如 Cux_Ar_Trx_Imp_t_Pub， 数据通过INTERFACE_RUN_ID 进行分组
- 该表中有所有AR TRX api 所需的字段，后期随业务增加可以添加
- 创建 PACKAGE Cux_Ar_Trx_Imp_Pub, 提供两个入口方法， #主要参数为INTERFACE_RUN_ID
	- Validate_Main
	- Import_Main
- 为了方便使用这个接口包 Cux_Ar_Trx_Imp_Pub ， 对insert 数据到Cux_Ar_Trx_Imp_t_Pub做了一层封装， 包： Cux_Ar_Trx_Imp_Pub_Wrap， 主要思想为对创建事务处理所需要的参数进行拆分方便使用和修改。

	- 利用record进行数据拆分：
	- 主要提供几类方法：
		- init : 生成interface_run_id， 初始化变量
		- build 方法, build 方法帮忙生成上述record记录
		- Add_Header, Add_Line 利用生成的record， 添加header line 进入到接口表
			- 如果需要一次生成多个发票，Add_Trx_Header与Add_Trx_Line交替执行即可。 
			- 其中记录会根据g_Type_Trx_Header_Feilds_Rec.Trx_Group_Identify 和g_Type_Trx_Header_Feilds_Rec.Trx_Number 进行分组。
			- Trx_Number为空时根据Trx_Group_Identify进行分组 
			- 每添加一个行时需要制定一个source_id， 用于后期获取错误信息。
		- Run_Validate ， 执行校验操作
		- Run_Import ， 执行导入操作
		- Get_Msg ，出现错误根据source_id 获取错误信息
```
# Records 定义
g_Type_Batch_Source_Rec
g_Type_Trx_Header_Feilds_Rec
g_Type_Trx_Type_Rec
g_Type_Customer_Rec
g_Type_Exchange_Rate_Rec
g_Type_Sales_Rec
g_Type_Term_Rec

TYPE g_Type_Header_Rec IS RECORD(
    Batch_Source_Rec               g_Type_Batch_Source_Rec,
    Trx_Header_Feilds_Rec          g_Type_Trx_Header_Feilds_Rec,
    Trx_Type_Rec                   g_Type_Trx_Type_Rec,
    Customer_Rec                   g_Type_Customer_Rec,
    Exchange_Rate_Rec              g_Type_Exchange_Rate_Rec,
    Term_Rec                       g_Type_Term_Rec,
    Sales_Rec                      g_Type_Sales_Rec,
    Header_Attribute_Rec           Cux_Flex_Utl.Attribute_Rec_Type,
    Interface_Header_Attribute_Rec Cux_Flex_Utl.Attribute_Rec_Type);
    
    
g_Type_Trx_Line_Feilds_Rec
g_Type_Line_Amount_Rec
g_Type_Memo_Line_Rec
g_Type_Tax_Rec
TYPE g_Type_Line_Rec IS RECORD(
    Trx_Line_Feilds_Rec          g_Type_Trx_Line_Feilds_Rec,
    Line_Amount_Rec              g_Type_Line_Amount_Rec,
    Memo_Line_Rec                g_Type_Memo_Line_Rec,
    Tax_Rec                      g_Type_Tax_Rec,
    Line_Attribute_Rec           Cux_Flex_Utl.Attribute_Rec_Type,
    Interface_Line_Attribute_Rec Cux_Flex_Utl.Attribute_Rec_Type);
```


```
# build 方法
Build_Batch_Source_Rec
Build_Trx_Header_Feilds_Rec
Build_Trx_Type_Rec
Build_Term_Rec
Build_Exchange_Rate_Rec
Build_Attribute_Rec
Build_Cust_Rec

Build_Trx_Line_Feilds_Rec
Build_Line_Amount_Rec
Build_Line_Amount_Rec
Trx_Number为空时根据Trx_Group_Identify进行分组

```

```
# Add Header, Add Line
PROCEDURE Add_Trx_Header(Pir_Batch_Source_Rec          IN g_Type_Batch_Source_Rec,
                           Pir_Trx_Header_Feilds_Rec     IN g_Type_Trx_Header_Feilds_Rec,
                           Pir_Trx_Type_Rec              IN g_Type_Trx_Type_Rec,
                           Pir_Customer_Rec              IN g_Type_Customer_Rec,
                           Pir_Term_Rec                  IN g_Type_Term_Rec DEFAULT NULL,
                           Pir_Exchange_Rate_Rec         IN g_Type_Exchange_Rate_Rec DEFAULT NULL,
                           Pir_Header_Attribute_Rec      IN Cux_Flex_Utl.Attribute_Rec_Type DEFAULT NULL,
                           Pir_Interface_Header_Attr_Rec IN Cux_Flex_Utl.Attribute_Rec_Type DEFAULT NULL,
                           Pir_Sales_Rec                 IN g_Type_Sales_Rec DEFAULT NULL);
                           
PROCEDURE Add_Trx_Line(Pir_Trx_Line_Feids_Rec      IN g_Type_Trx_Line_Feilds_Rec DEFAULT NULL,
                         Pir_Line_Amount_Rec         IN g_Type_Line_Amount_Rec,
                         Pir_Memo_Line_Rec           IN g_Type_Memo_Line_Rec DEFAULT NULL,
                         Pir_Tax_Rec                 IN g_Type_Tax_Rec DEFAULT NULL,
                         Pir_Line_Attribute_Rec      IN Cux_Flex_Utl.Attribute_Rec_Type DEFAULT NULL,
                         Pir_Interface_Line_Attr_Rec IN Cux_Flex_Utl.Attribute_Rec_Type DEFAULT NULL,
                         Pin_Source_Id               IN NUMBER);
```

```
  # validate, import , get_msg
  PROCEDURE Run_Validate(Pov_Return_Status OUT NOCOPY VARCHAR2,
                         Pov_Msg_Data      OUT NOCOPY VARCHAR2,
                         Piv_Commit        IN VARCHAR2 DEFAULT Fnd_Api.g_False);
  FUNCTION Get_Msg(Pin_Source_Id IN VARCHAR2) RETURN VARCHAR2;

  PROCEDURE Run_Import(Pov_Return_Status OUT NOCOPY VARCHAR2,
                       Pov_Msg_Data      OUT NOCOPY VARCHAR2,
                       Piv_Commit        IN VARCHAR2 DEFAULT Fnd_Api.g_False);
```



```
-- Created on 2015/11/13 by YIHAN 
DECLARE
  -- Local variables here

  Batch_Source_Rec      Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Batch_Source_Rec;
  Trx_Header_Fields_Rec Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Trx_Header_Feilds_Rec;
  Trx_Type_Rec          Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Trx_Type_Rec;
  Cust_Rec              Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Customer_Rec;
  Exchange_Rate_Rec     Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Exchange_Rate_Rec;
  Term_Rec              Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Term_Rec;

  Header_Attr_Rec           Cux_Flex_Utl.Attribute_Rec_Type;
  Header_Interface_Attr_Rec Cux_Flex_Utl.Attribute_Rec_Type;
  Line_Attribute_Rec        Cux_Flex_Utl.Attribute_Rec_Type;
  Line_Interface_Attr_Rec   Cux_Flex_Utl.Attribute_Rec_Type;

  Trx_Line_Feids_Rec Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Trx_Line_Feilds_Rec;
  Line_Amount_Rec    Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Line_Amount_Rec;
  Memo_Line_Rec      Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Memo_Line_Rec;
  Tax_Rec            Cux_Ar_Trx_Imp_Pub_Wrap.g_Type_Tax_Rec;

  Lv_Msg VARCHAR2(20000);
BEGIN
  IF :Init_Org = 'Y' THEN
    Mo_Global.Init('AR');
  END IF;

  :Rst := Fnd_Api.g_Ret_Sts_Success;
  Cux_Ar_Trx_Imp_Pub_Wrap.Init;
  IF Cux_Ar_Trx_Imp_Pub_Wrap.Get_Interface_Run_Id < 0 THEN
    :Rst := Fnd_Api.g_Ret_Sts_Error;
  END IF;

  Batch_Source_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Batch_Source_Rec(Piv_Batch_Source_Name => 'XXXXX');

  Trx_Header_Fields_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Trx_Header_Feilds_Rec(Piv_Trx_Group_Identify => '1234',
                                                                               Piv_Org_Name           => 'XXXXX',
                                                                               Pid_Trx_Date           => SYSDATE - 30,
                                                                               Pid_Gl_Date            => SYSDATE - 30,
                                                                               Piv_Currency_Code      => 'CNY');

  Trx_Type_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Trx_Type_Rec(Piv_Cust_Trx_Type_Name => 'XXXXX');
  Cust_Rec     := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Cust_Rec(Piv_Bill_To_Customer_Name => 'XXXXX',
                                                         Piv_Ship_To_Customer_Name => 'XXXXX');

  Header_Attr_Rec           := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Attribute_Rec(Piv_Attribute_Category => 'XXXXX');
  Header_Interface_Attr_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Attribute_Rec;

  Cux_Ar_Trx_Imp_Pub_Wrap.Add_Trx_Header(Pir_Batch_Source_Rec          => Batch_Source_Rec,
                                         Pir_Trx_Header_Feilds_Rec     => Trx_Header_Fields_Rec,
                                         Pir_Trx_Type_Rec              => Trx_Type_Rec,
                                         Pir_Customer_Rec              => Cust_Rec,
                                         Pir_Term_Rec                  => Term_Rec,
                                         Pir_Exchange_Rate_Rec         => Exchange_Rate_Rec,
                                         Pir_Header_Attribute_Rec      => Header_Attr_Rec,
                                         Pir_Interface_Header_Attr_Rec => Header_Interface_Attr_Rec);

  Line_Amount_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Line_Amount_Rec(Pin_Amount => '-1000');

  Memo_Line_Rec := Cux_Ar_Trx_Imp_Pub_Wrap.Build_Memo_Line_Rec(Piv_Memo_Line_Name => 'XXXXX');

  Cux_Ar_Trx_Imp_Pub_Wrap.Add_Trx_Line(Pir_Trx_Line_Feids_Rec      => Trx_Line_Feids_Rec,
                                       Pir_Line_Amount_Rec         => Line_Amount_Rec,
                                       Pir_Memo_Line_Rec           => Memo_Line_Rec,
                                       Pir_Tax_Rec                 => Tax_Rec,
                                       Pir_Line_Attribute_Rec      => Line_Attribute_Rec,
                                       Pir_Interface_Line_Attr_Rec => Line_Interface_Attr_Rec,
                                       Pin_Source_Id               => 112);
  Cux_Ar_Trx_Imp_Pub_Wrap.Add_Trx_Line(Pir_Trx_Line_Feids_Rec      => Trx_Line_Feids_Rec,
                                       Pir_Line_Amount_Rec         => Line_Amount_Rec,
                                       Pir_Memo_Line_Rec           => Memo_Line_Rec,
                                       Pir_Tax_Rec                 => Tax_Rec,
                                       Pir_Line_Attribute_Rec      => Line_Attribute_Rec,
                                       Pir_Interface_Line_Attr_Rec => Line_Interface_Attr_Rec,
                                       Pin_Source_Id               => 112);
  Cux_Ar_Trx_Imp_Pub_Wrap.Run_Validate(Pov_Return_Status => :Lv_Return_Status,
                                       Pov_Msg_Data      => :Lv_Msg_Data,
                                       Piv_Commit        => Fnd_Api.g_False);

  IF :Lv_Return_Status = Fnd_Api.g_Ret_Sts_Success THEN
    Cux_Ar_Trx_Imp_Pub_Wrap.Run_Import(Pov_Return_Status => :Lv_i_Return_Status,
                                       Pov_Msg_Data      => :Lv_i_Msg_Data,
                                       Piv_Commit        => Fnd_Api.g_False);
  ELSE
    Dbms_Output.Put_Line(Cux_Ar_Trx_Imp_Pub_Wrap.Get_Interface_Run_Id);
    :Lv_Err_Msg := Cux_Ar_Trx_Imp_Pub_Wrap.Get_Msg(Pin_Source_Id => 112);
  
  END IF;

EXCEPTION
  WHEN OTHERS THEN
    Dbms_Output.Put_Line(Dbms_Utility.Format_Error_Stack);
    Dbms_Output.Put_Line(Dbms_Utility.Format_Error_Backtrace);
    Dbms_Output.Put_Line(Dbms_Utility.Format_Call_Stack);
    RAISE;
END;

```
