# Two Claudes, one repo

*Draft, 2026-06-11. Byline: BlonDev. Edit freely.*

Tonight I found out I'd been running two AI coding sessions on the same task, at the same time, without knowing it. Nothing broke. That's the interesting part.

## The setup

I run a small product with a 2–3 person team and several Claude Code sessions going at once — some in windows I'm watching, some as background jobs I check on later. The sessions share two things: the repos on disk, and a persistent memory directory where they leave notes for each other across conversations. "Here's where the rollout stands. Here's the pending execution list. Here's what's still open."

That memory is what makes "pick up where we left off" work after a disconnect. It's also what makes the following possible.

## The incident

This evening, around 7pm, a session I'd been working with got disconnected. I opened a new one and typed: *"hi we were disconnected. can we pick up where we left off?"*

Around 9:30pm — having lost track of the first recovery — I opened *another* one and typed, essentially, the same thing: *"cool. can we pick up from the last session? we were disconnected."*

Both sessions read the same memory note. The note contained a pending to-do list: merge these two PRs, close that one with an explanation, delete a directory of process machinery we'd decided to drop, commit the retro doc. Both sessions set off to execute it.

Two agents. One to-do list. No locks, no queue, no awareness of each other.

## Why nothing broke

The first session got there two hours earlier and did the work: merged, closed, committed, pushed.

The second session did the thing that saved the evening: before executing anything, it checked *reality* — the live PR states on GitHub, the actual repo contents — rather than trusting the remembered plan. It found every box already ticked, stood down, and reported back, deadpan: *"previous session finished in parallel."*

When I asked the first session whether two sessions could really have been racing, it went and pulled the records: job state files, file modification times, PR timelines. One close-comment on the PR, not two. No double merges. The memory files had been written by both sessions — last writer wins — but the content happened to converge, because both were describing the same finished reality.

So: an accidental distributed system, rescued by two properties. The external actions were **idempotent-ish** (merging an already-merged PR is an error, not damage), and both agents had the habit of **verifying live state before acting on remembered state**.

## It wasn't even the only race that day

The same morning, my human collaborator raced the tooling and won. We'd run a structured clarify session that resolved how to fix a config mismatch between two services. He just… fixed it — a clean atomic rename, deployed — **27 minutes before** the spec tooling finished generating the task list prescribing the work he'd already done.

And while I was writing this, it happened *again*, live. One session — a veteran of the evening's first collision — was quietly draining the open-items list: it pushed pointer lines to two repos at 22:07 and 22:08, updated the retro doc at 22:10. Meanwhile a second session, started at 21:44 with "help me figure out what needs to be done from recent code reviews," was reconstructing that same list — from a snapshot of the world taken *before* the other session started emptying it. One agent computing a to-do list while another deletes its items in real time. I sent a third session in to watch, which makes this post itself part of the traffic: a note that third session left in shared memory at 21:58 (about a security branch I hadn't mentioned to anyone) was picked up and acted on by the first session six minutes later.

The encouraging part: by round two, the sessions had stopped merely surviving the concurrency and started *handling* it. Instead of overwriting the shared memory file, they appended to each other's notes — and began signing them with their own job IDs, citing one another like wary coworkers leaving initials on a whiteboard. Nobody told them to do that.

## What I'm taking from it

Not machinery. We're 2–3 people; I'm not adding distributed locks to my chat windows. Habits:

1. **Memory is intent; reality is truth.** A remembered to-do list is a hypothesis about the world. Every session re-verifies against the live system (GitHub, the deploy, the repo) before executing. This single habit is why tonight was a story and not an incident.
2. **Prefer side effects that are safe to repeat.** Merge-if-open, close-with-comment, idempotent syncs. The second writer should hit a wall, not a landmine.
3. **One live owner per area of work.** Before saying "pick up where we left off," check the job list — something may already be picking up.
4. **Write memory like the next reader is a stranger** — because it is. The note that let two sessions independently reconstruct the exact same plan is the same note that let the second one safely conclude the plan was done.

The punchline writes itself: the agents handled the race more gracefully than I started it. Both of them treated their own notes with suspicion and the world as the source of truth. That's not a bad principle for the humans, either.

## Postscript: how do you close one of these?

After all of the above, I asked a fresh session the dumbest possible question: *"how do i close a session?"* I'd written a whole post about sessions colliding and not once stopped to wonder why they don't just… end.

The answer explains the entire evening. A session in the agents view is a background job by default. It isn't tied to a window — closing the tab doesn't close it. While it's working or waiting on you, it runs. Once it goes idle and sits unattended for about an hour, the supervisor suspends the process to free resources but keeps the transcript and state on disk, so the next time you peek or reply it resumes exactly where it left off. To actually end one you have to say so: `/stop` from inside, `Ctrl+X` in the agents view (again within two seconds to delete), or `claude stop` / `claude rm` from a shell.

Which is to say: nothing here closes on its own. Two-Claudes-one-repo isn't a glitch I tripped over; it's the system working as designed. Disconnect from a session and it doesn't die — it waits. Open a new one to "pick up where we left off" and the old one is still there, idle, holding the same memory note, one keystroke from picking up too. I didn't summon a race condition by being careless. I summoned it by not knowing the off switch.

So, one more habit, and it's just point 3 again with the lights on: before "let's pick up where we left off," check the job list. The thing you mean to resume may already be resuming itself.
