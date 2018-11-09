### 1.Data Loading
- **Incremental Loading+Variable+QVD file**  

  **step1**: Connect ODBC `LIB CONNECT TO 'ODBC Connection;`

  **step2**:  Set Varible
  `let startday = date(Today()-31,'YYYY-MM-DD');   
  let midday1 =  date(Today()-20,'YYYY-MM-DD');   
  let midday2 =  date(Today()-10,'YYYY-MM-DD');   
  trace(startday);`

  **step3**:  Data Loading SQL (demo)   
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


### 2.Report Logic for Viz   
   * compute growth rate：  
  `SUM({$<year={$(=MAX(year))}>}distributorsti_count)/SUM({$<year={$(=MAX(year)-1)}>}distributorsti_count)-1`
  

   * show last day    
  `count(distinct if(p_event_date = max(total p_event_date),lps_did))`
  
   * cumulative computation  
    `Rangesum(above(sum({<type = {'0','1'},[report_qlik_base_data.region]={'1'}>}[report_qlik_base_data.regNum]),0,RowNo()))+
  Rangesum(above(sum({<type = {'1','3','4'},[report_qlik_base_data.region]={'2'}>}[report_qlik_base_data.regNum]),0,RowNo()))`

   * color set  
  `if(max($(dur2))=30,rgb(16, 78 ,120),   
  if(max($(dur2))=60,rgb(0, 139 ,139),  
  if(max($(dur2))<=90,rgb(135 ,206 ,255),  
  if(max($(dur2))=120,rgb(255 ,233, 191),   
  if(max($(dur2))=150,rgb(34, 139, 34),  
  if(max($(dur2))>150,rgb(205, 96 ,144)))))))` 

   * default show last day value and make filter auto   
  `if(GetSelectedCount(p_event_date)>1,count(distinct lps_did),count(distinct if(p_event_date= max(total p_event_date),lps_did)))`

   * use camera times 1 month
  `class(aggr(count(param1_value_id)/(today()-min(p_event_date))*30,lps_did_id),2)`

   * sumif 
  `Sum({<brand={' Desktop',' Notebook'}>}sdrev)
  /Sum({<brand={' Desktop',' Notebook'}>}ca)`

   * stacked barchart with percentage
  `Sum(sdrev)/sum(total <quarter> sdrev) --quater is x value(dimension)`

   * null and '' value: use len is more reliable
   `if(len(family_le)>1,family_le)`

   * auto show top 5 with any filter
  `if(Match(rank(sum(sdrev)),'1','2','3','4','5')>0,1-Sum(pcacost)/sum(sdrev))`
   * set auto title
  `if(GetSelectedCount(fy_quarter)>0,Sum({<fy_quarter={"$(vLastfy_quarter)"}>}sellin_sum) ,Sum({<year_fis={"$(vLast2Year_fis)"}>}sellin_sum)) `
   * variable in report
  ![Vairable](/variable.png)
  
   * avoid distinct between cities   
   `sum(aggr( Count(distinct {<chan_year={'$(vMaxYear_chan)'},chan_month={'$(vMaxYear_Month_chan)'},distr_channel={'41'}>}zsold_to),chan_city))`
  

