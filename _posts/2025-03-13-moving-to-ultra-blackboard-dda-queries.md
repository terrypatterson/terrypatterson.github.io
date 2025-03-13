---
title: Blackboard DDA Queries
date: 2025-03-13 09:00:00 -0600
categories: [SQL, DDA]
tags: [SQL, DDA, queries, Ultra]
---

Here are some Blackboard DDA Queries that I've developed during our migration to Ultra.

> Please note these queries may be ineffenient, feel free to make improvements and provide feedback.

## What deployed tests are using rubric associations with test questions

Blackboard Original Experience allowed the instructor the ability to associated a rubric with an essay question within a test. Ultra does not have this feature at the time we were implementing the course experience, therefore we needed to understand the instructor impact and document it for internal communications and communications with Anthology. Here's the query I put together for it.

```SQL

select distinct test.pk1 as "Test PK1" /* This ensures that there isn't any duplication of tests within a course. I found results were showing duplicates (probably an incorrect join) */
  , term.name as "Term"
  , course_main.course_id as "Course ID"
  , array_to_string(regexp_matches(encode(test.data::bytea, 'escape'), '<assessment title="(.*?)"','g'), '') as "Test Title"
  , rqcount.qcount as "Number of Rubric Questions"
  , case when course_assessment.pk1 is null then 'not deployed' else 'deployed' end as "Test Deployment Status" /* I limited the scope of the results to only show deployed tests. */
	from qti_asi_data as test
	left outer join course_assessment on course_assessment.qti_asi_data_pk1 = test.pk1
	inner join course_main on course_assessment.crsmain_pk1 = course_main.pk1
	left outer join course_term on course_main.pk1 = course_term.crsmain_pk1
    left outer join term on course_term.term_pk1 = term.pk1
	inner join qti_asi_data as section on section.parent_pk1 = test.pk1
	inner join qti_asi_data as question on question.parent_pk1 = section.pk1
	inner join (select COUNT(question.pk1) as qcount, test.pk1	as test_pk1 from qti_asi_data as test left outer join course_assessment on course_assessment.qti_asi_data_pk1 = test.pk1 inner join course_main on course_assessment.crsmain_pk1 = course_main.pk1 inner join qti_asi_data as section on section.parent_pk1 = test.pk1	inner join qti_asi_data as question on question.parent_pk1 = section.pk1 where question.pk1 IN (select qti_asi_data.pk1 from rubric r join rubric_association ra on r.pk1 = ra.rubric_pk1 join association_entity ae on ra.association_entity_pk1 = ae.pk1 join qti_asi_data on ae.qti_asi_data_pk1 = qti_asi_data.pk1) group by test.pk1) as rqcount on test.pk1 = rqcount.test_pk1
	where question.pk1 IN (select qti_asi_data.pk1 from rubric r join rubric_association ra on r.pk1 = ra.rubric_pk1 join association_entity ae on ra.association_entity_pk1 = ae.pk1 join qti_asi_data on ae.qti_asi_data_pk1 = qti_asi_data.pk1)

```