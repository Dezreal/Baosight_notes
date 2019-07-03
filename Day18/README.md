# Day18 - 05.29

### 工作内容

修改建表语句：

修改字段

将`status        VARCHAR2(8)`改为`ALIVE_FLAG           VARCHAR2(16)`

修改数据表基本属性为

```plsql
tablespace DS_K9PS01
  pctfree 10
  initrans 1
  maxtrans 255
  storage
  (
    initial 64K
    next 1M
    minextents 1
    maxextents unlimited
  );
```

修改授权语句为

```plsql
grant all on K9PS.T_PS_JOB_CODE_SPEC_REL to k9admin;
```

修改同义词语句为

```plsql
create public synonym k9admin.T_PS_JOB_CODE_SPEC_REL for K9PS.T_PS_JOB_CODE_SPEC_REL;
```

请部门前辈以system身份执行修改后的建表语句，完成数据表的创建