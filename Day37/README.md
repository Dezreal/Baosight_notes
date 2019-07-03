# Day37 - 06.25

### 工作内容

#### 概念

异常，程序运行过程中发生异常。异常是程序设计语言提供的一种功能，用来增强程序的健壮性和容错性

```plsql
   declare
       --声明变量
   begin
       --业务逻辑
   exception
       --处理异常
       when 异常1 then
         ...
       when 异常2 then
         ...
       when others then
         ...处理其它异常
   end;
```

#### 预定义异常

| 错误号   | 名称                   | 说明               |
| -------- | ---------------------- | ------------------ |
| ORA-0001 | Dup_val_on_index       | 破坏唯一性限制     |
| ORA-0005 | Timeout_on_resource    | 发生超时           |
| ORA-0061 | Transaction-backed-out | 由于死锁事务被取消 |
| ORA-1001 | Invalid-CURSOR         | 使用无效的游标     |
| ORA-1012 | Not-logged-on          | 没有连接到Oracle   |
| ORA-1403 | No_data_found          | 查询语句没有数据   |
| ORA-1422 | Too_many_rows          | 查询语句返回多行   |
| ORA-6501 | Program-error          | 内部错误           |
| ORA-6511 | CURSOR-already-OPEN    | 游标已经打开       |
| ORA-6530 | Access-INTO-null       | 访问Null的属性     |

#### 自定义异常

```plsql
    --自定义异常:  
    
     --声明异常
    异常名  exception;
    --抛出自定义的异常
    raise 异常名;   
```

#### RAISE_APPLICATION_ERROR函数

`RAISE_APPLICATION_ERROR`能将应用程序专有的错误从服务器端转达到客户端应用程序

`RAISE_APPLICATION_ERROR`的声明：

`PROCEDURE RAISE_APPLICATION_ERROR( error_number_in IN NUMBER, error_msg_in IN VARCHAR2);`

`error_number_in`取值在-20000到-20999 之间，这样就不会与 ORACLE 的任何错误代码发生冲突；`error_msg_in`的长度不能超过 2k，否则截取 2k