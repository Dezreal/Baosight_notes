# Day08

##### DMCM03.jsp

```jsp
<!DOCTYPE html>
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib prefix="EF" tagdir="/WEB-INF/tags/EF" %>

<c:set var="ctx" value="${pageContext.request.contextPath}"/>

<EF:EFPage title="使用EFPage模版的KendoUI示例">
    <EF:EFRegion id="inqu" title="查询条件">
        <div class="row">
            <EF:EFInput ename="inqu_status-0-product_id" cname="产品代号"/>
            <EF:EFInput ename="inqu_status-0-company_name" cname="公司名称"/>
        </div>
    </EF:EFRegion>
    <EF:EFRegion id="result" title="记录集" >
        <EF:EFGrid blockId="result" />
    </EF:EFRegion>
</EF:EFPage>
```

##### DMCM03.js
```javascript
$(function () {
    $("#QUERY").on("click", function (e) {
        resultGrid.dataSource.page(1);
    });
});
```
##### ServiceDMCM03.java
```java
package com.baosight.iplatdemo.dm.cm.service;

import com.baosight.iplat4j.common.constant.EPResource; 
import com.baosight.iplat4j.core.ei.EiConstant;
import com.baosight.iplat4j.core.ei.EiInfo;
import com.baosight.iplat4j.core.log.Logger;
import com.baosight.iplat4j.core.log.LoggerFactory;
import com.baosight.iplat4j.core.resource.I18nMessages;

import java.util.List;
import java.util.Map;

import com.baosight.iplatdemo.common.dm.domain.Tdmcm01;
import com.baosight.iplat4j.core.service.impl.ServiceEPBase;
import com.baosight.iplat4j.core.util.DateUtils;
import com.baosight.iplat4j.core.util.ExceptionUtil;
import com.baosight.iplat4j.core.web.threadlocal.UserSession;

/**
 * 单表增删改查示例
 * 1. ServiceEPBase有默认的query delete update insert的实现
 * 2. 批量操作
 */
public class ServiceDMCM01 extends ServiceEPBase {
	//创建logger
    private static final Logger logger = LoggerFactory.getLogger(ServiceDMCM01.class);

    //初始化加载逻辑
    public EiInfo initLoad(EiInfo inInfo) { 
        return query(inInfo); 
    }
    
    //查询
    @Override
    public EiInfo query(EiInfo inInfo) {
        EiInfo outInfo = super.query(inInfo, "tdmcm01.query", new Tdmcm01()); 
        return outInfo;
    } 

    //新增方法，在复杂业务中建议使用循环遍历数据，在循环中使用dao.insert新增，而不是使用super.insert
    public EiInfo insert(EiInfo inInfo) { 
    	EiInfo outInfo = super.insert(inInfo, "tdmcm01.insert");
        return outInfo;
    }
    
    //更新方法，在复杂业务中建议使用循环遍历数据，在循环中使用dao.update修改，而不是使用super.update
    public EiInfo update(EiInfo inInfo) {
    	EiInfo outInfo = super.update(inInfo, "tdmcm01.update");
        return outInfo;
    }

    //删除，没有特殊操作调用super.delete即可
    public EiInfo delete(EiInfo inInfo) {
        EiInfo outInfo = super.delete(inInfo, "tdmcm01.delete");
        return outInfo;
    }
}

```

