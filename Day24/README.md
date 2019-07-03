# Day24

### 工作内容

修改PS0051查询SQL

```plsql
		select
			ID "id", 
            POSTZTYPE_ID "postztypeId", 
            POST_TYPE "postType", 
            (select bdsplat.fun_hrog_org_fullname(t.org_code) from dual) as "orgCode",
            (select post_name from K9BD.T_BD_POSITION p where p.post_code = t.post_code) as "postCode", 
            JOB_SPEC_CODE "jobSpecCode", 
            COMMENTS "comments", 
            ALIVE_FLAG "aliveFlag", 
            CREATER "creater", 
            CREATE_DATE "createDate", 
            MODIFIER "modifier", 
            MODIFY_DATE "modifyDate"
		from  T_PS_JOB_CODE_SPEC_REL t
```

调整PS0051界面

学习用户信息表，登录信息表设计规范

学习MD5加密在信息表中的实际应用

学习Oracle tablespaces

创建登录日志表'k9admin.T_PS_LOGIN_EMP_LOG'