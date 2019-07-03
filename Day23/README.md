# Day23

### 工作内容

继续分析系统代码

#### PS1005 初始化逻辑

- 查：


`tpsjobspecification.query`

```xml
		select
			ID "id", 
                                  POST_CODE "postCode", 
                                  POST_name "postName", 
                                  POST_Type "postType", 
                                  STATUS "status", 
                                  VERSION "version", 
                                  UPDATE_TIME "updateTime", 
                                  CREATE_DATE "createDate", 
                                  MODIFY_DATE "modifyDate"
		from  T_PS_JOB_SPECIFICATION t
```



条件：

岗位规范编码postCode = `postCode`1至12位

规范类型postType = 2

- 如果为空

`jobSpecification.init(postCode.substring(1, 13));`

param.jobTypeCode = C / J / G

