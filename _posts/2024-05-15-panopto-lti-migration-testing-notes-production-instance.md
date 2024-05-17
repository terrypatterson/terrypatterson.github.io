---
title: Panopto LTI Migration Documentation - Production - Testing Notes
date: 2024-05-15 12:00:00 -0500
categories: [LTI Migration, Panopto]
tags: [LTI, panopto, testing-notes, production-instance]
---

# Panopto LTI Migration
## Production Instance

## Migration Configure Notes

Changes were made starting at 9:20 AM CT

**Issue:** Blackboard Developer Portal blocks use of slash (/) in placement names, however we can set the placement name to include a slash once created in the Blackboard instance.

Panopto Conversion Script started at 10:02 AT CT

This migrates folders to have the LTI folders point to the folders previously created with the building block.




## Testing Notes

Please note that the notes below are raw and may not have fully fleshed out information at this time. They are a work in progress.

- [ ] Test logins via the hosted site and via Blackboard to make sure all is well.
- [ ] Test provisioning a new course folder to ensure you want the option Prepend Course Code set to True.
- [ ] Test the tools in the test courses.
- [ ] Test all the new tools.
- [ ] Test disabling tools from Tools section to see if the white glove treatment will still work.
- [ ] Test making quiz tool unavailable (if necessary) from placements to see if it the white glove treatment will still work.


### Testing Matrix

| | Building Block Created | LTI Created | Building Block (Copy) | Building Block (Import) |
|---------|----------------|-------------|-----------------------|-------------------------|
|Create Panopto Video Link | ✔️* | ✔️ | ✔️* | ✔️* |
|Create Panopto Video Embed (Content Item)| ✔️* | ✔️ | ✔️* | ✔️* |
|Create Panopto Video Quiz | ✔️* | ✔️ | ✔️* | ✔️* |
|Create Panopto Student Submission | ✔️* | ✔️ | ✔️* | ✔️* |


| | Building Block Created | LTI Created | Building Block (Copy) | Building Block (Import) |
|---------|----------------|-------------|-----------------------|-------------------------|
|Access Panopto Video Link | ✔️* | ✔️ | ✔️ | ✔️ |
|Access Panopto Video Embed (Content Item)| ✔️* | ✔️ | ✔️ | ✔️ |
|Attempt Panopto Video Quiz | ❌ | ✔️ | ✔️ | ✔️ |
|Attempt Panopto Student Submission | ✔️* | ✔️ | ✔️ | ✔️ |

\* Denotes that the item is broken, but behaving as expected according to Panopto Support.

**Information:** In previous LTI embed issues (Such as VidGrid) the course_pk1 value, _123456_1 for example, was hard coded into the HTML code if a video was embedded into a content item. We verified that if a Panopto video is embedded with the LTI tool in a content area, and the content item is copied to a new course, the course_pk1 value will change in the HTML code. This verifies that if you copy the content between courses it won't be an issue. However instructors should not try to copy and paste the embed into another text editor if they are using the Original Experience.


**Information:** Course copy v2, [which I mentioned in the Getting Started post](https://terrypatterson.github.io/posts/panopto-lti-migration-documentation/), creates reference links within the destination course folder in Panopto. Now there are some exceptions:

* Reference copies are only made from content that resides within the source course folder.
* Any content within a users folder will do something
* Panopto's course copy process does not kick off at the same time when a course copy runs within Blackboard. To start the process, the instructor should access the Panopto course folder. This can be done by either creating a tool link to the Panopto Course Folder tool or by clicking on the Panopto Course Folder Tool in the Tools area for Original users or in the Books and Tools or Content Market areas for Ultra Users. See the images below.

In Original...

![A cropped screenshot of a user creating a Panopto Course Folder tool link in the course menu of a Blackboard original course and cropped screen capture of the Panopto Course Folder tool link in the Tools area of a Blackboard original course.](assets/img/posts/panopto-lti/original-course-folder-links.png)

In Ultra...

![A cropped screenshot of a link to access the Panopto Course Folder tool in the Ultra Content Market and a cropped screenshot of a link to access the Panopto Course Folder tool from the Books and Tools area of an Ultra course.](assets/img/posts/panopto-lti/ultra-course-folder-links.png)



Once a Panopto Course Copy starts, it may take up to 24 hours. Our testing found that on a copy of about 20 videos normally took under an hour.

**Issue:** 

**Problem:** 