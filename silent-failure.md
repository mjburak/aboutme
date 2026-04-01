## Silent Failure

A customer reported an adapter fetch that only worked once after updating their access token. The logs showed a permissions error but no detail. I sent them the official documentation to confirm they were using the correct credential type. They said yes. I had them run two curl statements: the first generated a token, the second used it to hit the endpoint and returned a 200. When I asked them to run that second curl again with the same token, it came back 401. That proved something was wrong with the token's behavior on reuse.

I set up a Zoom with the customer and their remote platform admin, described what the curl output showed, and within minutes the admin identified that the customer had been generating a user token, which is only valid for one hour, instead of an app token. That's exactly what the documentation I'd sent specified, but easy to miss in practice. The adapter's fetch interval exceeded that window, so every scheduled fetch after the first was hitting an already-expired token. Correct token type, fetches ran cleanly from there.

[← Back to Home](README.md)
