# Day17 - 05.28

### 工作内容

编写建表SQL语句

```plsql
-- Create table
create table K9PS.T_PS_JOB_CODE_SPEC_REL
(
  id            varchar2(36) not null,
  postztype_id  VARCHAR2(20) not null,
  post_type     VARCHAR2(32) not null,
  org_code      VARCHAR2(16) not null,
  post_code     VARCHAR2(16) not null,
  job_spec_code VARCHAR2(100) not null,
  comments      VARCHAR2(100),
  status        VARCHAR2(8) not null,
  creater       VARCHAR2(8),
  creater_date  VARCHAR2(20),
  modifier      VARCHAR2(8),
  modifier_date VARCHAR2(20)
)
tablespace DS_K9PS01
  storage
  (
    initial 64K
    minextents 1
    maxextents unlimited
  );
-- Add comments to the table 
comment on table K9PS.T_PS_JOB_CODE_SPEC_REL
  is '岗位编码与规程编码映射关系表';
-- Add comments to the columns 
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.id
  is 'id';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.postztype_id
  is '岗位类型';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.post_type
  is '岗位序列';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.org_code
  is '机构编码';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.post_code
  is '岗位编码';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.job_spec_code
  is '岗位规程编码';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.comments
  is '备注';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.status
  is '有效标记';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.creater
  is '创建人';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.creater_date
  is '创建日期';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.modifier
  is '修改人';
comment on column K9PS.T_PS_JOB_CODE_SPEC_REL.modifier_date
  is '修改日期';
-- Create/Recreate primary, unique and foreign key constraints 
alter table K9PS.T_PS_JOB_CODE_SPEC_REL
  add constraint PK_T_PS_JOB_CODE_SPEC_REL primary key (ID);
```

将新表授权给管理角色`k9admin`

```plsql
-- Grant/Revoke object privileges 
grant select, insert, update, delete, references, alter, index, debug on K9PS.T_PS_JOB_CODE_SPEC_REL to k9admin;
```

创建同义词

```pls
create public synonym T_PS_JOB_CODE_SPEC_REL for K9PS.T_PS_JOB_CODE_SPEC_REL;
```

