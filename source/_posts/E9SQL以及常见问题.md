---
title: E9SQL以及常见问题
date: 2026-06-29 16:00:00
permalink: /2026/06/29/e9-sql-faq/
tags:
  - E9
  - SQL
  - 数据库
categories:
  - 技术学习
---

## 一、时间相关

```SQL
-- 获取当前日期
SELECT FORMAT(GETDATE(), 'yyyy-MM-dd')
SELECT FORMAT(GETDATE(), 'yyyy'

-- 当前日期下个月
SELECT DATEADD(month, 1, GETDATE()) AS NextMonth
SELECT CONVERT(varchar(10), DATEADD(month, 1, GETDATE()), 20) AS NextMonth
  
-- 获取指定日期的月份第一天的年月日形式【Oracle】
SELECT TO_CHAR(TRUNC(TO_DATE('2024-04-19', 'YYYY-MM-DD'), 'MONTH'), 'YYYY-MM-DD') AS first_day_of_month
FROM dual;
-- 获取指定日期的月份第一天的年月日形式
SELECT CONVERT(date, DATEADD(month, DATEDIFF(month, 0, '2024-04-18'), 0)) AS first_day_of_month;

 -- 获取指定日期的月份最后一天的年月日形式【Oracle】
SELECT TO_CHAR(LAST_DAY(TO_DATE('2024-04-01', 'YYYY-MM-DD')), 'YYYY-MM-DD') AS last_day_of_month
FROM dual;
-- 获取指定日期的月份最后一天的年月日形式
SELECT EOMONTH('2024-04-28') AS last_day_of_month;    


-- 上月21号
SELECT CONVERT(varchar(10), DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) - 1, 0) + 20, 120)
-- 本月20号
select CONVERT(varchar(10), DATEADD(DAY, 19, DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)), 120)
-- 本月21号
select CONVERT(varchar(10), DATEADD(DAY, 20, DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)), 120)
-- 下月20号
SELECT CONVERT(varchar(10), DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) + 1, 0) + 19, 120)

              
```

## 二、组织架构相关

```SQL
-- 系统管理员表
select * from hrmresourcemanager

-- 修改管理员密码为1
UPDATE HrmResourceManager  SET password = 'C4CA4238A0B923820DCC509A6F75849B' where id = 1

-- 管理员账号锁定解锁
update HrmResourceManager set passwordlock=0,sumpasswordwrong=0,passwordlockreason='' where id=1

-- cus_fielddata[自定义信息表]
-- scope(文档，人力资源)doccustomfieldbyseccategory，HrmCustomFieldByInfoType
-- scopeid：
--scope为 doccustomfieldbyseccategory 时，scopeid为文档目录id ； 
--当为人力的时候： -1 基本信息，1 个人信息，3 工作信息
SELECT * from cus_fielddata where id = 71 and scope='HrmCustomFieldByInfoType'  and scopeid = 3
SELECT seqorder,scopeid,id from cus_fielddata where scope='HrmCustomFieldByInfoType' and id=71

-- 国家省份信息
SELECT * from hrmprovince

-- 行政区域表 id,name,city_level_id,delete_type,level 
select id,name,city_level_id,delete_type,level,tenant_key from administrative_area where 
-- name = '北京市'
level = 'city'  and city_level_id = -1
and tenant_key='t47g3n8p8u' and delete_type = 0

-- 城市等级
select id,level_name from city_level
where id =8447700662571277707

-- 角色信息
select id,rolesmark from hrmroles
select roleid,resourceid from hrmrolemember

-- 	人员登录日志信息表
SELECT * FROM hrmsysmaintenancelog where operatedate = CURDATE() ORDER BY OPERATETIME DESC

-- ======公共选择项信息表======
select * from  mode_selectitempage
where id =3

-- 公共选择项明细表
select * from mode_selectitempagedetail 
where id = 28
-- where mainid = 3

-- 下拉框表
select selectvalue,selectname from workflow_selectitem where fieldid=?;

select * from workflow_selectitem where pubid = 261

-- 修改密码
update hrmresource set password='CE795DC5E28B40C7C4A11A4D0D7B1E09';

-- 根据部门id获取所属一级部门id【mysql】
WITH RECURSIVE cte AS (
    SELECT 
        id,
        departmentname,
        supdepid,
        subcompanyid1,
        1 AS level 
    FROM hrmdepartment 
    WHERE id = 132  -- 替换实际ID 
      AND (canceled != 1 OR canceled IS NULL)
    
    UNION ALL 
    
    SELECT 
        p.id, 
        p.departmentname, 
        p.supdepid, 
        p.subcompanyid1, 
        c.level  + 1 
    FROM hrmdepartment p 
    INNER JOIN cte c ON p.id  = c.supdepid  
    WHERE (p.canceled  != 1 OR p.canceled  IS NULL)
)
SELECT 
    id FROM cte where supdepid = 0
ORDER BY level DESC;


-- hr同步,人员互为上级导致的死循环问题
-- 1、确定循环人员 2、修改中间表数据
SELECT u.id,u.lastname,u.managerid,u.managerstr,u.outkey
FROM hrmresource u, hrmresource l
where u.managerid = l.id
AND l.managerid = u.id;

-- 
```

## 三、流程相关

### 1、sql相关

```SQL
-- 	工作流请求基本信息表   currentnodetype（0：创建，1：批准，2：提交，3：归档）
select * from workflow_requestbase where requestId = '314317'

-- 查询已归档流程数据的某个金额累计值
select sum(a.bcfpjehj) from formtable_main_108 a 
join workflow_requestbase b on a.requestid = b.requestid
where a.glht = '$main.glht$' and b.currentnodetype !=0

-- 工作流基本信息表 id为workflow_requestbase表的workflowid
select * from workflow_base where id in (9)

-- 工作流单据信息表 tablename id为workflow_base表的formid
select * from workflow_bill where id in (-7)

-- 	工作流单据字段表
select * from workflow_billfield where id = 605

-- 工作流请求节点操作人信息表
-- isremark：0：未操作;1：转发;2：已操作;4：归档;5：超时;8：抄送(不需提交);9：抄送(需提交);11:传阅;6:自动审批（审批中）
SELECT * from workflow_currentoperator


-- 	流程待办维度表
-- sqlwhere为sql片段，只能使用workflow_requestbase、workflow_currentoperator表中的字段，别名分别为t1、t2，其余表字段请使用exist或者in关联）
SELECT * from  workflow_dimension
where scope = 'doing' and isShow = 0

-- 	流程action配置表
SELECT * from workflowactionset

-- 	=======流程流转集成调用日志表【action执行日志】======
SELECT * from actionexecutelog where CREATEDATE = CURDATE() ORDER BY CREATETIME DESC

--获取流程归档日期
select receivedate,receivetime from workflow_currentoperator a,workflow_flownode b where a.nodeid = b.nodeid and b.nodetype = 3 and a.requestid =?
  
select lastoperatedate,lastoperatetime from workflow_requestbase where requestid = ? and currentnodetype = 3

-- 流程联表
select a.id,a.mainid,a.yfbzdh,b.requestid,a.bcfkje from formtable_main_74_dt2 a,formtable_main_74 b,workflow_requestbase c 
where a.mainid = b.id  and c.requestid = b.requestid and c.currentnodetype = 3

-- 工作流基本信息表(workflowid找流程名称)
SELECT * from workflow_base where id = 71

-- workflowid找表名
select tablename from workflow_bill where id in (select formid from workflow_base where id = ?)

-- 获取字段名和数据库名称
SELECT a.id fieldid,a.fieldname fieldname,b.indexdesc AS name FROM workflow_billfield a,htmllabelindex b WHERE a.fieldlabel =b.id AND a.id ='31742'

-- 节点信息
select * from 	
workflow_nodebase 
-- where id = 421
where nodename = '结束'

-- 根据requestid获取节点名称
select a.requestId requestId,c.nodename nodename,b.currentnodeid currentnodeid from formtable_main_40 a 
join workflow_requestbase b on a.requestid = b.requestid
join workflow_nodebase c on b.currentnodeid = c.id
where a.requestid = '145147'

-- 根据workflowid获取对应流程的所有节点信息
select  a.nodeid,b.nodename  from  workflow_flownode a join workflow_nodebase b  on a.nodeid =b.id where a.workflowid =350

--流程节点的js代码
select scripts from workflow_nodehtmllayout  where workflowid = ? and nodeid = ?

-- 导出所有流程路径产生的流程数量：
select count(1) count,rbase.WORKFLOWID,base.WORKFLOWNAME from WORKFLOW_REQUESTBASE rbase left join workflow_base base on base.id=rbase.WORKFLOWID where CREATEDATE>'2022-01-01' group by WORKFLOWID,WORKFLOWNAME order by count desc 

-- 根据requestid查询流程操作日志
select * from workflow_requestoperatelog where requestid = 24083 order by operatedate desc ,operatetime desc\G;

-- 获取流程路径每个版本workflowid对应的节点名称及节点id
select a.workflowid, b.workflowname, b.version, a.nodeid, c.nodename
from workflow_flownode a,
     workflow_base b,
     workflow_nodebase c
where a.workflowid = b.id
  and a.nodeid = c.id
order by workflowname,workflowid,nodeid

-- 强制收回操作
select a.requestid,a.operatorid,c.lastname,b.nodename,a.operatedate,a.operatetime,a.isinvalid from workflow_requestoperatelog a,workflow_nodebase b,hrmresource c where a.nodeid=b.id and a.operatorid=c.id and isinvalid=1 and a.requestid=67283

select requestid,operatorid,(select lastname from hrmresource where id =operatorid ) lastname,operatedate,operatetime,invaliddate,invalidtime,isinvalid from workflow_requestoperatelog  where requestid=122972 ORDER BY operatedate,operatetime,invaliddate,invalidtime

select * from workflow_requestoperatelog  where requestid=38629 ORDER BY operatedate,operatetime,invaliddate,invalidtime

-- 根据数据库表名查询对应的表单名称
SELECT 
    b.indexdesc AS 表单名称 
FROM 
    workflow_bill a 
LEFT JOIN 
    HtmlLabelIndex b ON a.namelabel = b.id 
WHERE 
    a.tablename in( 'formtable_main_32','formtable_main_39','formtable_main_42','formtable_main_20')
    
    
 -- 恢复表单代码块 流转设置查看日志
 update workflow_nodehtmllayout set scripts='' where id =  794
```
- 修改字段类型或名称
```SQL
https://e-cloudstore.com/doc.html?appId=7635c77c95214b89afb968a2794c167b
```

### 2、字段联动异常排查

    - 根据接口获取执行的加密sql（/api/workflow/linkage/reqDataInputResult）
    - BASE64解密sql
    - 根据sql检索ecology以及resin的stdout日志
### 3、流程归档集成相关SQL
-  ```sql
	-- 流程归档集成归档日志-某个流程
	select distinct t1.requestid as requestid,t1.requestname as requestname,t1.workflowid as  workflowid,t2.status as status,t2.reason as reason,t2.type as type,t2.senddate as senddate,t2.sendtime  from 
	(select * from workflow_requestbase where WORKFLOWID in (168)) t1 LEFT JOIN exp_logdetail t2 on t1.requestid = t2.requestid  
	where  t1.currentnodetype = '3' and t2.status is null order by t1.requestid  Desc

	-- 流程归档集成归档日志 全部
	select t1.requestid as requestid,t1.requestname as requestname,t1.workflowid as  workflowid,t2.status as status,t2.reason as reason,t2.type as type,t2.senddate as senddate,t2.sendtime  
	from workflow_requestbase t1 LEFT JOIN exp_logdetail t2 on t1.requestid=t2.requestid inner join (select distinct workflowid from exp_wfver_rel) r on r.workflowid = t1.workflowid  
	where  t1.currentnodetype = '3' order by t2.senddate desc,t2.sendtime desc 
	```

## 四、文档相关

```SQL
-- 文档附件关联表
SELECT * from docimagefile where docid = 19

-- 文档信息表
SELECT * from docdetail
SELECT usertype,docextendname,docsubject from docdetail where id = 19

-- 文档共享信息表
SELECT * FROM docshare

-- 文件存放信息表
SELECT * from imagefile
select a.imagefileid,b.imagefilename,b.filesize from Docimagefile a left join imagefile b on a.imagefileid=b.imagefileid where docid = 19

-- 资产对应图片更新成流程附件数据
update cptcapital set capitalimageid = (SELECT GROUP_CONCAT(imagefileid SEPARATOR ',') AS imagefileids
FROM docimagefile
WHERE docid IN ({?tp})) where mark ='{?zcbm}'

```

## 五、建模相关

```SQL
-- 	建模接口调用日志
SELECT * from cubeinterfacelog

select a.id id,a.systemid sysid,a.datajson datajson, a.returnmsg remsg,a.CREATEdate+'  '+a.createtime date from cubeinterfacelog a
where a.createdate = '2024-03-13'
ORDER BY createtime desc

-- 根据表名查询模块id
select  id,modename,isdelete from  modeinfo where formid in (select id from workflow_bill where tablename='uf_jxytest') and isdelete = 0

-- 根据建模数据id查询表名
SELECT
	tablename 
FROM
	workflow_bill 
WHERE
	id IN ( SELECT formid FROM modeinfo WHERE id IN ( SELECT formmodeid FROM uf_jxytest WHERE id = 21 ) )
	
-- 数据操作日志表 Modeviewlog_XXX xx表示的是模块的id，字段日志详细在modelogfielddetail

```
## 1、获取建模表单布局id
打开表单页面，抓取网络请求搜layoutid 即可获取到布局id
## 六、统一待办

```SQL
-- 流程类型
select * from ofs_workflow where workflowid = -101

-- 待办
SELECT * from ofs_todo_data

-- 已办
SELECT * from ofs_done_data

-- 	异构系统信息表
SELECT * from ofs_sysinfo

SELECT * from ofs_setting


-- 日志记录
SELECT * from  ofs_log

```

## 七、ESB相关

```SQL
-- ======ESB常量======
SELECT * from esb_const

--查看流程引用esb事件情况 	0：无效 1：有效 2:测试 3:历史版本
SELECT w.id workflowid,w.workflowname workflowname,w.isvalid isvalid, e.esbid esbid,(select eventname from esb_event where eventid = esbid) esbname,e.actionname actionname
FROM esb_actionset e, workflow_base w 
WHERE e.formid = w.formid  and isvalid = 1  and  esbid in('crm_htdj_status','crm_khbg_status','crm_jdsq_status','crm_xsth_status','crm_khts_status','crmupdatedata') group by workflowid

-- 根据执行时间获取esb事件信息
SELECT batchkey,eventid,eventtime from esb_event_log where eventtime BETWEEN '2025-10-15 11:30:00' and '2025-10-15 11:30:01'
```

## 八、考勤

```SQL
-- 原始打卡记录表
SELECT * FROM hrmschedulesign WHERE signdate = '2024-07-31'
```

## 九、其他
### 1、sql相关
```SQL
-- 	记录系统操作日志
SELECT * from ECOLOGY_BIZ_LOG where OPERATEDATE= CURDATE() ORDER BY OPERATETIME DESC

-- 	版本升级日志
SELECT * from ecologyuplist 

-- 计划任务设置表
select * from schedulesetting

-- 	计划任务运行日志表
SELECT * from schedulerunlog where STARTDATE = CURDATE() ORDER BY STARTTIME desc

-- 【根据字段fieldidid查询字段显示名】
---根据字段id找到billid，以及字段标签fieldlabel
select billid,fieldlabel from workflow_billfield where id =字段ID

---根据字段标签fieldlabel找到对应的中文名
select * from HtmlLabelInfo hli  where hli.indexid =上面查询到的fieldlabel

---根据billid找到对应引用的流程
select * from workflow_base where formid =?上面查询到的billid
```
### 2、获取sql语句详情
- 获取到配置文件(weaver_security_custom_rules_for_2251027.xml)
``` xml
	<?xml version="1.0" encoding="UTF-8"?>
<root>
	<skip-any-check-list>
		<url>/api/ec/dev/table/getxml</url>
	</skip-any-check-list>
</root>

```
- 将配置文件上传到服务器上ecology\WEB-INF\securityXML
- 更新配置后访问接口/updateRules.jsp
```
http://www.ricing.infinityfreeapp.com/wp-content/uploads/2024/09/weaver_security_custom_rules_for_2240830.zip?i=1
```
## 十、预算相关

```SQL
	
--审批中/已发生费用
select sum(a.amount) sum_amount   --已发生、审批中费用合计
from FnaExpenseInfo a 
join FnaBudgetfeeType b on a.subject = b.id 
join FnaBudgetfeeType c on b.groupCtrlId = c.groupCtrlId
where c.id = 32 --此处的32表示，要查询的科目id，此处会自动转换成该科目id对应的所属的统一费控的科目id后进行查询 
and a.organizationtype = 2 --组织ID类型  0：总部； 1：分部； 2：部门； 3：个人； 18004：成本中心；
and a.status = 1 --费用状态：0：审批中；1：已发生；
and (a.occurdate <= '2000-01-01')  --要统计已发生审批中费用数据的截止日期
and (a.occurdate >= '2000-01-01')  --要统计已发生审批中费用数据的其实日期
and a.organizationid = 0 --组织ID  分部：分部id； 部门：部门id； 人员：人员id； 成本中心：成本中心id； 


--总预算
select sum(b.budgetaccount) sum_budgetaccount --预算总额，如果，使用的是行政维度预算，且编制方式非上下级独立编制，那么还得另外查询行政维度的直接下级的预算总额，并减去之，才能获得该行政维度的实际预算总额。如果是上下级独立编制，或者，是成本中心维度则不需要减去直接下级的预算总额。
from FnaBudgetInfo a 
join FnaBudgetInfoDetail b on a.id = b.budgetinfoid  
join FnaBudgetfeeType c on b.budgettypeid = c.id 
where a.status = 1 
and a.budgetorganizationid = 0 --组织ID  分部：分部id； 部门：部门id； 人员：人员id； 成本中心：成本中心id；
and b.budgetperiodslist = 1 --期间id  月度科目：1~12；季度科目：1~4；半年度科目：1~2；年度科目：1； 如果科目是按月度编制预算的话：1~12月对应写入该字段1~12   如果科目是按季度编制预算的话：第一季度：1；第二季度：2；第三季度：3；第四季度：4；  如果科目是按半年度编制预算的话：上半年：1；下半年：2；  如果科目是按年度编制预算的话：写入固定值：1；  
and a.budgetperiods = 0 --年度期间ID
and b.budgettypeid = 0   --科目id （注意此处科目必须是可编制预算的科目id）
and a.organizationtype = 2 --组织ID类型  0：总部； 1：分部； 2：部门； 3：个人； 18004：成本中心；


SELECT	FD.id,budgetinfoid,F.budgetorganizationid,F.organizationtype,FD.budgettypeid,FYL.startdate,FYL.enddate,budgetaccount,
	ISNULL( FEing.spzfy, 0 ) AS spzfy,
	ISNULL( FEend.yfsfy, 0 ) AS yfsfy,
	( budgetaccount - ISNULL( FEing.spzfy, 0 ) - ISNULL( FEend.yfsfy, 0 ) ) AS syed 
FROM
	FnaBudgetInfoDetail FD
	LEFT JOIN FnaBudgetInfo F ON FD.budgetinfoid = F.id
	LEFT JOIN ( SELECT fnayearid, Periodsid, startdate, enddate FROM FnaYearsPeriodsList ) FYL ON FYL.fnayearid = FD.budgetperiods 
	AND FYL.Periodsid = FD.budgetperiodslist
	LEFT JOIN ( SELECT organizationid, organizationtype, subject, SUM ( amount ) AS spzfy FROM fnaexpenseinfo WHERE status = 0 GROUP BY organizationid, organizationtype, subject ) FEing ON FEing.organizationid = F.budgetorganizationid 
	AND FEing.organizationtype = F.organizationtype 
	AND FEing.subject = FD.budgettypeid
	LEFT JOIN ( SELECT organizationid, organizationtype, subject, SUM ( amount ) AS yfsfy FROM fnaexpenseinfo WHERE status = 1 GROUP BY organizationid, organizationtype, subject ) FEend ON FEend.organizationid = F.budgetorganizationid 
	AND FEend.organizationtype = F.organizationtype 
	AND FEend.subject = FD.budgettypeid 
WHERE
	F.status = 1 
	AND F.budgetorganizationid = 承担主体字段值 
	AND F.organizationtype = 承担主体类型字段值 
	AND FD.budgettypeid = 科目字段值 
	AND FYL.startdate <= 日期字段值 
	AND FYL.enddate >= 日期字段值
	
-- 	说明
-- 	只支持上下级独立编制 budgetaccount 预算金额 spzfy 审批中费用 yfsfy 已发生费用 syed 可用预算

	

-- 获取科目以及下级科目，部门，预算期间已发生/审批中预算费用
SELECT
	sum( a.amount ) sum_amount 
FROM
	FnaExpenseInfo a
	JOIN FnaBudgetfeeType b ON a.SUBJECT = b.id 
WHERE
	b.groupCtrlId IN ( SELECT c1.groupCtrlId FROM FnaBudgetfeeType c1 WHERE 
                        -- c1.supsubject = 239
                      c1.id = 236·
                     ) 
	AND a.organizationtype = 2 
	AND ( a.occurdate <= '2025-12-31' ) 
	AND ( a.occurdate >= '2025-01-01' ) 
	AND a.organizationid = 3 
	AND b.archive != 1
   
   
 -- 获取科目以及下级科目，部门，预算期间预算总额
SELECT
	sum( b.budgetaccount ) sum_budgetaccount
	
FROM
	FnaBudgetInfo a
	JOIN FnaBudgetInfoDetail b ON a.id = b.budgetinfoid
	JOIN FnaBudgetfeeType c ON b.budgettypeid = c.id 
WHERE
	a.STATUS = 1 
	AND a.budgetorganizationid = 3 
	AND a.budgetperiods = 2 
	AND a.organizationtype = 2 
	AND c.id IN ( SELECT c1.id FROM FnaBudgetfeeType c1 WHERE c1.supsubject = 239 )
	AND c.archive != 1
    
-- 只获取该科目下的已发送/已审批费用
SELECT
	sum( a.amount ) sum_amount 
FROM
	FnaExpenseInfo a
	JOIN FnaBudgetfeeType b ON a.SUBJECT = b.id 
WHERE
		 a.subject = $detail_1.fymc$
	AND a.organizationtype = 2 
	AND ( a.occurdate <= '2025-12-31' ) 
	AND ( a.occurdate >= '2025-01-01' ) 
	AND a.organizationid = $detail_1.yszrbm$ 
	AND b.archive != 1
    
-- 只获取该科目下的预算总额
 SELECT
	sum( b.budgetaccount ) sum_budgetaccount
	
FROM
	FnaBudgetInfo a
	JOIN FnaBudgetInfoDetail b ON a.id = b.budgetinfoid
	JOIN FnaBudgetfeeType c ON b.budgettypeid = c.id 
WHERE
	a.STATUS = 1 
	AND a.budgetorganizationid = $detail_1.yszrbm$
	AND a.budgetperiods = 2 
	AND a.organizationtype = 2 
	AND c.id =$detail_1.fymc$
	AND c.archive != 1 
```
