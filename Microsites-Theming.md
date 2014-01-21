# Warning:
This is a draft document for a newly implemented feature in the edX platform, which still has a couple outstanding issues.

Please see PR #2061 for the latest comments and status
***
## Overview

This feature enables separate, deployable "micro-themes" (e.g. branding elements, Mako template overrides, etc.) for subdomains (e.g. foo.edx.org, bar.edx.org) in order to provide branding "multi-tenancy".

## Django configuration settings

MICROSITE_CONFIGURATION
MICROSITE_NAMES
MICROSITE_ROOT_DIR
VIRTUAL_UNIVERSITIES
SUBDOMAIN_BRANDING
FEATURES('USE_MICROSITES')
FEATURES('ENABLE_MKTG_SITE')

## Template Changes
