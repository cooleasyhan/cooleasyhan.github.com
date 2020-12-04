title: shell脚本片段
date: 2015-11-30
tags: Linux, Shell
sulg: shell

## read user enter
```
read -p "Read rows CNT[W]: " ROWS_CNT
```

## 获取文件行数
```
ROW=`wc -l $DATA_FILE | awk '{print $1}'`
```

## 浮点数计算
```
per=`awk 'BEGIN{printf "%.2f\n",'$wcnt'/'$scnt'}'`
``` 


## string=>list
```
list="a,
b,
c,
d
"

arr=(${list//,/ })

echo "====count===="

for v in ${arr[@]}
do
    i=`echo $v`
    echo i
done

```


## ubuntu命令行多线程断点续传下载工具-axel  
```
axel -n 10 xxxx.tar.gz
```
