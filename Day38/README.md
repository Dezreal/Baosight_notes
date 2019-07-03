# Day38 - 06.26

### 工作内容

#### SQL

分析平台权限管理相关逻辑，分析整理系统授权配置相关表逻辑

```plsql
--用户信息表
SELECT *
  FROM iplat.TES01 a
  left join iplat.T_ES_USER b
    on a.USER_ID = b.NAME
 WHERE 1 = 1
   AND a.USER_ID like ('%017470%')
   AND a.USER_CLASS like ('%[k9]%')
 ORDER BY a.USER_ID asc;

--查询某一员工所拥有的角色权限
select *
  from iplat.t_es_userset
 where name in (select set_name
                  from iplat.t_es_userset_member
                 where 1 = 1
                   AND member_name = '179116'
                   AND member_type = 'TYP_ES00' --
                )
   AND userset_type = 'Role' --
 order by sort_index DESC, name ASC;

--角色类型配置表
select * from iplat.t_es_userset;

--角色组织权限信息集跟用户表绑定
select * from iplat.t_es_userset_member;

--管理员帐户
select *
  from iplat.t_es_user
 where name in (select member_name
                  from iplat.t_es_userset_member
                 where 1 = 1
                   AND set_name = '_$SYSTEM_ADMIN'
                   AND member_type = 'TYP_ES00')
 order by sort_index DESC, name ASC;

--组织机构授权树
select * from iplat.t_es_authz_tree;

--组织机构跟角色绑定配置表
select * from iplat.t_es_role;

--机构角色配置（把机构跟角色类型进行绑定）
select *
  from iplat.t_es_userset
 where name in (select role_name
                  from iplat.t_es_role
                 where authz_node_name = 'BSZGAA00')
   AND userset_type = 'Role'
   AND ext3 = 'k9'
 order by name ASC, sort_index DESC;

--查询某一机构角色下有哪些用户在使用
select *
  from iplat.t_es_user
 where name in (select member_name
                  from iplat.t_es_userset_member
                 where 1 = 1
                   AND set_name = '_ROLE:BZ_CLASSIFY@BSZG' --
                   AND member_type = 'TYP_ES00')
 order by name ASC, sort_index DESC;

--页面注册表
SELECT *
  FROM iplat.tedfa00
 where 1 = 1
   AND form_ename = 'PS1014'
 ORDER BY form_ename asc;

--页面菜单注册表
select * from iplat.tedpi10;

--按钮注册表
select * from iplat.tedfa01;

--查询某一角色下的菜单定义 
select subject_name as "角色类型",
       subject_type,
       object_name  as "页面菜单号",
       object_type,
       opt
  from iplat.t_es_authorization
 where 1 = 1
   AND subject_name = 'BZ_JB_ISSUER'
   AND subject_type = 'TYP_ES01'
   AND opt = 'OPT_ES00'
 order by subject_name, subject_type, object_name, object_type, opt;
```

#### 主存储过程

```plsql
  procedure main is
  
  begin
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_base_info;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_org_relation;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_organization;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_employee_base;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_emp_assignation;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_position;
    p_xh_to_k9_bds_asyn.bdsplat_to_bd_job_type;
    commit;
  end main;
```

#### DBMS_Jobs

 学习Oracle定时任务DBMS_Jobs

```plsql
begin
  sys.dbms_job.submit(job => :job,
                      what => 'p_xh_to_k9_bds_asyn.main;',
                      interval => 'TRUNC(sysdate+ 1)  +1/ (24)');
  commit;
end;
/
```

#### interval

间隔/interval是指上一次执行结束到下一次开始执行的时间间隔，当interval设置为null时，该job执行结束后，就被从队列中删除。假如我们需要该job周期性地执行，则要用‘sysdate＋m’表示。

1. 
   每分钟执行
   `Interval => TRUNC(sysdate,'mi') + 1/ (24*60)`
   每小时执行
   `Interval => TRUNC(sysdate,'hh') + 1/ (24)`
2. 每天定时执行
   例如：每天的凌晨1点执行
   `Interval => TRUNC(sysdate+ 1)  +1/ (24)`
3. 每周定时执行
   例如：每周一凌晨1点执行
   `Interval => TRUNC(next_day(sysdate,'星期一'))+1/24`
4. 每月定时执行
   例如：每月1日凌晨1点执行
   `Interval =>TRUNC(LAST_DAY(SYSDATE))+1+1/24`
5. 每季度定时执行
   例如每季度的第一天凌晨1点执行
   `Interval => TRUNC(ADD_MONTHS(SYSDATE,3),'Q') + 1/24`
6. 每半年定时执行
   例如：每年7月1日和1月1日凌晨1点
   `Interval => ADD_MONTHS(trunc(sysdate,'yyyy'),6)+1/24`
7. 每年定时执行
   例如：每年1月1日凌晨1点执行
   `Interval =>ADD_MONTHS(trunc(sysdate,'yyyy'),12)+1/24`