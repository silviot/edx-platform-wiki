_Work in progress..._

## Update

Here's the PR for the hackathon project: https://github.com/edx/edx-platform/pull/4079
Here are the changes for once the PR is closed: https://github.com/edx/edx-platform/compare/master...hackathon;andya;xblock-admin-pages

## Overview

Currently, the only way to introduce new UI into Studio is either (a) xblocks to add new modules or (b) coding directly into the platform. In discussions with other groups, they all would like to plug in Studio capabilities without touching the code base. My idea is that we should think of xblock as being a platform plugin capability, rather than purely a way to build components for units. 

I was thinking about providing a mechanism to declare a global editing/configuring view in xblock, and then building a Studio preferences page that lets you navigate to them. These views would be global scoped (or maybe course scoped, or maybe both) and would let the course developer work across their units. For example, ORA right now shows a lot of staff features inline inside their component, but they would rather have a full screen experience where the course author doesn't have to drill down into a particular unit to find it. There's an interesting question as to whether these global admin views should be run in LMS (i.e. from the instructor dashboard) or from Studio, or whether we should support both. I'm obviously more interested in Studio, but I think instructor dashboard plugins would be an equally useful use case.

The basic approach will be to support a new ```admin_view``` method on xblocks that will be invoked in a global scope. This means that the view won't be able to access settings and content fields, but will be able to access globally scoped fields. It can obviously also access databases and/or services that the xblock uses for external storage. 

What I'm imagining for Studio would then be something like the Mac's System Preferences page. It would enumerate all of the xblocks that have registered admin pages (say ORA and Forums), and clicking on the name would open a full page rendering of the admin view for that block. We could also do the same thing for the Instructor Dashboard, although I would need to find a volunteer to help if we wanted to do it for the hackathon.

## Use cases

### Use Case 1: ORA

Implementation:
* Student submission xblock already implemented
* ORA lives in its own repo, and is included in edx-platform through a GitHub hash reference
* ORA drops in Python apps which requires some small amount of platform configuration
* ORA has back-end services that manage all of its own data

Requirements:
* All ORA configuration UI is done through the Studio xblock editor because there is no other option
* [[https://edx-wiki.atlassian.net/browse/TIM-512]] - Course authors can create and configure student assessor training
* [[https://edx-wiki.atlassian.net/browse/TIM-520]] - Authors can create and calibrate an AI assessment for student use

### Use Case 2: Forums

* Needs two UIs in LMS
* Inline discussions within a unit
* Discussions page available through a tab on the course
* Also requires other UIs
* allow course developer to define discussion hierarchy
* notification scheme in LMS so that new comments to a thread are surfaced
* admin UIs for managing the backend services
* possibly need capabilities added to the instructor dashboard

### Use Case 3: A/B Test

Note: A/B test was implemented as an xmodule because the necessary pieces were not available in time

Two parts required:
1.  Define an experiment and how the students will be broken into groups
2. When defining the course, allow a block to be added that defines the alternate experiences for each group

Part 2 is more straightforward but has challenges in Studio:
* The course developer can add an experiment block to their unit whenever they want a split experience
* Studio needs to support xblocks that have arbitrary children
* The course developer needs to be able to move items from one group to another

Part 1 is more challenging:
* Is an experiment a first-class xblock itself, or is it just course-scoped configuration for the xblock
* Studio doesn't have a good place for a central place to manage the experiments for a course
* This seems like managing textbooks, assets etc, but those are hard-coded pages built into the platform

Unique challenges
* Some A/B tests would like to work outside the scope of a unit, e.g. week 2 is different for group A and group B
* What if experiments want to work across courses and/or course runs?
