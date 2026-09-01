# The Job I Automated Myself Out Of

At eBay, I was handed a report twice a day: every server that had dropped out of rotation. My job was to figure out what was wrong, fix it, and put it back to work. Some days that report had 300 to 400 servers on it.

The company's UI could only handle one server at a time. Type in a name, wait 20 to 30 seconds for it to load, read the diagnostics, move to the next. Do that 300 to 400 times and you've burned the entire day just waiting on a spinner.

Nobody I worked with had touched bash. They'd been trained on Python, taught that everything should be an object whether it'll ever be reused or not. So the slow UI was just how the job worked. Nobody questioned it because nobody had another tool in their pocket to question it with.

I did. I wrote a separate bash script for each check the UI was running, ping, SSH, the rest. Each one wrapped a plain Linux command in a loop that read server names off a list. The loop wasn't doing anything clever. It just kept feeding names into the same command until the list ran out. All the actual work was the command itself.

I could open a window for each script and run the full list of 300 to 400 servers in seconds instead of hours. Then I'd scan the output for the smoking gun. What used to take all day took two hours, sometimes less.

Then I kept going. Over time I noticed some of the checks never produced anything meaningful. So I cut them. I kept whittling until I was down to two rules that covered everything:

- Can't ping it? Reboot it.
- Can't SSH to it? Destroy the VM and build a new one.

I ran on just those two rules for a month with no issues. Told my boss. He wanted another month of data before he'd believe it. Still no issues.

At that point I wrote it up and sent it to the Puppet team with a recommendation to automate the whole thing. Which they did. That job, the one that had been the dreaded rotation for years, stopped needing a person.

I didn't do it because I'm the smartest engineer in the room. I did it because I got frustrated sitting there watching a loading spinner, and I already had a tool that could fix that. The rest was just refusing to keep doing more work than the problem actually required.
