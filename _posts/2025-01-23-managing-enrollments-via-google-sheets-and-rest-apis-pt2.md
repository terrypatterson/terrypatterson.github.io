---
title: Managing Enrollments with Google Sheets and Rest APIs (Part 2)
date: 2025-01-23 09:00:00 -0600
categories: [Python, Rest APIs, Enrollments]
tags: [documentation, enrollments, rest api, python, scripting]
---

![Image of the Wong Ranch from Futurama. A two story manison sits in the red dirt of Mars in the background. In the foreground, a fence with a square arch holds a sign above the entrance saying "Wong Ranch" a sign hanging from the arch says "You've Come to the Wong Place"](assets/img/posts/python-rest-api-enroll/wong-ranch.png)

## Last Time at the Documentation Ranch...

 In Fall 2022, a team that manages the enrollment of tutors, supplemental instructors, and academic coaches approached the department with the request to help them enroll these users into their courses at the start of the coming Spring 2023 term. They had been keeping their enrollments in a Google spreadsheet (aka Google Sheets) and wanted to know if we could process this file. I was working on my Python certification so automating this with a Python script seemed like a great way to test out my python scripting skills and complete their task.

## Items to Improve

 The first iteration met the basic needs/requirements for the team. Enrollments were processed, but the interface was "clunky" and not processing everything easily. Things could be done better so I wanted to make some improvements.

 1. Improve the way the system feeds a file into the script. Would be great if the script could just grab the file from Google.
 2. Remove the trailing commas that come over in each line.

**Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.**

## 
