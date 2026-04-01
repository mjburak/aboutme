## Navigated a Multi-Vendor Bug

Multi-vendor issues were a weekly occurrence at Axonius. A customer would report an adapter fetch failure, but the real problem was in their identity layer. Some of the older adapters only surfaced a generic exception error in the UI with no detail. I'd pull the customer's logs, find the 401 underneath, and work through the token and permissions configuration on the Axonius side to rule that out. Once I'd confirmed the adapter itself was healthy, I'd bring in their Okta admin to look at the access token and permissions on their end. Nine times out of ten that's where the fix lived: a misconfigured token, a scope that hadn't been granted, or a service account that had been deactivated.

[← Back to Home](README.md)
