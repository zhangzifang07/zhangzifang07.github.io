---
title: E10SQL以及常见问题
date: 2026-06-29 16:00:00
permalink: /2026/06/29/e10-sql-faq/
tags:
  - E10
  - SQL
  - 数据库
categories:
  - 技术学习
---

# 一、组织架构

```SQL
-- 人员表
select * from EMPLOYEE where TENANT_KEY = 't4wqf62ghj' and
-- id = 1086699487085871119
username in( '姬兴洋','夏一')

-- 账号表
select * from user_info where TENANTKEY = 't4wqf62ghj'
select * from user_info_password

-- 人员关系表【上下级，层级等】
select * from emp_link

-- 人员自定义字段
select a.id,a.username,b.sszz
from eteams.employee a
join ecology10.ft_1256290351004131375 b on a.formdata = b.form_data_id
where  a.id =  1211824305394376705

-- 获取人员所属部门，分部 通过人员id
SELECT  
  e.id,  
  e.username,  
  e.DEPARTMENT, -- 所属组织=部门或者分部  
  CASE 
    WHEN d.id = sub.id AND sub.id IS NOT NULL THEN NULL 
    ELSE d.id 
  END AS deptId, -- 所属部门  
  sub.id AS subId -- 所属分部  
FROM employee e  
  LEFT JOIN department d ON e.DEPARTMENT = d.ID  
  LEFT JOIN department sub ON d.subcompanyid = sub.id
WHERE e.id = '指定人员ID' -- 请替换为具体的人员ID

-- 部门自定义字段
select a.id,a.code,a.name,a.outkey,a.type,formdata,b.bmjl,b.ddhb
from eteams.department a
join ecology10.ft_1165358755073105931 b on a.formdata = b.form_data_id
where  a.type ='department'

-- 部门&分部表
select id,code,name,parent,outkey,type,is_delete 
-- select * 
from department 
where TENANT_KEY = 't4wqf62ghj' ORDER BY type
and  type = 'subcompany' AND IS_DELETE = '0' AND virtualid = 1

-- 部门&分部上下级关系表(存储每个部门分部链式上下级关系数据)
select * from depart_link

-- 岗位表
select * from Position 
-- 人员角色关联表
select * from auth_user_role

select * from hrm_resource_log

-- hr同步删除数据

-- 人员数据
SELECT id, username, status, tenant_key, position , superior, department, last_login_time , active_date, permanently_delete 
FROM employee WHERE tenant_key = 't4wqf62ghj' and username not in ('姬兴洋','夏一')

DELETE from employee where tenant_key = 't4wqf62ghj' and username not in ('姬兴洋','夏一');

-- 部门数据
SELECT ID, CODE, DESCRIPTION, DISPORDER, NAME, PARENT, CREATOR, CREATE_DATE, STATUS, MODIFIER, MODIFIED, formdata, coadjutant, subcompanyid FROM DEPARTMENT WHERE TENANT_KEY = 't4wqf62ghj' AND (type = 'department' OR type IS NULL) AND IS_DELETE = '0' AND virtualid = 1

DELETE FROM DEPARTMENT WHERE TENANT_KEY = 't4wqf62ghj' AND (type = 'department' OR type IS NULL) AND IS_DELETE = '0' AND virtualid = 1

-- 分部数据
SELECT ID, CODE, DESCRIPTION, DISPORDER, NAME, PARENT, CREATOR, CREATE_DATE, STATUS, MODIFIER, MODIFIED, formdata, coadjutant FROM DEPARTMENT WHERE TENANT_KEY = 't4wqf62ghj' AND type = 'subcompany' AND IS_DELETE = '0' AND virtualid = 1 and code is null

DELETE FROM DEPARTMENT WHERE TENANT_KEY = 't4wqf62ghj' AND type = 'subcompany' AND IS_DELETE = '0' AND virtualid = 1 and code !='0000'

-- 判断部门-1211463747243565521是否属于某个部门-1211463747243565338及其子部门下
SELECT CASE 
		WHEN count(*) > 0 THEN 0
		WHEN '1211463747243565521' = '1211463747243565338' THEN 0
		ELSE 1
	END AS result
FROM eteams.depart_link dl
WHERE pid = '1211463747243565338'
	AND cid = '1211463747243565521'
	AND dl.TENANT_KEY IN ('t47g3n8p8u', 'all_teams')

-- 租户管理平台管理员信息(ecology10库)
select * from tenant_admin_manager

UPDATE tenant_admin_manager 
SET password = 'I9BEDkl3m82Lgy9xof4aVGR+KrHsFwigiyS7pyszoqmyD3M73m+Euks4FF9D/bb9' 
WHERE account = 'weaveradmin';

```
## 1、清理组织人事缓存
- 访问地址/sp/hrm/checkapi
``` 
/api/hrm/fix/removeCache 
{ "all":true }
```
## 2、delete_type字段值的含义
```sql
--IS_DELETE
逻辑删除（0未删除，1已删除）

-- delete_type
|0|默认值，代表正常，没有删除
|1|代表逻辑删除，但不移走数据，还存在当前表里（模块自己实现）
|2|代表逻辑删除，数据需要从当前表里移动到一个结构相同的备份表里（由定时任务完成，模块自实现）
|3|代表物理删除，由定时任务异步执行，会把所有3的数据物理删除
```
# 二、流程相关

```SQL
-- 流程状态 ： 0：草稿:1：审批中、3：正常归档、4：强制结束、、6：待提交、7：暂停、8：撤销
select ft.id,ft.jzry,wfreq.flowstatus from ft_1271101167615328606 ft
left join wfc_requestbase wfreq on ft.id = wfreq.requestid
where ft.delete_type = 0 and wfreq.flowstatus in (1,3,4)

-- 流程当前操作者
select * from wfc_operate where tenant_key = 't4wqf62ghj' and requestid = 1122700504261419019 and  delete_type = 0 and isremark in (0,1)

-- 流程节点表 nodetype(节点类型：0发起 1审批 2确认 3结束 4子流程节点 5投票 6循环审批 7自动处理 8等待 9查阅)
select id,nodename,nodetype,nodeattr,tenant_key from wfp_node where id=1087053250514534407

-- 获取某个节点下的流程数据
select *  from ft_ycsqd where id in (select requestid from wfc_currentnode where nodeid =1185047512149245954 and delete_type =0) and delete_type =0 and sfwj !=1

-- 获取流程当前节点
SELECT nodename FROM wfp_node 
WHERE id = ( SELECT nodeid FROM wfc_currentnode WHERE requestid =1122700504261419019 AND delete_type = 0 )

-- 工作流名称、节点名称、操作组名称查询参考sql
select wfp_base.id as workflowid,wfp_base.workflowname,wfp_node.nodename,wfp_opt_group.groupname 
from wfp_base,wfp_node,wfp_opt_group 
where wfp_base.id = wfp_node.workflowid and wfp_node.id = wfp_opt_group.nodeid and wfp_base.delete_type = 0 and wfp_node.delete_type = 0 and wfp_opt_group.delete_type = 0 and wfp_base.status = 1

-- 流程节点和工作流基本信息
select wn.id,wn.workflowid,wn.nodename,wn.nodetype,wb.workflowname from wfp_node wn,wfp_base wb
where wn.workflowid = wb.id
and wn.nodetype = 0 and wb.workflowname in('现场异常申请单','采购申请','出库申请单','委外申请单')

-- 查找流程的明细表名
select b.table_name,a.title from sub_form a left join form_table b on a.id = b.form_id 
where a.form_id in (select relatekey  from wfp_relateform   where  workflowid = 1204327968226631682)
and a.delete_type = 0

-- 查找流程的表名
select table_name from form_table where form_id = (select relatekey  from wfp_relateform   where  workflowid = 1204327968226631682)

-- 通过主流程请求id查询对应生成的子流程请求id：
select * from wfc_subRequest where mainrequestid=xxx

-- 通过子流程请求id查询对应生成的主流程请求id：
select * from wfc_subRequest where subRequestid=xxx

-- 选项模板的选项ID查询选项名称
-- 1、先根据选项模板名称查询选项模板id
select id from formdata_template where name = '税率'
-- 2、再根据选项模板id和选项value值查询选项名称
select value_key,name from formdata_template_details where template_id = 1269352467419291649

```

# 三、文件相关

```SQL
-- 流程附件从数据库里查询文件路径
-- 根据附件id先查到url
select  url   from fileobj f where id =1082709879018127363 order by UPLOAD_TIME  desc

-- 根据url查询存储位置
select file_path,file_url from file_storage_info fsi  where file_url =  'f250f7bd-cc5a-4ec9-ad3b-5d60ccb9d19f'
```

# 四、集成相关
## 1、HR同步日志排查

```SQL
-- ic_hr_synclogdt：同步日志明细表，记录每条同步记录的字段级变化详情
-- sync_status 0，失败 1，插入成功 2，更新成功 3，删除成功 4，封存成功 5，解封成功 6，警告 7 数据过滤
-- bs同步到E10人员状态转换 
-- 北森 1 待入职 2 试用 3 正式 4 调出 5 待调入 6 退休 8 离职 12 非正式 
-- E10: 1：试用 2：试用延期 3：正式;4：临时;5：实习 6：离职;7：退休
select 
	a.create_time AS 数据同步时间,a.sourcedata AS 第三方数据,a.targetdata AS 同步到OA的最终数据,a.out_key AS 第三方数据主键,a.target_key AS Oa数据主键,a.obj_name AS 同步数据名称,a.log_id AS 日志id,a.sync_status,a.errcode
from  ecology10.ic_hr_synclogdt a 
where 1=1
-- and	a.obj_name = '检具部'
AND a.targetdata is not null 
and a.TENANT_KEY = 't47g3n8p8u'
AND delete_type = 0 
-- and a.sync_status = 0
order by a.create_time desc limit 100
```
## 2、系统问题
- SDK包下载链接：http://10.147.49.77:10600/papi/open/singleSignon/sdk/download
## 3、接口地址查看
- 地址+/sp/opendoc

# 五、租户相关

```SQL
-- 租户信息
SELECT * from TENANT_INFO

-- 租户管理员信息
SELECT * FROM tenant_admin_manager
-- 重置密码默认  123456a@
update tenant_admin_manager set password = 'I9BEDkl3m82Lgy9xof4aVGR+KrHsFwigiyS7pyszoqmyD3M73m+Euks4FF9D/bb9' where account = 'weaveradmin';

-- 租户管理员锁定表
select * from tenant_admin_lock
```

# 六、ESB以及动作流相关
## 1、动作流请求参数设置信息接口
/api/bs/esb/setting/component/getComponentContext