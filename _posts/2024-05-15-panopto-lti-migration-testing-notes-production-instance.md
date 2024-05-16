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
|Create Panopto Video Link | | | | |
|Create Panopto Video Embed (Content Item)| | | | |
|Create Panopto Video Quiz | | | | |
|Create Panopto Student Submission | | | |


| | Building Block Created | LTI Created | Building Block (Copy) | Building Block (Import) |
|---------|----------------|-------------|-----------------------|-------------------------|
|Access Panopto Video Link | | | | |
|Access Panopto Video Embed (Content Item)| | | | |
|Attempt Panopto Video Quiz | ❌ | ✔️ | | |
|Attempt Panopto Student Submission | ✔️ | | |



**Information:**

**Issue:** 

**Problem:** 