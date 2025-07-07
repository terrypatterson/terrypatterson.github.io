---
title: Finding and Identifying Cross Linked Course Content
date: 2025-07-02 09:00:00 -0600
categories: [Cleanup, Orphaned Content, Cross Linked]
tags: [documentation, course removal]
---

# Introduction

Course deletions have always been a difficultly in Blackboard. If you were self-hosted and didn't have enough database or application server power, your system could go down with the simple deletion of some courses. (I know, I did it.). Then even in managed hosting or SaaS environments, deleting thousands of courses could delay course copies or imports for hours while the task workers processed these requests. So how can you delete courses without having either impact? This is why I wrote this script.

**Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.**

So this was built to delete courses in small batches so it wouldn't overtax the environment nor block the processing of other tasks such as course copies or imports. It also removed me from having to babysit the process so I could enjoy weekends and/or vacations.



## Running the Query

To find cross-linked content within courses, we use the following query. Note that a major part of the query is commented out. This query is too big to run as a complete process on our database. So we are going to break each part out into sections. 

```SQL

select 
	content.course_pk1
	, content.course_id
	, case 
		when position('internal/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/'||split_part(xyf_files.full_path, '/', 4)||'/' 
		when position('courses/' in xyf_files.full_path) > 0 or position('orgs/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/' 
		else '/'||split_part(xyf_files.full_path, '/', 2)||'/' 
		end as course_home
	, count(*) as embed_count
	, content.cnthndlr_handle as embed_type
from (
	--select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
	--	from course_contents_files
	--	left outer join files on course_contents_files.files_pk1 = files.pk1
	--	left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 
	--union
	--	select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Embedded Content' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, array_to_string(regexp_matches(course_contents.main_data, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from course_contents
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1
	--	where course_contents.main_data like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, announcements.pk1 as content_pk1, 'Announcement' as cnthndlr_handle, announcements.subject as content_title, course_main.course_id, array_to_string(regexp_matches(announcements.announcement, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from announcements
	--	left outer join course_main on announcements.crsmain_pk1 = course_main.pk1
	--	where announcements.announcement like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, qti_asi_data.pk1 as content_pk1, 'Assessment' as cnthndlr_handle, qti_asi_data.title as content_title, course_main.course_id, array_to_string(regexp_matches(encode(qti_asi_data.data::bytea, 'escape'), 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid 
	--	from qti_asi_data 
	--	left outer join course_main on qti_asi_data.crsmain_pk1 = course_main.pk1  
	--	where encode(data::bytea, 'escape') like '%bbcswebdav%' 
	--      and course_main.data_src_pk1 IN (PK1) 
	--union
	--	select course_main.pk1 as course_pk1, forum_main.pk1 as content_pk1, 'Discussion Forum' as cnthndlr_handle, forum_main.name as content_title, course_main.course_id, array_to_string(regexp_matches(forum_main.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from forum_main
	--	left outer join conference_main on forum_main.confmain_pk1 = conference_main.pk1 
	--	left outer join course_main on conference_main.crsmain_pk1 = course_main.pk1 
	--	where forum_main.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, msg_main.pk1 as content_pk1, 'Discussion Post' as cnthndlr_handle, msg_main.subject as content_title, course_main.course_id, array_to_string(regexp_matches(msg_main.msg_text, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from msg_main
	--	left outer join forum_main on msg_main.forummain_pk1 = forum_main.pk1 
	--	left outer join conference_main on forum_main.confmain_pk1 = conference_main.pk1 
	--	left outer join course_main on conference_main.crsmain_pk1 = course_main.pk1 
	--	where msg_main.msg_text like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, blogs.pk1 as content_pk1, 'Blog or Journal' as cnthndlr_handle, blogs.title as content_title, course_main.course_id, array_to_string(regexp_matches(blogs.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from blogs
	--	left outer join course_main on blogs.crsmain_pk1 = course_main.pk1 
	--	where blogs.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, blog_entry.pk1 as content_pk1, 'Blog Entry' as cnthndlr_handle, blog_entry.title as content_title, course_main.course_id, array_to_string(regexp_matches(blog_entry.description, 'bbcswebdav([^\s"''<\)]+)','g'), '') as xid
	--	from blog_entry 
	--	left outer join blogs on blog_entry.blog_pk1 = blogs.pk1 
	--	left outer join course_main on blogs.crsmain_pk1 = course_main.pk1 
	--	where blog_entry.description like '%bbcswebdav%'
	--union
	--	select course_main.pk1 as course_pk1, '0' as content_pk1, 'Banner' as cnthndlr_handle, 'Banner' as content_title, course_main.course_id, case when substr(course_main.banner_url,1,1) = '/' then course_main.banner_url else '0' end as xid
	--	from course_main 
	--	where course_main.banner_url is not null
) as content
left outer join (
	select *
	from dblink('dbname=DATABASE_cms_doc host=localhost sslmode=require user=USERNAME password=PASSWORD', 'select xyf_files.entry_id, xyf_urls.full_path from xyf_files left outer join xyf_urls on xyf_files.file_id = xyf_urls.file_id')
	as xyf_files(entry_id int, full_path text)
) as xyf_files on content.xid = '/xid-'||cast(xyf_files.entry_id as text)||'_1'
group by content.course_pk1, content.course_id, case when position('internal/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/'||split_part(xyf_files.full_path, '/', 4)||'/' when position('courses/' in xyf_files.full_path) > 0 or position('orgs/' in xyf_files.full_path) > 0 then '/'||split_part(xyf_files.full_path, '/', 2)||'/'||split_part(xyf_files.full_path, '/', 3)||'/' else '/'||split_part(xyf_files.full_path, '/', 2)||'/' end, content.cnthndlr_handle

```

Each run, we want to remove the -- from in front of a query within the from ( ). In this example we want to only run the query that will help us find cross-linked content that is attached within courses. So we convert this section:

```SQL

	--select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
	--	from course_contents_files
	--	left outer join files on course_contents_files.files_pk1 = files.pk1
	--	left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
	--	left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 

```

to this

```SQL

	select course_main.pk1 as course_pk1, course_contents.pk1 as content_pk1, 'Attached' as cnthndlr_handle, course_contents.title as content_title, course_main.course_id, files.file_name as xid
		from course_contents_files
		left outer join files on course_contents_files.files_pk1 = files.pk1
		left outer join course_contents on course_contents_files.course_contents_pk1 = course_contents.pk1
		left outer join course_main on course_contents.crsmain_pk1 = course_main.pk1 

```

We only comment that section out of the query. Now run the query. Once complete, you should only get the values associated with the attached files in the course. will appear like this.

| Course PK1 | Course ID | Course Home Directory | Count of Items | Type of Content |
| ---------- | --------- | --------------------- | -------------- | --------------- |
|937640|8251-000000-EMSP-2260-001|/courses/8251-000000-EMSP-2260-001/|11|Attached|
|912160|4567-000000-DFTG-1433-DLS-16wk|/courses/4567-000000-DFTG-1433-DLS-16wk/|7|Attached|
|879799|1237-000000-UXUI-1370-004|/courses/1237-000000-UXUI-1370-004/|15|Attached|
|933083|123R-000000-GERM-2311-001|/courses/123R-000000-GERM-2311-001/|8|Attached|
|920993|593F-000000-CHEM-2123-002|/courses/593F-000000-CHEM-2123-002/|9|Attached|
|888005|964S-000000-ITSC-2337-003|/courses/964S-000000-ITSC-2337-003/|37|Attached|

So with these results, how can you tell what content is cross-linked. That's a good question. If the Course ID does not match the Course Home Directory, the content is cross-linked. We will manipulate this content later in the steps, but make sure your output comes out like this.

When you run each section, save the output into a folder (such as 2024-cross-linked-content-data) as a CSV file.

![Screenshot of a list of files in Windows file manager](assets/img/posts/cross-linked-content/cross-linked-content-data-image.png)

