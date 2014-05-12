# The Problem

The "Legacy" (currently the default) instructor dashboard is a mess. "legacy.py" contains 2k lines of code, not nicely split up into functions. It has few comments and many pylint/pep8 violations. The code is very hard to maintain for us.

On the "edx.org" website, we currently encourage users to use the "Beta" dashboard. All of our documentation recommends using this, and now that we've reached near-parity in the instructor dashboards – at least, for the features used on the edx.org website – we wish to switch the two dashboards. The end-goal of this is to remove the "legacy.py" file from our codebase.

However, we recognize that removing the legacy dashboard wholesale is not a good solution for partners that may have extended this dashboard to suit their own various needs. Thus, we propose the following multi-phase solution, that hopefully will advance the goals of edx.org (removing messy, untested, hard-to-maintain code) while still preserving the needs of our partners and those who have extended our platform.
 
# The Solution

Solving this problem will consist of a couple phases:

1. Switch the positions of the "Legacy" and "Beta" dashboards, referring now to the "Beta" dashboard as the "Instructor Dashboard".
2. Remove functionality from the "Legacy" dashboard that is duplicated in the Instructor Dashboard.
3. Rename "Legacy" dashboard to "Advanced". 

## Phase 1

Switch the positions of the "Legacy" and "Beta" dashboards, referring now to the "Beta" dashboard as the "Instructor Dashboard". Change URL patterns such that the default /instructor pointer points to the new dashboard; bookmarks for the dashboard will thus now point to the new dash. On edx.org, banners will appear alerting users that the legacy dashboard will go away. Those who have forked the platform are welcome to also use this banner, or reword it as they wish (for example, explaining what features will remain and what will be deleted once Phases 2 & 3 are complete). A banner welcoming users to the Instructor Dashboard will appear for a time as users adjust.

The "Legacy" dashboard is put behind a feature flag, "ENABLE_INSTRUCTOR_LEGACY_DASHBOARD", for the moment set to "True" on edx.org. edx.org will turn this dashboard off after a transition period (we will monitor usage metrics to determine when it's reasonable to disable).

edx.org will no longer accept upstream contributions to "legacy.py" or any other part of the legacy dashboard.

### Note

Phase 1 is already nearly complete; see #2865 

## Phase 2

Remove functionality from the "Legacy" dashboard that is duplicated in the Instructor Dashboard. This involves ripping out huge sections of code in legacy.py, and editing associated templates, removing some tabs completely. Possible refactoring of the remaining code, depending on difficulty, should occur in this phase.

## Phase 3

Rename  "Legacy" dashboard to "Advanced". This will allow installations to preserve their existing dashboard extension points, until they can port them to the Instructor Dashboard.

We will not remove the legacy.py file or remaining templates from the codebase; however, we will indicate on the files that we are no longer developing the code or accepting contributions to it. 