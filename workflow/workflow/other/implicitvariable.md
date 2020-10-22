# 时间隐式变量说明

addSubTime\#\#\# 隐式参数

## 1. 时间隐式参数

a. 支持如下格式：

```text
${YYYYMMDD}   
或者 
${yyyyMMddHHmm} / ${yyyyMMddHHmmss}    
又或者 
 ${yyyy-MM-dd HH:mm}  
跟
 ${yyyy-MM-dd HH:mm:ss}
```

b. 更多复杂的格式

```text
 可以使用 ${YYYY},${MM},${dd},${HH},${mm},${ss},${SSSSSS}"  
来组合,比如：  
'${YYYY}'-'${MM}'-'${dd}' '${HH}':'${mm}':'${ss}':${SSSSSS}
又或者  
${yyyy}${MM}${dd}/${yyyy}${MM}${dd}${HH}
${yyyy}-${MM}-${dd} ${HH}:${mm}:${ss}
```

c. 当然也向下兼容

```text
${YYYYMMDD} 以及 ${YYYYMMDDHH} ，${YYYYMMDDHHmm} 等格式
```

d. 时间加减表达式  
基于V5.0+版本  
必须使用如下表达式（小时只支持24小时）

```text
$addSubTime(${yyyy-1}${MM+13}${dd+31})
$addSubTime(${yyyy}${MM}${dd-1}) 
$addSubTime(${yyyy}${MM-1}${dd})
$addSubTime(${yyyy}${MM-1}${dd-1})
$addSubTime(${yyyy-1}${MM}${dd})
$addSubTime(${yyyy-1}${MM}${dd-1})
$addSubTime(${yyyy-1}${MM-1}${dd-1})
$addSubTime(${yyyy-1}${MM-1}${dd-1}${HH-2})
$addSubTime(${yyyy-1}${MM-1}${dd-1}${HH+1})
$addSubTime(${yyyy}-${MM}-${dd-1} ${HH}:${mm}:${ss})
$addSubTime(${yyyy}-${MM}-${dd+1} ${HH}:${mm}:${ss})
```

