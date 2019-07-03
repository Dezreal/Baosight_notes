# Day29

### 工作内容

了解ixbus数据导入方式

学习Oracle存储过程语法，学习Oracle的package组织结构使用方法

package:

```plsql
create or replace package P_XH_TO_K9_BDS_ASYN is

  -- Author  : 丁畅
  -- Created : 2019/6/13 11:05:10
  -- Purpose : 数据同步
  
  -- Public type declarations
  --type <TypeName> is <Datatype>;
  
  -- Public constant declarations
  --<ConstantName> constant <Datatype> := <Value>;

  -- Public variable declarations
  --<VariableName> <Datatype>;

  -- Public function and procedure declarations
  procedure P_XH_TO_K9_BDS_ASYN_1 ;

end P_XH_TO_K9_BDS_ASYN;
```

package body:

```plsql
create or replace package body P_XH_TO_K9_BDS_ASYN is

  -- Private type declarations
  --type <TypeName> is <Datatype>;

  -- Private constant declarations
  --<ConstantName> constant <Datatype> := <Value>;

  -- Private variable declarations
  --<VariableName> <Datatype>;

  -- Function and procedure implementations
  procedure P_XH_TO_K9_BDS_ASYN_1 is
    CURSOR cur is
      select *
        from bdsplat.t_bds_org_base_info
       where org_code like 'BSZG%'
         and rownum < 10;
    n int;
  begin
    FOR cr in cur LOOP
      BEGIN
        select count(*)
          into n
          from k9bd.t_bd_org_base_info b
         where b.id = cr.org_id;
        if n = 0 then
          insert into k9bd.t_bd_org_base_info
            (id,
             org_code,
             org_name,
             org_full_name,
             org_type_id,
             org_type_name,
             from_date,
             to_date,
             use_status,
             create_date)
          values
            (cr.org_id,
             cr.org_code,
             nvl(cr.org_name,
                 decode(cr.org_type_id,
                        1,
                        '虚拟部门',
                        5,
                        '虚拟作业区',
                        7,
                        '虚拟班组',
                        cr.org_name)),
             cr.org_full_name,
             cr.org_type_id,
             cr.org_type_name,
             cr.from_date,
             cr.to_date,
             1,
             to_char(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
        else
          update k9bd.t_bd_org_base_info
             set org_code      = cr.org_code,
                 org_name      = nvl(cr.org_name,
                                     decode(cr.org_type_id,
                                            1,
                                            '虚拟部门',
                                            5,
                                            '虚拟作业区',
                                            7,
                                            '虚拟班组',
                                            cr.org_name)),
                 org_full_name = cr.org_full_name,
                 org_type_id   = cr.org_type_id,
                 org_type_name = cr.org_type_name,
                 from_date     = cr.from_date,
                 to_date       = cr.to_date,
                 use_status    = 1,
                 modify_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
           where id = cr.org_id;
        END if;
      END;
    END LOOP;
  end P_XH_TO_K9_BDS_ASYN_1;

begin
  -- Initialization
  DBMS_OUTPUT.ENABLE(buffer_size => null);
  --<Statement>;
end P_XH_TO_K9_BDS_ASYN;
create or replace package body P_XH_TO_K9_BDS_ASYN is

  -- Private type declarations
  --type <TypeName> is <Datatype>;

  -- Private constant declarations
  --<ConstantName> constant <Datatype> := <Value>;

  -- Private variable declarations
  --<VariableName> <Datatype>;

  -- Function and procedure implementations
  procedure P_XH_TO_K9_BDS_ASYN_1 is
    CURSOR cur is
      select *
        from bdsplat.t_bds_org_base_info
       where org_code like 'BSZG%'
         and rownum < 10;
    n int;
  begin
    FOR cr in cur LOOP
      BEGIN
        select count(*)
          into n
          from k9bd.t_bd_org_base_info b
         where b.id = cr.org_id;
        if n = 0 then
          insert into k9bd.t_bd_org_base_info
            (id,
             org_code,
             org_name,
             org_full_name,
             org_type_id,
             org_type_name,
             from_date,
             to_date,
             use_status,
             create_date)
          values
            (cr.org_id,
             cr.org_code,
             nvl(cr.org_name,
                 decode(cr.org_type_id,
                        1,
                        '虚拟部门',
                        5,
                        '虚拟作业区',
                        7,
                        '虚拟班组',
                        cr.org_name)),
             cr.org_full_name,
             cr.org_type_id,
             cr.org_type_name,
             cr.from_date,
             cr.to_date,
             1,
             to_char(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
        else
          update k9bd.t_bd_org_base_info
             set org_code      = cr.org_code,
                 org_name      = cr.org_name,
                 org_full_name = cr.org_full_name,
                 org_type_id   = cr.org_type_id,
                 org_type_name = cr.org_type_name,
                 from_date     = cr.from_date,
                 to_date       = cr.to_date,
                 use_status    = 1,
                 modify_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
           where id = cr.org_id;
        END if;
      END;
    END LOOP;
  end P_XH_TO_K9_BDS_ASYN_1;

begin
  -- Initialization
  DBMS_OUTPUT.ENABLE(buffer_size => null);
  --<Statement>;
end P_XH_TO_K9_BDS_ASYN;

```



学习数据表全量同步方式及相关存储过程的编写