![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Modify Custom SQL Job Alert Notification Process
**Post Date: November 9, 2015**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>I was thinking perhaps someone would want to rename their Job Alert notifications; if they followed the instructions I put on this blog. Don't forget you have to modify the Trigger on the msdb..sysjobhistory table as well. Additionally; you would need to deploy the changes across all your database servers to make it consistent. Here's some SQL logic to take care of all of that for you. It will both check for the Job names to change, and confirm they were indeed changed along with the create date of the Trigger after it's dropped and recreated with the appropriate Job name coded within it.</p> 


## SQL-Logic
```SQL
use msdb;
set nocount on
 
if exists(select name from msdb..sysjobs where name = 'SEND SQL BACKUP ALERTS') begin
exec msdb..sp_update_job
@job_name = 'SEND SQL BACKUP ALERTS'
,   @new_name = 'DATABASE ALERTING - Backups'
end
 
if exists(select name from msdb..sysjobs where name = 'SEND SQL JOB ALERTS') begin
exec msdb..sp_update_job
@job_name = 'SEND SQL JOB ALERTS'
,   @new_name = 'DATABASE ALERTING - Jobs'
end
 
use msdb;
set nocount on
set ansi_nulls on
set quoted_identifier on
 
drop trigger trig_check_for_job_failure
go
 
create trigger [dbo].[trig_check_for_job_failure] on [dbo].[sysjobhistory] after insert
as
begin
set nocount on
declare @is_fail    int
set @is_fail    = (select case when [message] like '%The step failed%' then 1 else 0 end from msdb..sysjobhistory where instance_id in (select max(instance_id) from [msdb]..[sysjobhistory])) if   @is_fail    = 1
begin
exec msdb.dbo.sp_start_job @job_name = 'DATABASE ALERTING - Jobs' end
end
go
 
select name from msdb..sysjobs where name like 'Database%' order by name asc
 
select
left(strig.create_date, 19) + ' ' + datename(dw, strig.create_date) ,   'table_name'    = stable.name
,   'trigger_name'  = strig.name
from
sys.triggers strig join sys.tables stable on strig.parent_id = stable.object_id where
stable.name = 'sysjobhistory'
order by
stable.name, strig.name asc
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

    
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

