
=============================================================================================================
//Search for User login Activities

index=_audit action="login attempt" NOT (user=admin OR user="splunk-system-user" OR user="XXXXX-*")|eval Timestamp=strftime(_time,"%d/%b/%Y %H:%M:%S")|table Timestamp,user,agency,action,info|sort 0 Timestamp

=============================================================================================================
//Search for search_id , search query, user, runtime, etc

index=_audit NOT(user="splunk-system-user" OR user="admin" OR user="XXXXX-*") action=search info!="granted"|table search_id,search,scan_count,event_count,result_count,available_count,drop_count,is_realtime,exec_time,search_et,search_lt,api_et,api_lt,searched_buckets,total_run_time,info,user|eval Run_Time=toString(total_run_time,"duration")|eval exec_time=strftime(exec_time,"%d/%b/%y %H:%M:%S"),search_et=strftime(search_et,"%d/%b/%y %H:%M:%S"),search_lt=strftime(search_lt,"%d/%b/%y %H:%M:%S")|RENAME Run_Time as "Search Run Time",exec_time as "Search Exec.Time",search_et as "Search Data From", search_lt as "Search Data To"|fields - total_run_time,api_et,api_lt,available_count,drop_count,is_realtime|sort 0 -"Search Run Time"|join search_id [search index=_audit NOT(user="splunk-system-user" OR user="admin" OR user="XXXXX-*") action=search info=granted search=* NOT "search_id='scheduler" NOT "search='|history" NOT "search='typeahead" NOT "search='| metadata type=* | search totalCount>0"|fields search_id, search]


=============================================================================================================
//Search for user search time by status

index=_audit NOT(user="admin" OR user="splunk-system-user" OR user="XXXXX-*") action=search info!="granted"|stats count(search_id) as tsearches,SUM(total_run_time) as trun by info,user|eval Run_Time=toString(trun,"duration")|RENAME Run_Time as "Total Run Time", tsearches as "Total Searches",info as "Search Status"|fields user,"Total Searches","Search Status","Total Run Time"

=============================================================================================================
