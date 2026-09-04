# Pairing with AI on a Real Data Migration Problem

**What it actually looks like to build something real with AI, including the parts that didn't work.**

## The ask

A friend reached out for help with his Shopify and Etsy shops. He'd recently purchased a commercial license to sell 3D-printed items from a vendor, and needed the titles, descriptions, and images pulled from the vendor's site into his own shops. The vendor's catalog listed 557 items across 16 categories. I told him, "no problem."

## Scoping the problem

I spent about 20 minutes getting familiar with the vendor site. The item pages themselves had no usable descriptions, just a "Join Discord" button. So I copied one item's name, searched for it, and found 4-5 listings for the same product scattered across other sites. I fed those descriptions to Gemini, which built a reusable template that only needed the title and category swapped per item.

I also noticed the vendor's image URLs followed a clean, predictable pattern: `[url]/1.jpg`, `[url]/2.jpg`, and so on. That was going to matter a lot for how this got automated.

## Round one: Gemini

I asked Gemini to help me collect titles, categories, and image URLs from the vendor site. Its plan required me to do all the heavy lifting by hand, including downloading and re-saving every single image. My friend doesn't run an image server, so linking directly to the vendor's existing URLs was obviously the more efficient path. I said as much. Gemini dug in and insisted on the download-everything approach anyway, even after I explained the constraint again.

I've learned that when an AI won't budge on a technical approach that doesn't fit the actual situation, that's the signal to stop arguing and try something else.

## Round two: Claude

I gave Claude the same request. Its proposed approach matched what I actually needed: a JavaScript scraper I could paste into the browser console, running in my own logged-in session, that would pull titles and image URLs directly from each category page rather than downloading anything.

The workflow that emerged:

1. Load a category page.
2. Open Chrome DevTools, switch to Console, paste the script.
3. The script auto-scrolled to load every item, then visited each product's own page to grab its full image gallery.
4. Output: a CSV of everything successfully scraped, and a separate CSV of anything skipped, with a reason.

Going through all 16 categories this way took about 40 minutes.

## Chasing down the skipped items

The skipped list all said "no gallery images found," but the actual causes varied:

- Some products used image filenames like `one.jpg` instead of `1.jpg`.
- Some hosted images on an entirely different CDN subdomain than the rest of the catalog.
- Some linked directly to the CDN instead of through the resize proxy the rest of the site used.
- One approach I tried (matching everything up to a "Community Prints" section heading to avoid scraping customer photos) turned out to be unreliable, that exact phrase sometimes appeared early in the page as a stray JavaScript comment, cutting off the real product images before the regex ever saw them.

Each time something failed, Claude proposed a specific, testable hypothesis. I'd paste a small diagnostic snippet into the console, report back exactly what it printed, and we'd narrow it down from there. It took over an hour of this back-and-forth to get every skipped item accounted for.

## Merging into two very different formats

Etsy and Shopify don't share a CSV schema, so this needed two separate outputs. Shopify publishes extensive documentation on its import format, so Claude built a merge script around that directly. Etsy doesn't offer a blank import template at all, only a bulk *export* of listings already in the shop. That export was enough to reverse-engineer Etsy's real column structure and build a second conversion script targeting it.

## Where automation should stop

We looked for a native bulk-upload option on Etsy and came up empty, it turns out one doesn't exist for creating new listings, only third-party tools do that. Rather than force a workaround, we mapped out the real options together, and I passed my friend a clear recommendation, including a specific warning: only use a third-party tool that connects via Etsy's actual OAuth login, never one asking for his Etsy password directly.

## Result

- **16** categories
- **369** items
- **6,691** images
- Two correctly formatted import files, ready for pricing, built from what would have otherwise been weeks of manual copy-paste

My friend handled the final uploads himself. What was originally agreed at $15/hour turned into more, once he saw five hours of work replace what would have been a month of tedious manual entry on his end.

## The actual takeaway

The value here wasn't "AI did it for me." It was pairing with a fast, capable collaborator willing to propose a hypothesis, read the actual error output, and change course, the same iterative process good debugging always looks like, just compressed into an evening instead of days.
