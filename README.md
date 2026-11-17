# Best VPS for n8n: How Many vCPUs You Actually Need, Which RackNerd Plan Fits, and What Year Two Really Costs

Two vCPUs and 4 GB of RAM will run almost any personal n8n instance. If your webhooks come from clients and other people's apps, buy four vCPUs, because that is the point where throughput stops being your problem.

On RackNerd's current annual specials, that translates to the **2 GB KVM plan at $35.99/year** for solo automation work, and the **6 GB plan at $89.99/year** once you need more cores. The 4 GB plan in between is the one most people buy by reflex and probably shouldn't, for a reason that only shows up in load-test numbers. More on that below.

Everything here comes from a five-server benchmark of n8n 2.46.6 with PostgreSQL 16 (Helsinki, 5 November 2026) and from current provider pricing pages. Where I couldn't verify a number, it isn't in the article.

> **Quick pick:** RackNerd's 2 GB KVM plan gives you 2 vCores, 2 GB RAM and 35 GB SSD for **$35.99/year**, roughly $3/month. That meets the 2 vCPU / 2 GB minimum RackNerd lists for its own one-click n8n app, and it leaves room for PostgreSQL and a reverse proxy on the same box.
> 👉 [Check the 2 GB KVM plan at RackNerd](https://bit.ly/RacKnerd)

## Every RackNerd KVM plan, priced for an n8n install

All five run on KVM with full root access, a 1 Gbps port, and the ctrl control panel, where n8n, Coolify, Portainer and Docker are one-click installs. You can pick CentOS, Rocky, AlmaLinux, Fedora, Debian or Ubuntu at order time, and reinstall later from the panel.

| Plan | vCPU / RAM / SSD | Price (annual billing) | Who it fits | Order |
| --- | --- | --- | --- | --- |
| 1 GB KVM | 1 vCore / 1 GB / 20 GB | $21.99/year (≈$1.83/mo) | Below the stated minimum for the n8n one-click app; only for a hand-built, very small instance | [Order the 1 GB plan](https://bit.ly/RacKnerd) |
| **2 GB KVM** | 2 vCores / 2 GB / 35 GB | **$35.99/year (≈$3.00/mo)** | Most self-hosters: one n8n instance, PostgreSQL, daily workflows | [Order the 2 GB plan](https://bit.ly/RacKnerd) |
| 4 GB KVM | 2 vCores / 4 GB / 60 GB | $59.99/year (≈$5.00/mo) | Database and history headroom, but the same two cores | [Order the 4 GB plan](https://bit.ly/RacKnerd) |
| 6 GB KVM | 6 vCores / 6 GB / 100 GB | $89.99/year (≈$7.50/mo) | Client work, bursty webhooks, plans for queue mode | [Order the 6 GB plan](https://bit.ly/RacKnerd) |
| 8 GB KVM | 7 vCores / 8 GB / 150 GB | $119.99/year (≈$10.00/mo) | Several instances or genuinely heavy parallel load | [Order the 8 GB plan](https://bit.ly/RacKnerd) |

Transfer allowances scale with the plan: 3 TB, 5 TB, 7 TB, 12 TB and 20 TB a month respectively, all on a 1 Gbps port. n8n itself will not come close to those limits. Workflow traffic is JSON in, JSON out.

## Does n8n need CPU or RAM? The load test says CPU

A five-server benchmark run on Hetzner Cloud in Helsinki on 5 November 2026 tested n8n 2.46.6 with PostgreSQL 16 in Docker Compose. The workflow was a webhook that ran a Code node and returned a result, fired with ApacheBench at 1, 10, 25 and 50 concurrent requests.

The results line up almost perfectly with core count:

| Server | vCPU / RAM | Executions/s | p95 at 10 concurrent | p95 at 50 concurrent |
| --- | --- | --- | --- | --- |
| CX23 | 2 / 4 GB | 15.04 | 893 ms | 3,954 ms |
| CX33 | 4 / 8 GB | 31.93 | 438 ms | 1,846 ms |
| CX43 | 8 / 16 GB | 36.14 | 341 ms | 1,682 ms |

Doubling from two cores to four took throughput from 15 to almost 32 executions per second. Doubling again, to eight cores, added 13% and cost 88% more. The reason is structural: in normal mode, the n8n main process both receives webhooks and runs executions, so it cannot use eight cores no matter how many you rent. Going past four cores means queue mode, with Redis, workers and a load balancer in front.

Memory told the opposite story. The n8n container's peak RAM never passed **1,049.6 MiB** across every test, including 50 concurrent requests. On a production 2 vCPU box, the whole system idled at 1,036 MiB of 4 GB, with n8n at 379.7 MiB and PostgreSQL at 69.6 MiB. A single request on the small box had a median latency of 104 ms.

So the sizing rule is short: buy cores first, and treat 4 GB as comfortable rather than tight. The older advice of "2 GB minimum, 4 GB preferred" isn't wrong, it's just cautious. The machine that runs out first is the CPU.

Two footnotes worth knowing before you compare prices. Shared vCPUs suffer from noisy neighbours, and the test day happened to record 0.0% CPU steal, so a cheap shared host can behave worse than its spec sheet on a bad afternoon. And if you might ever run queue mode, use PostgreSQL from day one, because n8n's own documentation says a distributed setup on SQLite isn't supported.

## RackNerd's 4 GB plan: more RAM, same two cores

Look again at the RackNerd table. The 2 GB and 4 GB plans both have **2 vCores**. The $24/year difference buys storage and memory, not throughput. Since the benchmark shows n8n peaking under 1.1 GB of container RAM, that extra memory mostly covers PostgreSQL growth, execution history and Docker images rather than n8n itself.

That makes the 4 GB plan a reasonable buy if you keep months of execution data or run a couple of extra containers next to n8n. It is a poor buy if what you actually want is a faster instance, because the next real step up in compute is the 6 GB plan at 6 vCores. That jump costs $30/year more than the 4 GB plan and roughly quadruples the core count. Per core, it's the best-value row in the table.

If your workflows are personal and run a few hundred times a day, none of this matters much. 👉 [The $35.99/year 2 GB plan](https://bit.ly/RacKnerd) will sit at 5% CPU and you'll forget it exists.

## Year two is where cheap VPS pricing gets expensive

This is the part of the comparison that most "best VPS" lists skip, and it's the part that decides what you actually pay.

| Provider | Plan | First-term price | Renewal price | Three-year cost |
| --- | --- | --- | --- | --- |
| RackNerd | 2 GB KVM | $35.99/year | Annual rate shown at checkout | $107.97 at the listed rate |
| Hostinger | KVM 2 (2 vCPU / 8 GB) | $8.99/mo on a 24-month term | $14.99/mo | ≈$395.64 |
| DreamHost | Stack 8 (8 GB) | $9.49/mo on a 2-year term | $14.99/mo | ≈$407.64 |
| Bluehost | NVMe 8 (8 GB) | $12.99/mo on a 24-month term | $28.99/mo | ≈$659.64 |
| Hosting.com | Self-managed 8 GB, 4 cores | $9.99/mo on a 36-month term | $29.00/mo | ≈$707.64 |

Those renewal figures come from the providers' own comparison pages, and the pattern is consistent: the advertised price covers the first term only, then roughly doubles. Hostinger's $8.99 becomes $14.99. Hosting.com's $9.99 becomes $29.

RackNerd sells annual, so the number you see is one year of service, not a teaser month. The three-year figure above assumes the listed rate holds on renewal, which is the one thing you should confirm in the cart before paying, because a renewal rate is the number that decides whether year two is still a good deal. Same for the refund window: read the terms on the order page rather than trusting a blog post, including this one.

## The "KVM2 crashes above 10 nodes" thread, explained

A widely shared r/n8n post describes a Hostinger KVM2 (2 vCPU, 8 GB) that "crashes" past ten nodes, looping from the starting node until manually stopped, even in test mode with pinned data.

The replies are more useful than the original complaint. One user pointed out that a KVM2 shouldn't fall over at ten nodes, and mentioned running a **70-node workflow on a 1 GB Google Cloud free instance** without issues. Another, writing in Spanish, said the problem is how you organise nodes: heavy parallel branches choke the server, and running them sequentially fixes it. A third runs heavy motivational voice-over workflows on AWS at **$150/month** and reports no crashes at all.

Both things are true at once. Node count says nothing about load: ten nodes in a straight line that each call an API can be lighter than three nodes looping over 5,000 rows. The benchmark numbers above measure concurrent webhook throughput, not workflow complexity, so they don't contradict a 1 GB machine handling a long workflow that runs once an hour.

Before you upgrade anything, spend twenty minutes on the workflow:

1. Set the workflow to execute sequentially instead of in parallel branches.
2. Turn off saving successful execution data, and pin test data instead of re-running live calls.
3. Look for a loop or a Wait node feeding back into an earlier node, which produces exactly the "circling back to the start" symptom.
4. Check whether pinned data is being discarded on each run, forcing every node to re-execute.

If the instance still stalls under concurrent webhook traffic after that, the bottleneck is cores, and the fix is a plan with more of them.

## How RackNerd compares to the usual shortlist

Prices below are the verified current figures, in the currency each provider lists. RackNerd line items are annual; the rest are monthly unless noted.

| Provider / plan | vCPU / RAM | Price | Term and renewal |
| --- | --- | --- | --- |
| RackNerd 2 GB KVM | 2 / 2 GB | $35.99/year | Annual |
| RackNerd 6 GB KVM | 6 / 6 GB | $89.99/year | Annual |
| Hetzner CX23 | 2 / 4 GB | €6.64/mo | Monthly, no term |
| Hetzner CX33 | 4 / 8 GB | €10.27/mo | Monthly, no term |
| OVHcloud VPS-1 (n8n preinstalled) | 2 vCores / 4 GB / 40 GB NVMe | from $4.54/mo | Monthly |
| OVHcloud VPS-2 (n8n preinstalled) | 4 vCores / 8 GB / 75 GB NVMe | from $8.50/mo | Monthly |
| Hostinger KVM 2 | 2 / 8 GB / 100 GB NVMe | $8.99/mo | 24-month term, renews $14.99 |
| DreamHost Stack 8 | 8 GB | $9.49/mo | 2-year term, renews $14.99 |
| DigitalOcean Droplet | 2 / 4 GiB | $24/mo | Monthly |
| n8n Cloud Starter | 2,500 executions/mo, 5 concurrent | €20/mo | Billed yearly, managed |

A few things that table doesn't say out loud.

Hetzner CX33 is the strongest pure-performance buy on the list: four cores at €10.27 monthly with no commitment, and it posted the 31.93 executions/s figure above. It's also the plan to pick if you want an EU location next to European APIs and hate prepaying. OVHcloud's n8n VPS line is the only one here with n8n preinstalled, at "from" prices that shift with configuration, and OVHcloud states plainly that support covers the infrastructure, not the n8n application. DigitalOcean's $24 buys regions and documentation rather than n8n speed; the same workload runs on a two-core box for a third of that. n8n Cloud Starter at €20/month is the option for people who don't want a server at all, though 2,500 executions is a ceiling worth calculating against your own history before committing.

RackNerd's position in that field is price per year, not per month. $35.99 annually for a two-core KVM box with root access, Docker and a one-click n8n install undercuts every monthly-billed option here once you stop dividing by twelve and start comparing what leaves your account over three years.

## Pick by what you're actually running

- **Personal workflows, a handful of runs per hour, one instance:** RackNerd 2 GB at $35.99/year. Nothing else here beats it on total cost.
- **Webhooks from clients, bursts of concurrent executions, or a plan to add workers later:** RackNerd 6 GB at $89.99/year, with PostgreSQL from the start.
- **You want monthly billing and an EU datacentre:** Hetzner CX33 at €10.27.
- **You want n8n preinstalled and don't mind reading docs:** OVHcloud VPS-2 from $8.50.
- **You never want to touch a terminal:** n8n Cloud Starter at €20/month.
- **You're on a 1 GB or 2 vCPU box already and it's slow:** fix the workflow first, as in the checklist above. Then add cores.

## Before you click order

Five things worth confirming, all of which have bitten someone:

- **Location.** RackNerd deploys in multiple datacentres. Pick the one closest to the APIs your workflows hammer, since every HTTP Request node pays that latency. If you're travelling, the server's location matters more than yours.
- **Root and Docker.** RackNerd specials are unmanaged KVM with full root access; Portainer installs in one click from ctrl, or bring your own Docker over SSH.
- **Disk.** A fresh n8n plus PostgreSQL install used 4.7 GB of a 38 GB disk in testing. 35 GB is fine to start, but execution history grows quietly, so prune it on a schedule.
- **Database choice.** PostgreSQL if queue mode is even a maybe. SQLite can't do distributed setups, and migrating later is more work than choosing correctly now.
- **Renewal and refund terms.** Both are on the order page. Read them before entering card details, on any provider.

My rule, in one line: buy cores, take the smallest plan that clears your provider's own stated minimum, and check the renewal rate before you check the discount.

For most people reading this, that's 👉 [the 2 GB KVM plan at $35.99 a year](https://bit.ly/RacKnerd), with n8n installed from the panel in a few minutes and $3 a month leaving your account instead of $15.
