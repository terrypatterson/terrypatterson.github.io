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

## Script Review

The first thing I need to do is call on several libraries that will help me navigate the issue of getting to the file without requiring the script runner to input the absolute filepath. 

```python

import requests
import pathlib
import os
import shutil
import pandas as pd
from datetime import datetime

# Update the following value when terms change, "Spring 2024" for example.

term = "TERM YEAR"

# datetime object containing current date and time
now = datetime.now()

```
This part of the script remains the same. The bbrest library has been a great library for this script, much appreciation and respect to [Matthew Deakyne](https://github.com/mdeakyne) for his work on developing and supporting this library. The picture below is myself and Matthew at Blackboard DevCon in 2019 with our buddy, Mark Reynolds sneaking into the picture.

[![A photo of three white males, Terry Patterson and Matthew Deakyne lean on each other back to back while another man, Mark Reynolds sneaks into the photo from the left side ](https://live.staticflickr.com/65535/48353180187_42aa56b3e0_w.jpg)](https://www.flickr.com/photos/terrypatterson/48353180187/in/album-72157709819185206)

```python

now = datetime.now()

from bbrest import BbRest
key = [REST API KEY]
secret = [REST API SECRET]
learnfqdn = [BLACKBOARD INSTANCE]
bb = BbRest(key, secret, "https://"+ str(learnfqdn) +"")  # Does a lot! Get the system version, pull in the functions from dev portal, 2-legged authentication w/ caching of token.

# Create a log file for the day's run
# Create the file name.
log_file_name = "bb-rest-enrollment-testing-log-" + now.strftime("%Y-%m-%d") + ".txt"
log_file_in_sub = "logs\\" + log_file_name

```

```python

downloaded_enrollment_csv = "Tutor _ Embedded Coach Enrollments - " + term + ".csv"
enrollment_input_dir = "C:\\Users\\username\\Downloads"
enrollment_feed_dir = "D:\\python-scripts\\uploaded-files"
downloaded_full_enrollment_csv = os.path.join(enrollment_input_dir, downloaded_enrollment_csv)
enrollment_preprocess_dir = "D:\\python-scripts\\processing"

```