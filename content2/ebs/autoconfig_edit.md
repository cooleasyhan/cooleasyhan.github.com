title: 定位autoconfig模板文件
date: 2015-11-30
tags: ebs
sulg: autoconfig_edit


1、定位模版文件  
$AD_TOP/bin/adtmplreport.sh contextfile=$CONTEXT_FILE target=$FORMS_WEB_CONFIG_FILE
以上命令，通过查看报表日志，可分别得到如下模版结果：
$FND_TOP/admin/template/forms_web_1012_cfg.tmp

2、创建custom目录  
mkdir $FND_TOP/admin/template/custom

3、把上述文件拷贝到custom目录  
cp -i $FND_TOP/admin/template/forms_web_1012_cfg.tmp $FND_TOP/admin/template/custom/forms_web_1012_cfg.tmp


4、编辑custom目录下的模版文件  
forms_web_1012_cfg.tmp文件中的archive2=修改为下面的行：

  添加金税的jar包  
  archive2=,/OA_JAVA/hand/hand.jar


5、在测试模式运行autoconfig   
$AD_TOP/bin/adchkcfg.sh contextfile=$CONTEXT_FILE

6、正式运行autoconfig  
$AD_TOP/bin/adconfig.sh contextfile=$CONTEXT_FILE

metalink参考:270519.1
