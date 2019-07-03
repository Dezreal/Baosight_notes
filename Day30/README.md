# Day30

### 工作内容

完成测试环境ixbus数据同步

完成package "P_XH_TO_K9_BDS_ASYN"的三个存储过程

```plsql
create or replace package body P_XH_TO_K9_BDS_ASYN is

  -- Private type declarations
  --type <TypeName> is <Datatype>;

  -- Private constant declarations
  --<ConstantName> constant <Datatype> := <Value>;

  -- Private variable declarations
  --<VariableName> <Datatype>;

  -- Function and procedure implementations
  procedure bdsplat_to_bd_org_base_info is
    CURSOR cur is
      select *
        from bdsplat.t_bds_org_base_info o
       where o.dimension_id = 1
      
       start with o.org_code = 'BSZG'
      
      connect by prior o.org_code = o.up_org_code;
    n int;
  
  begin
    FOR cr in cur LOOP
      BEGIN
        --遍历
      
        if cr.alive_flag = 1 and cr.is_alive_flag = 1 and
           cr.to_date is null and
           (cr.use_status is null or cr.use_status = 1) then
          --有效
          select count(*)
            into n
            from k9bd.t_bd_org_base_info b
           where b.id = cr.org_id;
          if n = 0 then
            --新数据
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
               to_char(cr.from_date, 'yyyy-mm-dd HH24:mi:ss'),
               to_char(cr.to_date, 'yyyy-mm-dd HH24:mi:ss'),
               1,
               to_char(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
          else
            --旧数据
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
                   from_date     = to_char(cr.from_date,
                                           'yyyy-mm-dd HH24:mi:ss'),
                   to_date       = to_char(cr.to_date,
                                           'yyyy-mm-dd HH24:mi:ss'),
                   use_status    = 1,
                   modify_date   = TO_CHAR(SYSDATE(),
                                           'YYYY-MM-DD HH24:MI:SS')
             where id = cr.org_id;
          END if;
        else
          --失效(失效仅更新，不做插入)
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
                 from_date     = to_char(cr.from_date,
                                         'yyyy-mm-dd HH24:mi:ss'),
                 to_date       = to_char(nvl(cr.to_date, sysdate()),
                                         'yyyy-mm-dd HH24:mi:ss'),
                 use_status    = 1,
                 modify_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
           where id = cr.org_id;
        end if;
      END;
    END LOOP;
  
    update K9BD.t_bd_org_base_info k --反查
    
       SET k.to_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'),
           modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where not exists (select 1
              from bdsplat.t_bds_org_base_info o
             where k.id = o.org_id);
  
  end bdsplat_to_bd_org_base_info;

  procedure bdsplat_to_bd_org_relation is
    CURSOR cur is
      select *
        from bdsplat.t_bds_org_base_info o
       where o.dimension_id = 1
      
       start with o.org_code = 'BSZG'
      
      connect by prior o.org_code = o.up_org_code;
    n int;
  begin
    FOR cr in cur LOOP
      BEGIN
        --遍历
      
        if cr.alive_flag = 1 and cr.is_alive_flag = 1 and
           cr.to_date is null and
           (cr.use_status is null or cr.use_status = 1) then
          --有效
          select count(*)
            into n
            from k9bd.t_bd_org_relation b
           where b.id = cr.org_id;
          if n = 0 then
            --新数据
            insert into k9bd.t_bd_org_relation
              (id,
               org_code,
               TARGET_ORG_CODE,
               Org_Level,
               from_date,
               to_date,
               status,
               create_date)
            values
              (cr.org_id,
               cr.org_code,
               CR.UP_ORG_CODE,
               (select level - 1
                  from bdsplat.t_bds_org_base_info p
                 where p.record_id = cr.record_id
                
                 start with p.org_code = 'BSZG'
                        and p.alive_flag = 1
                connect by prior p.org_code = p.up_org_code
                       and p.alive_flag = 1),
               to_char(cr.from_date, 'yyyy-mm-dd HH24:mi:ss'),
               to_char(cr.to_date, 'yyyy-mm-dd HH24:mi:ss'),
               1,
               to_char(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
          else
            --旧数据
            update k9bd.t_bd_org_relation
               set org_code        = cr.org_code,
                   target_org_code = cr.up_org_code,
                   org_level      =
                   (select level - 1
                      from bdsplat.t_bds_org_base_info p
                     where p.record_id = cr.record_id
                    
                     start with p.org_code = 'BSZG'
                            and p.alive_flag = 1
                    connect by prior p.org_code = p.up_org_code
                           and p.alive_flag = 1),
                   from_date       = to_char(cr.from_date,
                                             'yyyy-mm-dd HH24:mi:ss'),
                   to_date         = to_char(cr.to_date,
                                             'yyyy-mm-dd HH24:mi:ss'),
                   status          = 1,
                   modify_date     = TO_CHAR(SYSDATE(),
                                             'YYYY-MM-DD HH24:MI:SS')
             where id = cr.org_id;
          END if;
        else
          --失效
          update k9bd.t_bd_org_relation
             set org_code        = cr.org_code,
                 target_org_code = cr.up_org_code,
                 org_level      =
                 (select level - 1
                    from bdsplat.t_bds_org_base_info p
                   where p.record_id = cr.record_id
                  
                   start with p.org_code = 'BSZG'
                          and p.alive_flag = 1
                  connect by prior p.org_code = p.up_org_code
                         and p.alive_flag = 1),
                 from_date       = to_char(cr.from_date,
                                           'yyyy-mm-dd HH24:mi:ss'),
                 to_date         = to_char(nvl(cr.to_date, sysdate()),
                                           'yyyy-mm-dd HH24:mi:ss'),
                 status          = 1,
                 modify_date     = TO_CHAR(SYSDATE(),
                                           'YYYY-MM-DD HH24:MI:SS')
           where id = cr.org_id;
        end if;
      END;
    END LOOP;
  
    update K9BD.t_bd_org_relation k --反查
    
       SET k.to_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'),
           modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where not exists (select 1
              from bdsplat.t_bds_org_base_info o
             where k.id = o.org_id);
  end bdsplat_to_bd_org_relation;

  procedure bdsplat_to_bd_organization is
    CURSOR cur is
      select *
        from bdsplat.t_bds_org_base_info o
       where o.dimension_id = 1
      
       start with o.org_code = 'BSZG'
      
      connect by prior o.org_code = o.up_org_code;
    n int;
  begin
    FOR cr in cur LOOP
      BEGIN
        --遍历
      
        if cr.alive_flag = 1 and cr.is_alive_flag = 1 and
           cr.to_date is null and
           (cr.use_status is null or cr.use_status = 1) then
          --有效
          select count(*)
            into n
            from k9bd.t_bd_organization b
           where b.id = cr.org_id;
          if n = 0 then
            --新数据
            insert into k9bd.t_bd_organization
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
               to_char(cr.from_date, 'yyyy-mm-dd HH24:mi:ss'),
               to_char(cr.to_date, 'yyyy-mm-dd HH24:mi:ss'),
               1,
               to_char(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'));
          else
            --旧数据
            update k9bd.t_bd_organization
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
                   from_date     = to_char(cr.from_date,
                                           'yyyy-mm-dd HH24:mi:ss'),
                   to_date       = to_char(cr.to_date,
                                           'yyyy-mm-dd HH24:mi:ss'),
                   use_status    = 1,
                   modify_date   = TO_CHAR(SYSDATE(),
                                           'YYYY-MM-DD HH24:MI:SS')
             where id = cr.org_id;
          END if;
        else
          --失效
          update k9bd.t_bd_organization
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
                 from_date     = to_char(cr.from_date,
                                         'yyyy-mm-dd HH24:mi:ss'),
                 to_date       = to_char(nvl(cr.to_date, sysdate()),
                                         'yyyy-mm-dd HH24:mi:ss'),
                 use_status    = 1,
                 modify_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
           where id = cr.org_id;
        end if;
      END;
    END LOOP;
  
    update K9BD.t_bd_organization k --反查
    
       SET k.to_date   = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS'),
           modify_date = TO_CHAR(SYSDATE(), 'YYYY-MM-DD HH24:MI:SS')
     where not exists (select 1
              from bdsplat.t_bds_org_base_info o
             where k.id = o.org_id);
  
  end bdsplat_to_bd_organization;

begin
  -- Initialization
  DBMS_OUTPUT.ENABLE(buffer_size => null);
  --<Statement>;
end P_XH_TO_K9_BDS_ASYN;

```

