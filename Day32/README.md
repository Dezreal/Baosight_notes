# Day32

### 工作内容

继续设计存储过程

新的序列：

```plsql
-- Create sequence 
create sequence SEQ_BD_POSITION
minvalue 1
start with 1
increment by 1
cache 10;
```

存储过程

```plsql
procedure bdsplat_to_bd_position is
    cursor cur is
      select *
        FROM T_BDS_POST_BASE_INFO t
       where t.alive_flag = 1
         and t.org_code in (select o.org_code
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
    currow k9bd.t_bd_position%rowtype;
  begin
    for cr in cur loop
      select count(1)
        into n
        from k9bd.t_bd_position
       where post_code = cr.post_code;
      if n = 0 then
        --新数据
        insert into k9bd.t_bd_position
          (id,
           post_code,
           post_name,
           job_type_code,
           org_code,
           status,
           create_date)
        values
          (SEQ_BD_POSITION.NEXTVAL,
           cr.POST_code,
           cr.post_name,
           cr.job_type_code,
           cr.org_code,
           1,
           
           to_char(sysdate(), 'YYYY-MM-DD HH24:MI:SS'));
      else
        --旧数据
        select *
          into currow
          from k9bd.t_Bd_Position
         where post_code = cr.post_code;
        if currow.post_name != cr.post_name or
           currow.job_type_code != cr.job_type_code or
           currow.org_code != cr.org_code then
          --有变动
          update k9bd.t_Bd_Position
             set post_name     = cr.post_name,
                 job_type_code = cr.job_type_code,
                 org_code      = cr.org_code,
                 
                 status      = 1,
                 modify_date = to_char(sysdate(), 'YYYY-MM-DD HH24:MI:SS')
           where post_code = currow.post_code;
        end if;
      end if;
    end loop;
    --反查无效的或被删除的数据
    update k9bd.t_Bd_Position bd
       set bd.status      = 0,
           bd.modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where bd.post_code not in
           (select t.post_code
              FROM T_BDS_POST_BASE_INFO t
             where t.alive_flag = 1
               and t.org_code in
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
  end bdsplat_to_bd_position;
```

```plsql
  procedure bdsplat_to_bd_job_type is
    cursor cur is
      select * from bdsplat.t_bds_post_job_type e where alive_flag = 1; -- TODO 有效性检查
  begin
    delete k9bd.t_bd_job_type; --清表
    for cr in cur loop
      insert into k9bd.t_bd_job_type
        (job_type_code,
         job_type_name,
         super_job_type_code,
         disply_order,
         crteate_date,
         modify_date)
      values
        (cr.job_type_code,
         cr.job_type_name,
         cr.super_job_type_code,
         cr.disply_order,
         TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'),
         TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
    end loop; 
  end bdsplat_to_bd_job_type;
```

`bdsplat_to_bd_job_type`思路：

目标表没有有效标记字段，因此先将目标表清空，再遍历源表**有效**数据并插入，`crteate_date`, `modify_date`均设为当前时间。



以相同思路修改之前的三个存储过程：

```plsql
  p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_base_info;
  p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_relation;
  p_xh_to_k9_bds_asyn.bdsplat_to_bd_organization;
```



