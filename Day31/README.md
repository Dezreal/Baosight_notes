# Day31

### 工作内容

学习并设计Oracle的sequence

```plsql
-- Create sequence 
create sequence SEQ_BD_EMPLOYEE_BASE
minvalue 1
maxvalue 9999999999999999999999999999
start with 5500
increment by 1
cache 10;

-- Create sequence 
create sequence SEQ_BD_EMP_ASSIGNATION
minvalue 1
maxvalue 9999999999999999999999999999
start with 1
increment by 1
cache 10;
```

重新整理思路，继续编写存储过程

```plsql
  procedure bdsplat_to_bd_employee_base is
    cursor cur is
      select *
        from t_bds_emp_base_info e
       where e.alive_flag = 1
         and e.change_type != '0'
         and e.org_code in (select o.org_code
                              from t_bds_org_base_info o
                             where o.alive_flag = 1
                               and o.dimension_id = 1
                               and o.is_alive_flag = 1
                               and o.to_date is null
                               and (o.use_status is null or o.use_status = 1)
                             start with o.org_code = 'BSZG'
                                    and o.alive_flag = 1
                            connect by prior o.org_code = o.up_org_code
                                   and o.alive_flag = 1);
    n      int;
    currow k9bd.t_bd_employee_base%rowtype;
  begin
    for cr in cur loop
      select count(1)
        into n
        from k9bd.t_bd_employee_base
       where k9bd.t_bd_employee_base.emp_code = cr.emp_code;
      if n = 0 then
        -- 新数据
        insert into k9bd.t_bd_employee_base
          (id, emp_code, emp_name, gender, birthday, status, create_date)
        values
          (k9admin.seq_bd_employee_base.nextval,
           cr.emp_code,
           cr.emp_name,
           cr.gender,
           TO_CHAR(cr.birthday, 'YYYY-MM-DD'),
           1,
           TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
      
      else
        --旧数据
        select *
          into currow
          from k9bd.t_bd_employee_base
         where emp_code = cr.emp_code;
        IF currow.emp_name <> cr.emp_name or currow.gender <> cr.gender or
           currow.birthday <> TO_CHAR(cr.birthday, 'YYYY-MM-DD') or
           currow.status = 0 then
          --有变动
          update k9bd.t_bd_employee_base
             set emp_name    = cr.emp_name,
                 gender      = cr.gender,
                 birthday    = TO_CHAR(cr.birthday, 'YYYY-MM-DD'),
                 status      = 1,
                 modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
           where id = currow.id;
        end if;
      end if;
    end loop;
    --反查无效的或被删除的数据
    update k9bd.t_bd_employee_base bd
       set bd.status      = 0,
           bd.modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where bd.emp_code not in
           (select e.emp_code
              from t_bds_emp_base_info e
             where e.alive_flag = 1
               and e.change_type != '0'
               and e.org_code in
                   (select o.org_code
                      from t_bds_org_base_info o
                     where o.alive_flag = 1
                       and o.dimension_id = 1
                       and o.is_alive_flag = 1
                       and o.to_date is null
                       and (o.use_status is null or o.use_status = 1)
                     start with o.org_code = 'BSZG'
                            and o.alive_flag = 1
                    connect by prior o.org_code = o.up_org_code
                           and o.alive_flag = 1));
  end bdsplat_to_bd_employee_base;
```

```plsql
  procedure bdsplat_to_bd_emp_assignation is
    cursor cur is
      select *
        from t_bds_emp_base_info e
       where e.alive_flag = 1
         and e.change_type != '0'
         and e.org_code in (select o.org_code
                              from t_bds_org_base_info o
                             where o.alive_flag = 1
                               and o.dimension_id = 1
                               and o.is_alive_flag = 1
                               and o.to_date is null
                               and (o.use_status is null or o.use_status = 1)
                             start with o.org_code = 'BSZG'
                                    and o.alive_flag = 1
                            connect by prior o.org_code = o.up_org_code
                                   and o.alive_flag = 1);
    n      int;
    currow k9bd.t_bd_emp_assignation%rowtype;
  begin
    for cr in cur loop
      select count(1)
        into n
        from k9bd.t_bd_emp_assignation
       where emp_code = cr.emp_code;
      if n = 0 then
        --新数据
        insert into k9bd.t_bd_emp_assignation

          (id,
           emp_code,
           org_code,
           post_code,
           from_date,
           to_date,
           status,
           create_date)
        values
          (SEQ_BD_EMP_ASSIGNATION.NEXTVAL,
           cr.emp_code,
           cr.org_code,
           cr.post_code,
           to_char(cr.from_date, 'YYYY-MM-DD'),
           to_char(cr.to_date, 'YYYY-MM-DD'),
           1,
           to_char(sysdate(), 'YYYY-MM-DD HH24:MI:SS'));
      else
        --旧数据
        select *
          into currow
          from k9bd.t_bd_emp_assignation
         where emp_code = cr.emp_code;
        if currow.org_code != cr.org_code or
           currow.post_code != cr.post_code or
           currow.from_date != TO_CHAR(cr.from_date, 'YYYY-MM-DD') or
           currow.to_date != TO_CHAR(cr.to_date, 'YYYY-MM-DD') then
          --有变动
          update k9bd.t_bd_emp_assignation
             set org_code    = cr.org_code,
                 post_code   = cr.post_code,
                 from_date   = to_char(cr.from_date, 'YYYY-MM-DD'),
                 to_date     = to_char(cr.to_date, 'YYYY-MM-DD'),
                 status      = 1,
                 modify_date = to_char(sysdate(), 'YYYY-MM-DD HH24:MI:SS')
           where emp_code = currow.emp_code;
        end if;
      end if;
    end loop;
    --反查无效的或被删除的数据
    update k9bd.t_bd_emp_assignation bd
       set bd.status      = 0,
           bd.modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where bd.emp_code not in
           (select e.emp_code
              from t_bds_emp_base_info e
             where e.alive_flag = 1
               and e.change_type != '0'
               and e.org_code in
                   (select o.org_code
                      from t_bds_org_base_info o
                     where o.alive_flag = 1
                       and o.dimension_id = 1
                       and o.is_alive_flag = 1
                       and o.to_date is null
                       and (o.use_status is null or o.use_status = 1)
                     start with o.org_code = 'BSZG'
                            and o.alive_flag = 1
                    connect by prior o.org_code = o.up_org_code
                           and o.alive_flag = 1));
  end bdsplat_to_bd_emp_assignation;
```

主要思路：

- 逻辑
  - 遍历源表的**有效**数据：
    - 目标表没有的，插入，设`status`为1, `create_date`为当前时间
    - 目标表已有的，对比各字段，若有不同，更新所有此前用于对比的字段，设`modify_date`为当前时间，`status`保持为1不变
  - 遍历目标表的数据，在源表的**有效**数据中查询不到的，设`status`为0，`modify_date`为当前时间

- 格式
  - `from_date`,`to_date`等同步字段格式为`'YYYY-MM-DD'`,`create_date`,`modify_date`等标记字段格式为`'YYYY-MM-DD HH24:MI:SS'`
