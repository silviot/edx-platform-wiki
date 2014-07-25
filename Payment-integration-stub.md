@codygman Can I ask for a bit more detail about the particular paypal payment flow that this integration uses?  A bit of background might be useful here.

The original Cybersource integration for Stanford had to use a particular integration (Hosted Order Page) provided by Cybersource, which influenced the abstract processor interface that was designed (which means it's not set in stone, btw).  HOP integration is fairly old-fashioned.  It starts with a cart object on edx-platform which has a bunch of items.  HOP then does the following steps:
1. 