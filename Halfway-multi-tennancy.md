Instead of doing true multi-tennancy, this is an interim solution that will allow us to have multiple instances sharing much of the infrastructure, without fully duplicated instances.

Requirements
------------

Goals
1. Ability to support different themes
2. Data segregation
3. Done with what we have now

Non-Goals
1. Different code bases -- we want to install the same code to all instances
2. Shared everything
3. Full isolation

Proposal
--------

![multi-pre.png]

![multi-post.png]