---
layout: post
title: SSH Is Not a Desktop
postDate: 2026-09-18T09:00:00-05:00
categories: []
tags: [ai, agents, claude-code]
published: true
permalink:
image: /assets/2026-09-18-SSH-Is-Not-A-Desktop/featured-image.png
---

![SSH Is Not a Desktop](/assets/2026-09-18-SSH-Is-Not-A-Desktop/featured-image.png)

I do most of my work across three physical PCs. Two of them (I'll call them `devbox1` and `devbox2`) are beefy machines that sit in my office running all the time. The third is my laptop, which goes where I go.

Most of my development work these days is done through Claude Code. When I'm away from my office, I don't really want Claude running on the laptop. I want it running on the always-on machines, with the laptop acting as a window into Claude Code running over there.

## Why remote in at all?

There are two reasons I'd rather not run Claude Code directly on my laptop.

**The laptop doesn't have everything.** My desktop machines have the full toolchain installed and configured: Docker, kubectl and access to my Kubernetes cluster, multiple .NET SDKs, database tools, and all my repos checked out at the same paths. Claude Code is only as capable as the tools it can reach. On the laptop, some of those tools are missing and others aren't configured, so the agent ends up blocked or improvising. I could try to keep the laptop in sync with the desktops, but that's a lot of ongoing work for a machine that's mostly meant to be light and portable.

**Claude Code uses a lot of bandwidth.** People don't expect this one. Every turn of a Claude Code conversation sends the context (the conversation so far, the file contents it has read, tool output, and so on) up to the model, and streams the response back. In a long session working on a real codebase, that adds up to a lot of data, and much of it is _upload_, which is usually the weakest part of any connection. On airplane Wi-Fi or a congested hotel network, running Claude locally can go from sluggish to unusable.

An SSH session, on the other hand, only sends keystrokes and terminal text. When Claude runs on my desktop, all the heavy traffic goes over my office's fast, stable connection, and the only thing crossing the hotel Wi-Fi is what's on my screen. The same bad network that makes local Claude Code unusable works fine for a remote terminal.

So the plan was simple. Windows has shipped an OpenSSH server for years, and from my laptop it's one command to get a shell on either box: `ssh devbox1`, `cd` into a repo, run `claude`, and get to work.

It mostly works. But "mostly" hides four problems that have cost me a surprising amount of time. The short version is that an SSH session is _not_ a desktop session, and both Claude Code and the tools it relies on (especially `git`, `gh`, and Docker) quietly assume they're running in one.

## Problem 1: Closing the client kills the work

The first thing that bit me: if I close the terminal on my laptop, or the laptop goes to sleep, or the Wi-Fi hiccups, whatever I was running on the remote machine dies with it.

That's actually reasonable behavior from the SSH server's point of view. On Windows, the OpenSSH server runs everything spawned by a session inside a job object, and when the session ends the job is torn down, along with every process in it. Backgrounding things doesn't help, and neither does `Start-Process`. If a process was born inside that SSH session, it dies with it.

For a quick `git status` that's fine. For a Claude Code session that's twenty minutes into a refactor, it's painful. The agent is killed mid-thought, and any in-flight work is left however it happened to be when the connection dropped.

On Linux the standard answer is `tmux` or `screen`: run your work inside a terminal multiplexer that lives independently of the SSH connection, and reattach later. Windows doesn't have a native equivalent. You can get there through WSL, but a lot of my work is Windows-native (.NET, PowerShell, Windows paths), so that's only a partial answer.

Claude Code's session history helps. After reconnecting, I can run `claude --resume` and pick up the conversation, but that only restores the conversation. Whatever Claude was doing when the connection dropped (a build, a test run, a half-applied set of edits) was interrupted, and I have to work out where things stand before carrying on. On a flaky connection, where the whole point is that the network is unreliable, that happens often.

This turned out to be the problem that pushed me beyond plain SSH. The real fix is to have Claude Code running somewhere that doesn't depend on my connection at all, and just _attach_ to it from wherever I am. That's the subject of the next couple of posts. For this post, the lesson is that SSH works well for short, disposable commands, but it's a poor place to start anything long-lived.

## Problem 2: Git and gh need different authentication

The second problem showed up the first time an agent tried to push a branch.

On a normal Windows dev box, Git is configured to use Git Credential Manager (GCM). GCM is great: it pops up a browser or a dialog, you sign in to GitHub, and it caches a token so you never think about it again.

The phrase "pops up" is the problem. In an SSH session there is nothing to pop up _on_. GCM needs either a GUI or an interactive terminal it can prompt through, and when it has neither it fails with the uninformative:

```
unable to read askpass response
could not read Username for 'https://github.com'
```

This applies to Claude Code's own shell tool too, because the commands an agent runs are non-interactive by nature. So the credential setup that works perfectly when I'm sitting at the machine fails as soon as the work is driven remotely or by an agent.

The fix was to stop using HTTPS credentials for Git on these machines entirely and switch to SSH keys for GitHub:

- A dedicated GitHub key, `~/.ssh/id_ed25519_github`, with **no passphrase**. That's deliberate. A passphrase means `ssh-agent`, and an agent started in my desktop session isn't reachable from an SSH login, so you end up with exactly the same kind of failure, just in a different spot.
- An entry in `~/.ssh/config` that scopes that key to `github.com` with `IdentitiesOnly yes`. Without that, ssh helpfully offers my _other_ key first (the one I use to remote between machines). GitHub rejects it, and after enough rejected keys you get "Too many authentication failures" and nothing works.
- A global URL rewrite so every repo uses SSH, regardless of how it was cloned:

```bash
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

One gotcha with that last one: `git remote -v` still _shows_ the https URL, because the rewrite happens when git actually talks to the remote. If you want to see what's really going to be used, `git ls-remote --get-url <url>` tells the truth.

The `gh` CLI is a separate story, because it doesn't use Git's credentials at all. It stores its own token, and by default it puts that in the OS credential store. That works well at the desktop, but it's another piece of state that belongs to the interactive session, and I've had `gh` report that its token is invalid in situations where it had been working fine moments before at the console. If that's a problem for you, `gh auth login --insecure-storage` writes the token to a plain file (`%APPDATA%\GitHub CLI\hosts.yml`) that any shell can read. As the flag name suggests, that's a tradeoff, and you should think about it before taking it.

It's also worth knowing that the GitHub MCP server authenticates on its own path, independent of both Git and `gh`. During one stretch where `gh` was broken over SSH, the agent could still read issues and open PRs through MCP, which kept me productive while I sorted out the CLI.

## Problem 3: Reboots leave things half-started

These machines are "always on" until they aren't. Patch Tuesday comes around, Windows Update decides it's time to reboot overnight, and the machine comes back up to the login screen with nobody logged in.

The OpenSSH server is fine with that, because it runs as a Windows service, so I can still connect and start Claude. But a lot of what Claude needs to do real work isn't built that way. Docker Desktop is the big one. Despite the Windows services it installs, Docker Desktop is really a per-user application that starts when you log in to the desktop. No interactive logon, no Docker. The same is true of anything else that starts from a Startup folder or a "run at login" setting, such as tray apps, VPN clients, and sync tools.

From an SSH session, the result is confusing. I connect, start Claude, and everything looks normal until the agent tries to build a container or run the tests that depend on one. Then it fails with an error saying it can't connect to the Docker engine. Claude will sometimes cheerfully try to "fix" that by digging through Docker's configuration, when the real problem is just that nobody has logged in since the reboot.

And you can't really fix it from SSH. Starting Docker Desktop from an SSH session either doesn't work or starts it inside the SSH session's job, where it dies as soon as I disconnect (see Problem 1). The practical fix is to open a GUI remote session, log in so the normal startup apps run, wait for Docker to report that it's ready, and then go back to SSH and Claude.

I've mostly learned to check for this first after any patch Tuesday. Setting a machine to log in automatically would solve it, but that means leaving a logged-in desktop sitting there, which is a security tradeoff I'm not keen on.

## Problem 4: Sometimes you just need the GUI

This is the one I find the most frustrating, because it undercuts the whole point.

A lot of authentication and troubleshooting on Windows simply assumes someone is sitting at the console:

- Signing in to GitHub (or Azure, or Claude) for the first time usually means a browser-based OAuth flow.
- Windows Hello, UAC prompts, and "allow this app?" dialogs appear on the physical desktop, not in my SSH terminal.
- Credential stores are often tied to the interactive logon session, so a token you set up at the desktop may not be usable from an SSH session, and vice versa.
- After a reboot, services like Docker Desktop don't start until someone logs in to the desktop.
- When something fails over SSH, the most useful diagnostic step is often "does it work when I'm actually logged in?", which you can't answer from SSH.

So in practice, making the SSH workflow work, and fixing it when it breaks, often means opening a GUI remote session to the _same_ machine using AnyDesk or RustDesk. I sign in to the desktop, complete the browser login or approve the prompt, confirm that the tool works interactively, and then go back to SSH to see whether it works there too.

It works, but it's clunky. I'm using a GUI remote desktop tool to fix problems in my headless remote shell. And it only works because both machines stay logged in to a desktop session (or I'm willing to log in remotely), which is precisely the dependency I was trying to get rid of.

The long-term answer is the same pattern as the Git fix: find the credential path for each tool that doesn't depend on an interactive session, and set it up once while you're at the GUI. Plain SSH keys instead of GCM. File-based tokens where the risk is acceptable. Services that start at boot rather than at login. Every tool I move to that kind of path is one less reason to reach for AnyDesk.

## A productivity tip: review through a branch

One more thing, and this applies however you reach the remote Claude, whether over SSH or any of the options I'll cover in future posts.

When Claude is running on another machine, the changes it makes live on that machine too. Reviewing them from a terminal (scrolling through `git diff` over a slow SSH connection) works, but it's not a great experience, especially for anything larger than a few lines.

So I have Claude do its work in a git branch and push that branch to GitHub, usually with a pull request. Then I review the changes on my laptop the way I'd review anyone else's: in the browser, with GitHub's diff view, where I can comment on specific lines. If I want to run or debug the code locally, I can pull the branch down onto the laptop, since it's just a branch.

This post is an example. Claude drafted it on `devbox1` while I was talking to it from my laptop. When I wanted to actually read the draft, I asked Claude to create a PR, and I read it from the laptop on GitHub. Any comments I had went back to Claude to address, and the next push updated the same PR.

It also takes care of a quieter risk. Work that lives only on the remote machine isn't backed up anywhere until it's pushed. If the session dies or the machine gets rebooted, a pushed branch means nothing is lost.

## Lessons

If you're thinking about SSHing into a Windows workstation so you can run Claude Code there, here's what I'd pass along:

1. **Expect disconnects to kill your session.** On Windows, anything started in an SSH session dies when the session does, and a Claude Code session is no exception. `claude --resume` brings back the conversation, but not the interrupted work.
2. **Assume nothing can prompt.** Any credential flow that pops up a dialog, opens a browser, or asks for a passphrase is going to fail over SSH and inside agent shells. Pick non-interactive credentials on purpose.
3. **Know what needs a logon.** After a reboot, anything that starts at desktop login (Docker Desktop above all) won't be running. When Claude hits a strange failure after patch Tuesday, check that first.
4. **Keep a GUI path available.** You'll still need a real desktop session now and then, for first-time sign-ins, for getting startup apps running after a reboot, and for "does this even work locally?" troubleshooting. Have AnyDesk, RustDesk, or RDP set up before you need it, not after.
5. **Review through a branch.** Have Claude push its work to a branch (ideally with a PR) so you can review it comfortably from wherever you are, and so the work is safe off the remote machine.

All of this is worth the effort. Running Claude Code on a well-equipped desktop and reaching it over a thin SSH connection means I get my full toolchain from anywhere, and a bad airplane connection is only a nuisance instead of a showstopper.

None of these problems is hard once you understand them. What makes them hard is that the failure messages almost never say "this is failing because there's no desktop session." Once I started treating "is this running in an interactive session or not?" as the first question to ask, most of the mystery went away.

SSH was my first step, not my last. Problems 2 through 4 apply no matter how you reach the machine. Problem 1, though, is specific to SSH, and it's the one that bothered me the most. In the next post I'll look at Claude Code's Remote Control feature, which keeps the session running on the desktop and lets me attach from my laptop, a browser, or my phone. After that, I'll cover going a step further and running a Remote Control _server_ that's always there, even when nobody is logged in.
