## The 500 That Wasn't

At Axonius, a customer reported that an adapter was failing, but the UI only showed "Unhandled exception," which gave them nothing to act on. There was a 500 in the logs, and the first read was a server-side problem on the remote platform. The customer had been told the issue was on their end, and they weren't satisfied with that answer. A 500 is the standard read for a website, but APIs don't always play by those rules, so I didn't take it at face value.

I pulled the logs in Coralogix, found the 500, and reviewed every entry around that timestamp. Nothing out of the ordinary, which told me the answer wasn't going to be in the logs alone. Since this customer was hosted by Axonius, I connected to their instance and ran curl against the same endpoint the adapter was hitting, with -v so I could see the full request headers and not just the response. Comparing the request against the remote platform's API documentation, I found it: the path was truncated. The remote API wasn't failing. It was rejecting a malformed request we were sending.

I escalated to Engineering with a direct link to the exact Coralogix record and the relevant API documentation, so they could go straight to the fix instead of re-investigating from scratch. Since it was a one-line change, they had a fix available within the hour. The customer approved the deployment and the adapter ran to completion.

[← Back to Home](README.md)
