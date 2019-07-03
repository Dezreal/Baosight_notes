# Day39 - 06.27

### 工作内容

补充存储过程`bdsplat_to_bd_emp_assignation`：人员变动，如果离职或者作废，需要取消相关的角色权限和用户

```plsql
    --人员失效，删除所有组织权限
    delete from iplat.t_es_userset_member i
     where i.member_type = 'TYP_ES00' --TODO 不确定
       and i.member_name in (select emp_code
                               from k9bd.t_bd_employee_base bd
                              where bd.status = 0);
```

补充存储过程`bdsplat_to_bd_org_base_info`：机构撤销对角色权限（用户）相关的取消

```plsql
    --机构失效，删除所有用户绑定
    delete from iplat.t_es_userset_member i
     where REGEXP_SUBSTR(i.set_name, '[^@]+', 1, 2, 'i') in
           (select bd.org_code
              from k9bd.t_bd_org_base_info bd
             where bd.use_status = 0);
    --机构失效，删除所有角色类型
    delete from iplat.t_es_userset i
     where REGEXP_SUBSTR(i.name, '[^@]+', 1, 2, 'i') in
           (select bd.org_code
              from k9bd.t_bd_org_base_info bd
             where bd.use_status = 0);
    --机构失效，删除所有机构角色
    delete from iplat.t_es_role i
     where i.authz_node_name in
           (select bd.org_code
              from k9bd.t_bd_org_base_info bd
             where bd.use_status = 0);
```

