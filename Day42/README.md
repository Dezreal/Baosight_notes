# Day42 - 07.02

### 工作内容

公共日志表序列

```plsql
-- Create sequence 
create sequence SEQ_T_HX_COMMON_LOG
minvalue 1
maxvalue 999999999999
start with 1
increment by 1
cache 10;
```

Oracle自治事务

```plsql
  --公共日志记录过程包，独立事务提交
  procedure p_common_log(v_log_type_code varchar2,
                         v_log_type_name varchar2,
                         v_log_note      varchar2,
                         v_log_detail    varchar2,
                         v_remark        varchar2,
                         v_note_id       varchar2,
                         v_user_id       varchar2) is
    PRAGMA AUTONOMOUS_TRANSACTION; -- 自治事务
  begin
    insert into t_hx_common_log
      (log_type_code,
       log_type_name,
       log_note,
       log_detail,
       start_time,
       end_time,
       remark,
       note_id,
       status,
       alive_flag,
       creater,
       create_date,
       modifier,
       modify_date)
    values
      (v_log_type_code,
       v_log_type_name,
       v_log_note,
       v_log_detail,
       current_timestamp,
       null,
       v_remark,
       v_note_id,
       null,
       '1',
       v_user_id,
       sysdate,
       null,
       null);
    commit;

  end;
```

Oracle 自制事务是指的存储过程和函数可以自己处理内部事务不受外部事务的影响，用`pragma autonomous_transaction`来声明，要创建一个自治事务，必须在匿名块的最高层或者存储过程、函数、数据包或触发的定义部分中，使用PL/SQL中的`PRAGMA AUTONOMOUS_TRANSACTION`语句。在这样的模块或过程中执行的SQL语句都是自治的。

更新`main`过程

```plsql
  procedure main is
  
  begin
  
    --执行同步
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_base_info;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_relation;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_organization;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_employee_base;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_emp_assignation;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_position;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_job_type;
    commit;
    --记录操作日志
    p_hx_log.p_common_log('K9_BDS_ASYN',
                          'eHR数据同步',
                          to_char(sysdate, 'YYYYMMDD'),
                          '由bdsplat向k9bd同步员工、岗位、机构信息',
                          '更新k9bd相关数据表',
                          k9admin.seq_t_hx_common_log.nextval(),
                          'admin');
  end main;
```

