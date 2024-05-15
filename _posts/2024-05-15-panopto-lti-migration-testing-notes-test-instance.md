---
title: Panopto LTI Migration Documentation - Test Instance - Testing Notes
date: 2024-05-15 12:00:00 -0600
categories: [LTI Migration, Panopto]
tags: [LTI, panopto, testing-notes, demo-instance, test-instance]
---

# Panopto LTI Migration
## Demo / Test Instance


## Testing Notes

Please note that the notes below are raw and may not have fully fleshed out information at this time. They are a work in progress.

- [ ] Test logins via the hosted site and via Blackboard to make sure all is well.
- [ ] Test provisioning a new course folder to ensure you want the option Prepend Course Code set to True.
- [ ] Test the tools in the test courses.
- [ ] Test all the new tools.
- [ ] Test disabling tools from Tools section to see if the white glove treatment will still work.
- [ ] Test making quiz tool unavailable (if necessary) from placements to see if it the white glove treatment will still work.

**Information:** Creation of the Panopto Folder now happens when you click on the Panopto Course Tool Link Folder. The three page workflow with the building block has changed to provisioning the folder when you click the LTI Link.

**Issue:** When in the Panopto Course Folder - Clicking the close button in the upper right-hand corner throws an error.

**Problem:** The Panopto Video Embed Tool does not embed the video. It only creates a link within the course based on testing.

What is the proper way to create a Video Embed within the Blackboard Learn application? If embedded within a course item and copied, will the permissions sync over for the embed within the course item.

**Information:** New workflow on quiz creation process

**Issue:** Panopto Quiz defaults to 100, would like to have the ability to set a points possible in the workflow

**Information:** Panopto Student Submission - Change from Video link to Video Embed within the Create Submission. 

**Issue:** Student submission videos don’t appear to reside in the student’s Panopto My Folder, this doesn’t appear to be the case. (Follow-up: Review student’s My Folder requirements / needs with Mario)

**Issue:** Capture window did not auto-close after upload. (See screenshot below) Document, that is expected behavior.

**Issue:** Panopto Student Submission Tool appears within the Build Content area which it shouldn’t as it should only be available under the plus icon in the Text Editor. How can we properly hide that?

**Issue:** Course Copy v2 - Delays in the course copy process? Seems to have taken over 20 minutes for a course copy to appear in the backend of Panopto, should those delays be expected and shouldn’t those 

Video was uploaded into 18 and copied via the item copy feature within Blackboard to section 21. Video does not exist in the 21 Panopto folder nor did the permissions get updated to allow 18 folder content to be accessed by Section 21 viewers. 

Made the Blackboard Building Block Unavailable

Links in Panopto LTI Testing 001 all worked successfully

Copied Panopto LTI Testing 001 to section 025
- Quiz Link breaks
- Embedded Panopto Videos and Panopto Video Links permissions not there
- Content was not copied


Course Copies made before Panopto LTI Migration
- Quiz Links break
- Embedded Panopto Videos and Panopto Video Links permissions are not there for users
- Referenced content copies don’t reside in the destination course. Support teams will have to manually copy content from Panopto folder in source course over to destination course.
- Student Submissions Still Work

Course Copies made after Panopto LTI Migration
- Quiz Links break
- Embedded Panopto Videos and Panopto Video Links permissions are not there for users
- Panopto copied referenced Panopto content from source course into the destination course’s Panopto folder.
- Student Submissions Still Work
