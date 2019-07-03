# Day26 - 06.10

### 工作内容

编写员工登录日志表建表SQL

```plsql
-- Create table
create table K9ADMIN.T_PS_LOGIN_EMP_LOG
(
  emp_code       varchar2(100) not null,
  login_date     date not null,
  login_type     varchar2(8) not null,
  job_desc_time  number default 0 not null,
  job_spec_time  number default 0 not null,
  log_off_date   date,
  creater        varchar2(20),
  create_date    date,
  modifier       varchar2(20),
  modify_date    date
)
tablespace DS_K901
  pctfree 10
  initrans 1
  maxtrans 255
  storage
  (
    initial 64K
    minextents 1
    maxextents unlimited
  );
-- Add comments to the columns 
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.emp_code
  is '登录账号';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.login_date
  is '登录时间';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.login_type
  is '登录类型';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.job_desc_time
  is '浏览说明书时长';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.job_spec_time
  is '浏览规程时长';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.log_off_date
  is '账号退出时间';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.creater
  is '创建人';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.create_date
  is '创建时间';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.modifier
  is '修改人';
comment on column K9ADMIN.T_PS_LOGIN_EMP_LOG.modify_date
  is '修改时间';
-- Add comments to the table 
comment on table K9ADMIN.T_PS_LOGIN_EMP_LOG
  is '岗位规范员工登录日志表';
-- Create/Recreate primary, unique and foreign key constraints 
alter table K9ADMIN.T_PS_LOGIN_EMP_LOG
  add constraint PK_T_PS_LOGIN_EMP_LOG primary key (emp_code, login_date);
```

修改建表语句

- 将登录与规范查看设为同级
- 不再区分登录方式
- 增加有效标记
- 调整备注、账号、修改人、创建人等字段字符串长度

```plsql
-- Create table
create table T_PS_LOGIN_EMP_LOG
(
  emp_code       varchar2(32) not null,
  operation_type varchar2(8) not null,
  start_time     date not null,
  end_time       date,
  alive_flag     varchar2(8) not null,
  comments       varchar2(1024),
  creater        varchar2(32),
  create_date    date,
  modifier       varchar2(32),
  modify_date    date
)
tablespace DS_K9PS01
  pctfree 10
  initrans 1
  maxtrans 255
  storage
  (
    initial 64K
    minextents 1
    maxextents unlimited
  );
-- Add comments to the table 
comment on table T_PS_LOGIN_EMP_LOG
  is '岗位规范员工登录日志表';
-- Add comments to the columns 
comment on column T_PS_LOGIN_EMP_LOG.emp_code
  is '登录账号';
comment on column T_PS_LOGIN_EMP_LOG.operation_type
  is '操作类型（1.登录，2.说明书，3.规程，4.规范）';
comment on column T_PS_LOGIN_EMP_LOG.start_time
  is '开始时间';
comment on column T_PS_LOGIN_EMP_LOG.end_time
  is '结束时间';
comment on column T_PS_LOGIN_EMP_LOG.alive_flag
  is '有效标记（1.有效，0.无效）';
comment on column T_PS_LOGIN_EMP_LOG.comments
  is '备注';
comment on column T_PS_LOGIN_EMP_LOG.creater
  is '创建人';
comment on column T_PS_LOGIN_EMP_LOG.create_date
  is '创建时间';
comment on column T_PS_LOGIN_EMP_LOG.modifier
  is '修改人';
comment on column T_PS_LOGIN_EMP_LOG.modify_date
  is '修改时间';
-- Create/Recreate primary, unique and foreign key constraints 
alter table T_PS_LOGIN_EMP_LOG
  add constraint PK_T_PS_LOGIN_EMP_LOG primary key (EMP_CODE, START_TIME);
```



