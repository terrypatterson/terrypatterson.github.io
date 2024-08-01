---
title: Panopto LTI Migration Documentation
date: 2024-05-09 09:00:00 -0600
categories: [LTI Migration, Panopto]
tags: [LTI, panopto, main-documentation, documentation, queries]
---

# Panopto LTI Migration
## SQL Queries

### Find Panopto Content Items within Blackboard Courses

This document covers the queries used in the migration of my institution from the Panopto Building block integration to the LTI integration. 


The query below displays content from courses where a Panopto video, quiz, mashup, or link was embedded using the building block. The output results in the following information

| Output | Description |
|--------|-------------|
|Course ID|Blackboard Course ID|
|Course Name | Blackboard Course Name|
|Panopto Content Type | What content type is displayed (Quiz, Embed, Link, Embed in Content Item)|
|Content Item Availability | Is the item available to students|
|Date Created|The date the item was created either by a user or via a course copy
|Date Modified|The date the item was last modified, if never modified the date will be the same as the Date Created|

You will need to replace the following in the query below.

|Name|Replace With|
|----|------------|
|Data Source Key|Your data source key for the courses you want to review. Can be commented out if you want to look at every course or you can use the commented out line above to find a set of course IDs with a common term code.|
|PANOPTO_INSTANCE_DOMAIN|Replace this with your Panopto instance domain, "myuni.hosted.panopto.com" for example.|
|INSTANCE_NAME|Replace this with the Instance Name that coordinates with your building block provider configuration within Panopto.|


```
select distinct(cc.pk1),
	   cm.course_id as "Course ID",
	   cm.course_name as "Course Name",
	   case
	   	when cc.cnthndlr_handle = 'resource/x-bb-bltiplacement-panopto-quiz-lti' then 'Panopto Quiz'
	   	when cc.cnthndlr_handle = 'resource/bb-panopto-bc-mashup' then 'Panopto Embed Mashup Item'
		when cc.cnthndlr_handle like 'hyperlink/coursecast' then 'Panopto Video Link'
		else 'Panopto Embed into Content Item'
	   end as "Panopto Content Type",
	   cc.title as "Content Name within Blackboard",
	   case
	   	when cc.available_ind = 'Y' then 'Yes'
	   	when cc.available_ind = 'N' then 'No'
	   	else 'Unknown Status'
	   end as "Content Item Availability",
	   cc.dtcreated as "Date Created",
	   cc.dtmodified as "Date Modified"
--	   cc.cnthndlr_handle
from course_main cm
	join data_source ds
	 on cm.data_src_pk1 = ds.pk1
	join course_contents cc
	 on cm.pk1 = cc.crsmain_pk1

where 
--cm.course_id LIKE '%224F%'
ds.batch_uid = '(Insert Data Source Key)'
--and cc.dtcreated > '2024-05-23 12:45:00' -- Could set a date and time to find newer created items if running to update data
and (
cc.cnthndlr_handle like 'resource/x-bb-bltiplacement-panopto-quiz-lti'
OR cc.cnthndlr_handle like 'resource/bb-panopto-bc-mashup'
OR cc.cnthndlr_handle like 'hyperlink/coursecast'
OR cc.main_data like '%src="https://PANOPTO_INSTANCE_DOMAIN/Panopto/Pages/Embed.aspx?instance=INSTANCE_NAME%'
--or cc.main_data like '%LTI.aspx&amp%'
)
```


### Panopto Building Block Content with Instructor Information

Want the same information as above, however need to reach out to the instructors who are impacted by the change? Then this query is for you. Note that this version uses a term name to define what courses will be in the output. You can also use a Course ID structure to find specific courses. 

This output will be slightly different. It will output the following

|Output|Description|
|------|-----------|
|Course ID| Blackboard Course ID|
|Course Name| Blackboard Course Name|
|Instructor Emails|List of Emails for users who are active instructors in the course|
|Instructor Name|Instructor Names for active instructor users|
|Term|Name of Term (if used)|
|Number of Panopto Quizzes|Counts number of Panopto Quizzes in the course|
|Number of Panopto Embeds|Counts number of Panopto Embeds in the course|
|Number of Panopto Links|Counts number of Panopto Links in the course|
|Number of Panopto Content Item Embeds|Counts number of Panopto Content Item Embeds in the course|

**Warning: The email and names will be duplicated, so deduplicate any emails when you pull from the output.**

```
select distinct(cm.course_id),
			   cm.course_name, 
			   STRING_AGG(u.email, ':') AS emails,
			   string_agg(distinct u.firstname||' '||u.lastname, ', ') as instructors,
			   t."name",
			   tbl_b.panopto_place_quiz,
			   tbl_a.panopto_place_embed,
			   tbl_c.panopto_place_link,
			   tbl_d.panopto_place_content_item
from course_main cm
	join course_users cu
	 on cm.pk1 = cu.crsmain_pk1
	join users u
	 on cu.users_pk1 = u.pk1
	join course_term ct
	 on cm.pk1 = ct.crsmain_pk1
    left join term t
     on ct.term_pk1 = t.pk1
    join course_contents cc
     on cm.pk1 = cc.crsmain_pk1
    left join (select COUNT(distinct(cc.pk1)) as panopto_place_embed, cm.pk1 as course_pk1 from course_main cm join course_contents cc on cm.pk1 = cc.crsmain_pk1 where cc.cnthndlr_handle like 'resource/x-bb-bltiplacement-panopto-quiz-lti' group by cm.pk1) tbl_a on cm.pk1 = tbl_a.course_pk1
	left join (select COUNT(distinct(cc.pk1)) as panopto_place_quiz, cm.pk1 as course_pk1 from course_main cm join course_contents cc on cm.pk1 = cc.crsmain_pk1 where cc.cnthndlr_handle like 'resource/bb-panopto-bc-mashup' group by cm.pk1) tbl_b on cm.pk1 = tbl_b.course_pk1
	left join (select COUNT(distinct(cc.pk1)) as panopto_place_link, cm.pk1 as course_pk1 from course_main cm join course_contents cc on cm.pk1 = cc.crsmain_pk1 where cc.cnthndlr_handle like 'hyperlink/coursecast' group by cm.pk1) tbl_c on cm.pk1 = tbl_c.course_pk1
	left join (select COUNT(distinct(cc.pk1)) as panopto_place_content_item, cm.pk1 as course_pk1 from course_main cm join course_contents cc on cm.pk1 = cc.crsmain_pk1 where cc.main_data like '%src="https://austincc.hosted.panopto.com/Panopto/Pages/Embed.aspx?instance=Blackboard%' and cc.cnthndlr_handle != 'resource/bb-panopto-bc-mashup' group by cm.pk1) tbl_d on cm.pk1 = tbl_d.course_pk1

where cu.role = 'P'
  and cu.row_status = '0'
  and cu.data_src_pk1 != '2'
  and t."name" like 'Summer 2024'
  --and cm.course_id LIKE '%224F%'
  and (tbl_a.panopto_place_embed > 0 or tbl_b.panopto_place_quiz > 0 or tbl_c.panopto_place_link > 0 or tbl_d.panopto_place_content_item > 0)
  
group by cm.course_id, cm.course_name, t."name", tbl_a.panopto_place_embed, tbl_b.panopto_place_quiz, tbl_c.panopto_place_link, tbl_d.panopto_place_content_item
```


### Panopto Building Block Content Individual Queries

If you need to only pull a specific set of data from Blackboard, you can use the queries below to find the information on the various building block types.

```

--- Panopto Quizzes
select distinct(cc.pk1), t."name" , cm.course_id, cm.course_name, cc.title, cc.cnthndlr_handle
from course_main cm
  join course_contents cc
   on cm.pk1 = cc.crsmain_pk1
  join course_users cu
   on cm.pk1 = cu.crsmain_pk1
  join users u
   on cu.users_pk1 = u.pk1
  join course_term ct
   on cm.pk1 = ct.crsmain_pk1
  left join term t
   on ct.term_pk1 = t.pk1
where cc.cnthndlr_handle like 'resource/x-bb-bltiplacement-panopto-quiz-lti'
order by t."name", cc.pk1


--- Panopto Building Block Mashup
select distinct(cc.pk1), t."name" , cm.course_id, cm.course_name, cc.title, cc.cnthndlr_handle
from course_main cm
  join course_contents cc
   on cm.pk1 = cc.crsmain_pk1
  join course_users cu
   on cm.pk1 = cu.crsmain_pk1
  join users u
   on cu.users_pk1 = u.pk1
  join course_term ct
   on cm.pk1 = ct.crsmain_pk1
  left join term t
   on ct.term_pk1 = t.pk1
where cc.cnthndlr_handle like 'resource/bb-panopto-bc-mashup'
order by t."name", cc.pk1


--- Panopto Building Block Hyperlinks
select distinct(cc.pk1), t."name" , cm.course_id, cm.course_name, cc.title, cc.cnthndlr_handle
from course_main cm
  join course_contents cc
   on cm.pk1 = cc.crsmain_pk1
  join course_users cu
   on cm.pk1 = cu.crsmain_pk1
  join users u
   on cu.users_pk1 = u.pk1
  join course_term ct
   on cm.pk1 = ct.crsmain_pk1
  left join term t
   on ct.term_pk1 = t.pk1
where cc.cnthndlr_handle like 'hyperlink/coursecast'
order by t."name", cc.pk1
