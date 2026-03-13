---
title: "Claude Code Turned a Two-Command Deploy Into a Investigation"
date: 2026-03-13T01:00:00-08:00
draft: false
category: "Claude Code"
excerpt: "I asked Claude Code to add markdown files and push. It ran 30+ commands and diagnosed a broken CI pipeline I didn't know about."
tags: ["Claude Code", "AI", "Building"]
---

# Claude Code Turned a Two-Command Deploy Into an Investigation

The task was simple. I had markdown files. I needed them on my Hugo blog. That's `git add`, `git push`. Two commands. The CI pipeline handles the rest.

Here's what Claude Code actually did:

1. Explored my entire repo structure
2. Read every existing blog post to understand front matter conventions
3. Fetched the README from a separate GitHub repo for a project description
4. Created task lists to track its own progress
5. Ran a humanizer skill on the text
6. Renamed files
7. Built locally with `hugo --minify` to check output
8. Pushed to GitHub
9. Watched it fail
10. Pulled the CI logs
11. Diagnosed that `hugo-version: "latest"` in my GitHub Actions workflow was pulling Hugo v0.157.0, which removed `site.Author` from the template API, breaking PaperMod's RSS template
12. Pinned Hugo to v0.145.0
13. Pushed again
14. Watched it succeed
15. Then discovered one post still wasn't rendering because the date was set to 10 AM Pacific, which is 6 PM UTC, and Hugo skips future-dated posts by default, and CI runs in UTC
16. Fixed the date
17. Pushed a third time

Three commits and 30+ tool calls for what should have been two commands.

The thing is, it wasn't wrong. My CI had been broken since January and I didn't know it. Every push since my last successful deploy would have failed silently. Claude Code found a real bug. It just found it in the most Claude Code way possible: by turning a simple task into a forensic investigation and narrating every step.

The lesson, if there is one, is that AI agents optimize for correctness over efficiency. They will verify everything. They will check the build. They will read the logs. They will not just type `git push` and walk away. Whether that's a feature or a flaw depends on whether your CI is actually broken.

Mine was.
