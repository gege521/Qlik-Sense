### 1.Data Loading
- 增量加载+变量+qvd使用   

-  链接ODBC数据库`LIB CONNECT TO 'ODBC Connection';` 

-  设置变量
  `let startday = date(Today()-31,'YYYY-MM-DD');  
  let midday1 =  date(Today()-20,'YYYY-MM-DD');  
  let midday2 =  date(Today()-10,'YYYY-MM-DD');  
  trace(startday);`

-  Demo   
  ```temp:
  sql 
  SELECT p_event_date,lps_did ,device_model,app_channel,app_version,province,city,town,app_key,t2.range,t2.category  
  from  d_pc_ace.ods_accessible_device_dau_list t1   
  LEFT JOIN d_pc_ace.ace_software_device_model t2   
  on t1.device_model = t2.series   
  where p_event_date >= '$(startday)'   
  and p_event_date< '$(midday1)'   
  and app_key in ('1','2','3');  
  temp:  
  Concatenate    
  sql   
  SELECT p_event_date,lps_did,device_model,app_channel,app_version,province,city,town,app_key,t2.range,t2.category   
  from  d_pc_ace.ods_accessible_device_dau_list t1      
  LEFT JOIN d_pc_ace.ace_software_device_model t2     
  on t1.device_model = t2.series   
  where p_event_date >= '$(midday1)'    
  and p_event_date< '$(midday2)'   
  and app_key in ('1','2','3');  
  STORE temp INTO [lib://name.qvd](qvd);   
  load p_event_date, lps_did ,device_model,app_channel,app_version,province,city,town,app_key,    
  if(isnull(range),'other',range)as range,    
  if(isnull(category),'other',category)as category       
  from [lib://name.qvd](qvd);     


### 报表逻辑 
- **计算环比**：  
  `SUM({$<year={$(=MAX(year))}>}distributorsti_count)/SUM({$<year={$(=MAX(year)-1)}>}distributorsti_count)-1`






