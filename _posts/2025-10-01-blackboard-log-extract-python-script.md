---
title: Improving Blackboard Log Expansion with a Python Script
date: 2025-10-01 09:00:00 -0600
categories: [Log Management, Blackboard Admin]
tags: [documentation, log management]
---

# Introduction

Ever since moving to SaaS environments, Blackboard administrators have had a tough time trying to find issues or problems that require pulling information only found in the logs of the Blackboard application. The process was tedious as you had to expand each hour and when you were facing looking at multiple hours (or worse, an entire day) it was going to be a tedious process.

My old process was something like this...

1. Go to my Blackboard instance and under the Admin area, go to Manage Content.
2. Under internal > logs in the Content Collection you find the storage for the compressed log files. For example, September 17, 2023 would be found under the folders 2023 > 09 > 17.
3. Under the date folder then you have 24 subfolders which contain compressed .gz files that hold the systems log files for each application node/server.

The script provided by Anthology/Blackboard would required you to have a folder that contained the compressed log files and then a different destination folder for the expanded log file structure.

```cmd
convertlogs.py -f C:\Users\UserName\Downloads\Bb-Logs\09-17-2023\11 -o C:\Users\UserName\Desktop\Learn-Logs\09-17-2023\11
```

The admin has to create multiple folders and run this command multiple times. Just in general an inefficent process when trying to expand multiple hours of logs to find an issue.

So taking this code and my pre-existing processes for log management and created a brand new script that does the following:

<div style="border: 1px solid; padding:10px; font-style: italic; font-weight: bold;">
Note: This is a living document, some information maybe incorrect or incomplete. Please contact me if you have any updates and/or corrections.<br /><br />
I use Artifical Intelligence services (ChatGPT, Claude, Microsoft Copilot, Google Gemini, and  Perplexity) to help me plan, develop, and troubleshoot documentation and scripts shared on this site.
</div><br />

So this was built to delete courses in small batches so it wouldn't overtax the environment nor block the processing of other tasks such as course copies or imports. It also removed me from having to babysit the process so I could enjoy weekends and/or vacations.

## Running the Query

To find cross-linked content within courses, we use the following query. Note that a major part of the query is commented out. This query is too big to run as a complete process on our database. So we are going to break each part out into sections.

Each run, we want to remove the -- from in front of a query within the from ( ). In this example we want to only run the query that will help us find cross-linked content that is attached within courses. So we convert this section:
