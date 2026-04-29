---
layout: post
title: "From Shared Hosting to Zero Trust"
date: 2026-04-29 02:00:00 -0700
---

The phone wakes me before my brain does. 99+ on the messaging app icon. I haven't even unlocked the screen but I already know. Pull on whatever clothes I find, open the laptop on the dining table, hit the URL. 502. Refresh. 503. Refresh. 502 again. The error code keeps flipping between the load balancer telling me the backend is gone and the backend telling me it's overwhelmed, and I haven't even finished waking up.

Some hours later, feet feeling cold, problem traced back to a read replica that had drifted far enough behind the primary that the application was stuck retrying for data it had just written, fix deployed, dashboard quiet. I sit there staring at the fridge door and think: I never planned to do this.

---

This is a side project. A community platform I built and have been keeping alive on my own time, separate from my day job, which has nothing to do with backend engineering. There are tens of thousands of people who use it daily. There is no on-call rotation. There is no DevOps team. There is just me, the AWS console, and a long enough memory of past incidents to know that the next one is always one deploy away.

What I want to write here isn't a tutorial. It's the path I actually walked: what I built, what broke it, what I learned, and what I deliberately chose not to do. If you're starting out with AWS and trying to figure out where the next step is, this is the map I wish someone had drawn for me.

## Era 1: Just make it run

In the beginning, the only requirement was that the site exist on the internet at a URL.

I started on shared hosting. The kind where you SSH in, edit PHP files in place, and hope no one notices you're learning in production. It worked, in the sense that the site loaded. The first mistake I made was paying for a three-year discount up front, before I understood that "cheap" on shared hosting buys you exactly the kind of platform you cannot scale on. By the time I needed more, I was locked in for years to a tier with nowhere to go.

The migration to a real cloud provider eventually happened, less because I had outgrown shared hosting and more because I had started to understand what it wasn't doing for me. No way to scale a single component independently. No clean separation between application and database. I moved to a managed application platform with a managed relational database underneath.

A side project run on personal money pays attention to bills. AWS pricing varies meaningfully between regions for the same service, and at one point I migrated my entire stack to a cheaper region because the latency tradeoff was acceptable for my users. I also spent more late nights than I want to admit inside Cost Explorer, hunting down small things: a load balancer I had given up on deleting and then forgotten about, an Elastic IP no longer attached to anything, an RDS instance whose storage I had grown for a one-time spike and then could not shrink back. Production cost intuition is a skill you acquire from cursing last month's bill.

Era 1 was also when I rewrote the backend. The original was, to be honest about it, embarrassing. I am not going to call it "just early code" or "a learning artifact." It was bad. Genuinely bad. No one would have wanted to read it. The infrastructure migration gave me cover to refactor large pieces under the framing of "we're moving anyway, may as well clean it up."

There were also evenings lost to small absurd things. One was several hours convinced something deep in my Nginx setup had broken, before I noticed that a single misplaced empty line in the config was silently preventing PHP requests from being handed off to the interpreter at all. Identifying the empty line took longer than fixing it.

What I didn't do at this stage was anything resembling observability. Logs were files. When something broke, I downloaded them and grepped. I had no metrics, no dashboards, nothing that showed me how the system was doing without me going and asking it directly.

## Era 2: Make it not break

The middle era was driven by the part of the inbox that read "site is slow today."

Traffic grew. Lag became a tax. Pages timed out under bursts. The first instinct of every self-taught operator is to add more boxes, more application servers, bigger database. The smarter move is to figure out what kind of work is actually saturating you.

For me, most of it was reads. Read/write splitting at the database layer was the first real fix. A read replica took the bulk of read traffic off the primary, and the application learned to route reads and writes to different endpoints.

That last sentence sounds easier than it is. Some "reads" wear the shape of writes, in the sense that they cannot tolerate stale data: a balance shown right after a deduction, a moderation status checked right after an update, anything where the user just did something and the next page expects to reflect it. Routing those to a replica that is milliseconds behind reality produces bugs that are very hard to reproduce. I learned this by routing the wrong things. The rule I eventually settled on: if the cost of seeing slightly stale data is unacceptable for that path, the query goes to the primary, regardless of whether the call technically reads.

Replicas didn't fix the fact that the same expensive query was running against the database hundreds of times for data that almost never changed. That is what introduced a cache layer in front of the database. Hot reads now hit Redis. The database stopped being asked the same questions over and over. My first cache configuration shipped with the wrong eviction policy, which silently started rejecting writes once memory filled, and the application happily read stale or missing values for a while before I noticed. Caches reward attention to defaults.

Around the same time, I noticed certain operations had no business being on the request path at all. Sending emails, complex statistics calculations, corrections to historical data. Anything where the user did not need an immediate answer. These moved into a queue, with worker processes pulling jobs off of it on their own schedule. The request returns instantly; the work happens in the background.

Background work has its own discipline. A heavy batch that looks like a free win on the request path can flood writes faster than the replica can keep up, or land on top of the nightly RDS backup window and turn a routine job into a noisy incident. Heavy jobs need to be paced, staggered, and aware of when your backups run. "Async" is not the same as "free."

By the time both of these were in place, the same Redis instance was simultaneously a cache, a session store, and a queue broker, sharing memory across three completely different durability profiles. Cache is fungible: losing it costs latency. Sessions, if you lose them, log everyone out. The queue holds work that has not happened yet: losing it means dropping things users had already paid attention to. So the single Redis became three nodes. Three responsibilities, three durability profiles, three failure modes.

Much later in this era, when database connections themselves became the bottleneck (workers spawning enough at peak that the database ran out of room to think), I added an RDS Proxy in front of the primary. It pools a fixed number of real connections and multiplexes the application's short ones over them. It helped, but less than I had hoped, because AWS does not currently allow RDS Proxy in front of MySQL read replicas. The write side gets pooled. The read side, which is most of the traffic, does not. I have not yet found a workaround I am happy with.

Email was its own small saga. Outbound mail from a community platform turns out to be harder than it looks. Deliverability rules are strict, free providers throttle, and at one point I was rotating through ten different Google accounts as senders. That setup was every bit as fragile as it sounds. I eventually moved to AWS's own email service and started paying for it directly. It cost less than I had been afraid of, ran without my supervision, and stopped being on my mind. Some lessons are "pay the company that does this for a living and move on."

Around this period I also added a CDN in front of the load balancer, a web application firewall on top of that, and started using a log search service instead of grepping local files. The WAF earned its keep more times than I expected. It blocked several DDoS waves before they reached my origin, and it caught a steady stream of injection attempts at the application layer that would otherwise have hit the database.

The search field was its own lesson, separate from the WAF. In the early days, users started getting creative with wildcard characters to do fuzzier searches than the original implementation had anticipated. None of those queries were malicious, but they made the shape of the risk obvious: any place where user input directly shapes a database query is a place an attacker is going to lean on, eventually. I tightened the input handling. I have since spent some genuinely fun hours tracing the patterns left behind by attackers who tried anyway. Those are stories for another time.

I tried alarms in this era and got it wrong. I set too many. Every minor blip woke me up. I couldn't tell which alerts mattered, and the ones that did got buried under the ones that didn't. I eventually turned most of them off, which was also wrong, but in a different direction. I'm still learning the shape of "alert only on what users actually feel," graduated tiers, dashboards for the rest. Solo on-call is a design problem, not a stamina problem, and treating it like stamina is how you burn out.

The debt I carried out of this era was real, and almost all of it was security. The database still had a public IP. One security group covered the database, the application servers, and the cache: they trusted each other's traffic, and they all accepted public traffic on the same rules. Secrets sat in environment files. SSH was a fact of life. The root account was still the one I used for the AWS console. I had a working system, and I had bought myself maybe two more years before the next thing broke.

## Era 3: Make it not bleed

The third migration was different. Nothing was on fire. The site worked. But somewhere in the middle of reading about a string of breaches at companies much larger than mine, I started to feel the weight of what I had not done.

The problem with my security posture wasn't any single thing. It was that the whole shape of it was reactive. Security groups restricted who could talk to the database, but the database was still on the public internet. Anyone in the world could attempt to connect. SSH worked because the bastion host had the right key on the right port, but the port was open, and "the right key" lived in too many places I had lost track of. Credentials sat in env files on disk. The IAM role attached to the application could do more than the application actually needed. And, embarrassingly, my own console access was still the root user of the account.

None of this was _wrong_ in the sense of being broken. It was wrong in the sense that the next person who wanted to get in only had to find one mistake.

The redesign that followed has a few principles worth naming, because they apply to anyone going through the same transition.

**Stop using the root account.** Until this redesign, my primary access to the AWS console was the root user. I knew this was bad. I had read the warnings. I kept doing it because creating a personal IAM user, scoping its permissions, and configuring multi-factor authentication felt like one of those overhead tasks I would get to later. When I finally did it, I did it properly. A personal IAM user with scoped permissions, multi-factor authentication enforced on the profile, and the root credentials locked away. The root account no longer logs in for daily work. It exists for the account-recovery cases that nothing else can do. The friction I had been avoiding turned out to be about ten minutes of one-time setup.

**Network isolation beats network filtering.** A security group is an allow-list. As long as a resource has a public IP, the attack surface exists. You're just betting on the filter. A private subnet means there is no route from the internet to that resource at all. The database, the cache, the application servers all moved into private subnets. The only thing in a public subnet is the load balancer, which speaks HTTPS to the world and forwards application traffic inward. Outbound access for things like pulling container images goes through a NAT gateway, which is one-way by design. The internet cannot initiate a connection back through it.

**Get rid of the bastion host.** The bastion was the next attack surface. It had to be somewhere. It had to listen on a port. It had to hold a key that could reach the database. AWS Systems Manager Session Manager solves this by reversing the connection. A small agent inside the private network reaches outward to the AWS control plane and holds a session open. When I want to connect, I authenticate to AWS, and my session is brokered through that existing tunnel. There is no inbound port. There is no bastion. There are no SSH keys to rotate. Every session is logged.

The first time I used it, my instinct was that it couldn't possibly be that simple. It was. The simplicity is the point. Most of the work the old setup was doing was protecting itself.

**Split IAM by what the role is for.** The application's container needs two different kinds of permission, and conflating them is a common early mistake. The execution role is what lets the container _start_: pull the image from the registry, write logs to the log service. The task role is what the application uses _while running_: read a specific bucket, fetch a specific secret. Splitting these means a compromise of the running application cannot, for example, pull arbitrary images or rewrite log groups. Each role gets only what it specifically needs. The principle is old; the discipline of actually applying it takes a while to internalize.

**Secrets are a service, not a file.** Moving credentials out of env files into a managed secrets service changes the model. Secrets aren't checked out at deploy time and copied around; they're fetched at runtime by an authorized identity. Rotation becomes possible. Auditing becomes possible. The blast radius of a leaked file shrinks dramatically.

**Encrypt by default, on principle.** Volumes encrypted with managed keys, traffic between services using TLS. None of this stops a determined attacker who already has the right access. What it does is constrain the bad stories you'd have to tell after an incident. "The data was at rest, encrypted, with key access logged" is a different conversation than the alternative.

What runs today is a fully containerized application on serverless container compute, fronted by a load balancer in the only public subnet, with everything else (database, cache, supporting services) in private subnets reached only through the Zero Trust session layer. Security groups chain inward: the load balancer accepts traffic from the world, the application accepts traffic only from the load balancer, the database accepts traffic only from the application. Each layer is a hop, and each hop is closed by default.

One operational decision worth mentioning here: the application server and the queue worker run from the same Docker image. A supervisor configuration inside the container decides which role it plays based on how the task is started. One artifact, one place to push updates, two roles that scale independently. Solo operations rewards every reduction in moving parts.

It is, by a wide margin, the calmest the system has ever felt to run.

## What I deliberately haven't done

This is the part I think most architecture write-ups skip, and I think it's the most useful part for anyone reading.

I do not have my infrastructure defined as code. The whole stack lives in the AWS console with some JSON pasted in where it had to be. I know this is the standard recommendation. I have not done it because the architecture doesn't change often enough for the cost of building the Terraform footprint to pay back yet. This is the gap I'm most aware of, and the one I would close first if I had a free week.

I do not have a CI/CD pipeline. Deploys are still build-the-image, push-to-registry, update-the-service, done by hand. It is the next obvious automation, and the friction of doing it manually is probably the only reason I haven't yet been embarrassed into it.

I do not have multi-region failover. If the region I run in goes down, the site goes down. I made this choice consciously: the user community is forgiving of rare regional outages, the cost of true active-active is significant, and I have offline backups for the truly catastrophic case. Most architectures do not need disaster recovery; they need someone willing to think honestly about what kind of disaster they're recovering from.

Each of these is a debt. Each of them is a debt I chose. The thing that took me longest to learn about running production systems is that the goal isn't to have everything. The goal is to know exactly which things you don't have, and why.

## What seven years of this actually teaches you

There is no architecture you arrive at. There is the architecture that grew out of every incident you survived, every alarm you set wrong, every time you said no to a feature because the underlying thing was already too fragile. Best practices are a starting point; the system you actually run is shaped by what your particular system has actually punished you for.

Security, eventually, stops feeling like a checklist and starts feeling like asymmetry. You want every additional step an attacker would need to take to cost them more than the previous one cost you. Closing the database to the public internet is not a 10% improvement on having it open behind a security group. It is a category change. The same is true of removing the bastion, splitting IAM roles, moving secrets out of files. Each one converts an attack from "possible if they find a mistake" to "not possible without a different attack entirely."

Operating a real system on your own, for years, is not glamorous. It is mostly small decisions that no one will ever see, made on weeknights, after a long day of doing something else. But there is a particular kind of confidence that comes from having held the same system together long enough that you remember every reason you have for the choices in it. That confidence isn't transferrable from a course or a certification. You have to earn it by being the person whose phone goes off at 3 a.m.

I did not plan to become that person. I'm glad I am.
