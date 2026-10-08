# CM demo 2 multi-region (cmdemo2): presenter's walkthrough

A 20 to 30 minute tour of a live system, for a class on DevOps and GitOps. Every label in quotes is the page's own wording, read on 2026-10-07 and 2026-10-08 and rehearsed against the live pages on 2026-10-08 (dates in UTC). Numbers are from those days and will differ on the day: read them off the page, do not recite them from here.

Time budget: 1 running (3 min), 2 Runtime view (5), 3 where it comes from (4), 4 how a change travels (4), 5 failover (6), 6 operations (3), 7 fleet (2).

## Addresses

| What | Address | Sign-in |
|---|---|---|
| App, tdd | https://cmdemo2-tdd-ui-fqbpbvbgdcgsb8bv.z02.azurefd.net | no |
| App, uat | https://cmdemo2-uat-ui-gjfyh5dcf2e2aagg.z02.azurefd.net | no |
| App, prod | https://cmdemo2-prod-ui-a2b5hkfrchg3ckew.z02.azurefd.net | no |
| Dashboard, tdd | https://calm-wave-0b8361f10.6.azurestaticapps.net/ | no |
| Dashboard, uat | https://gentle-wave-0b8a73410.2.azurestaticapps.net/ | no |
| Dashboard, prod | https://agreeable-meadow-01d3c7010.1.azurestaticapps.net/ | no |
| System repository | https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system | no |
| App repository | https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-workorders | no |
| Dashboard repository | https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-dashboard | no |
| Octopus Deploy, space "cmdemo2 demo" | https://clearmeasure.octopus.app/app#/Spaces-356/projects | yes |
| Azure portal (every underlined name or number on a dashboard) | opens from the dashboard | yes |
| Fleet page | https://stcmfleetprodw35dyr.z19.web.core.windows.net | no |

Each dashboard shows all three environments. Add `#runtime/tdd`, `#runtime/uat` or `#runtime/prod` to a dashboard's address to open the Runtime view of one environment.

## Before the class

1. Open these tabs in this order: app prod, dashboard tdd, dashboard uat with `#runtime/uat`, cmdemo2-system on GitHub, Octopus projects, fleet page.
2. Sign in to Octopus and to the Azure portal now. Both ask for a sign-in, and a sign-in in front of a class costs a minute.
3. On a dashboard, read the header. It must say "All 8 nodes healthy" (three Front Door endpoints and five web apps). The browser tab's title says the same.
4. On each environment, read the banner: "Expected to serve traffic: westus3 (primary)", "Front Door agrees: it is healthy.", "Pinned 2.4.18", "In sync: all 2 nodes run 2.4.18." (the version will differ; tdd has one node and says "In sync: westus3 runs 2.4.18.").
5. Use a browser window at least 1400 px wide; at 1300 px the diagram is cut off on the right. On the Runtime view "Fit to width" is already on (the button is black, like the chosen environment): do not press it. A press shows the diagram at its actual size, 1880 px wide, which scrolls sideways; a second press brings it back.
6. Leave "Probe" on "Health check". "Liveness" hides the health check marks and the database's state.
7. Open the fleet page and read cmdemo2's card, so that its state is no surprise (a card that says "BEHIND THE STANDARD" or "NEEDS ATTENTION" names the reason under it).
8. Decide who starts the failover test in step 5: you, signed in to Octopus, or the system's owner on your word. Check that the person can run runbooks in the space (not verified here).
9. Run nothing destructive. No runbook except "Failover test" in uat, and that one only in step 5 and only after saying so. Nothing that removes Front Door or a public address: the system keeps them.
10. Leaving a browser on a dashboard is fine. The page stops checking while its tab is hidden.

The dashboard is a Blazor WebAssembly page: its first load takes a few seconds, and the header says "Checking 8 nodes" and "Last refresh: not yet" until the first answers arrive.

## 1. What is running (3 min)

1. Open the app in prod: https://cmdemo2-prod-ui-a2b5hkfrchg3ckew.z02.azurefd.net
   - Point at: the browser tab's title "Clear Measure - .NET Bootcamp - Work Order Application", the menu ("Home", "Health Check", "Work Order Management", "Church Bulletin", "Login"), and the last line of the page, which shows the version and the commit, for example `2.4.18+7053d58…`. The line ends with "· unknown · unknown"; the page does not say what those two fields are.
   - Say: "This is a .NET work-order app. This address is Azure Front Door, not a server. Remember the version number at the bottom; we will meet it four more times."
   - No sign-in is needed for the start page. Signing in to the app was not tried here.
2. Open the dashboard: https://calm-wave-0b8361f10.6.azurestaticapps.net/ (the "Health" tab is the default).
   - Point at the header: "All 8 nodes healthy", "Live", "Last refresh", and "Cost of the system".
   - Say: "This page has no server behind it. Your browser asks every node directly, every 30 seconds. What you see is what a client on the internet sees."
3. Scroll through the three sections "tdd", "uat", "prod".
   - Point at: tdd has two tiles ("Front Door" and "westus3"); uat and prod have three ("Front Door", "westus3" with "primary" and "serves traffic", "eastus2" with "standby").
   - Point at one tile: "HTTP status", "Latency", "Version", "Last check", then "LAST MINUTE", "PROCESS" and "HEALTH CHECKS". The green strip at the bottom of a tile is its last 30 checks.
   - Say: "Three environments, one page. tdd has one region. uat and prod have a primary in westus3 and a hot standby in eastus2 behind Front Door."
4. Optional, 20 seconds: open https://cmdemo2-prod-ui-a2b5hkfrchg3ckew.z02.azurefd.net/_version and `/_healthcheck`. The first answers a small JSON with the version, the second the word "Healthy". Say: "The dashboard reads exactly these."

If slow or red: a first answer after idle took 0.3 to 1.2 seconds here. If a tile is not "Healthy", press "Check now" once. If it stays red, read "HEALTH CHECKS" on that tile: an unhealthy tile names the entry that failed. Then teach from it; do not hide it.

## 2. The Runtime view (5 min)

1. On the tdd dashboard, click the tab "Runtime", then the button "prod". Address: https://calm-wave-0b8361f10.6.azurestaticapps.net/#runtime/prod . Leave "Fit to width" as it is: it is on.
   - Say: "Same checks, drawn as a C4 deployment diagram. The boxes were drawn when the dashboard was deployed. The colours, words and numbers are live."
2. Point at the frames, left to right.
   - "rg-cmdemo2-edge" with "afd-cmdemo2 [Front Door profile, Standard: global]" and the endpoint "cmdemo2-prod-ui".
   - "westus3 [Azure region: primary]" with the mark "serving traffic", and under it "eastus2 [Azure region: standby]" with "standby: ready". Each holds an App Service plan, size B1, and a web app.
   - "centralus [Azure region: data, static sites]": the database "sqldb-cmdemo2-prod" and the dashboard's Static Web App.
   - Say: "Two regions run the app. One database serves both, and it lives in a third region. Keep that in mind for the failover."
3. Point at the Front Door endpoint's tile: "routes to westus3 (priority 1)" and "agrees with the web apps".
   - Then the two arrows out of it: "origin, priority 1" with "first, while healthy" (solid green: it carries the traffic) and "origin, priority 2" with "when priority 1 is down" (dotted grey: idle). Each line ends with Front Door's health probes of the last minute, for example "· 68 probes".
   - Say: "Front Door probes both origins and sends traffic to the healthy one with the lowest priority. A failover is the green line moving from priority 1 to priority 2."
4. Point at the numbers on the arrows: "0 calls/min" from the browser, "0 calls/min" to each origin, and about 20 to 30 "calls/min" on "reads and writes [TCP 1433]" with "app queries · 54 background".
   - Say: "Each web app counts its own last minute and publishes it at `/_telemetry`. Nobody is using the app, so requests are zero. The database still gets queries: the health checks, and a message bus that polls in the background."
5. Switch to "tdd" (button "tdd"; address `#runtime/tdd`). Scroll to the bottom: "Traffic". Check that "Environment" says "tdd". Press "Generate traffic" once.
   - What it does, in the page's words: "2 requests a second for 60 s from this browser to ui at cmdemo2-tdd-ui-…azurefd.net: the start page and the app's representative reads".
   - The button becomes "Stop traffic". Under it a counter runs: "18 sent, 18 answered; 51 s left."
   - "What just happened" gets a line: "Traffic started: 2 requests a second for 60 s to ui at …".
6. Scroll back up to the diagram and watch for one minute. Observed here on 2026-10-07:
   - After 20 s: "21 calls/min" on browser to Front Door and on "origin, priority 1"; the web app's tile says "21 req/min · p95 65 ms"; the database arrow rose from 17 to 33.
   - After 40 s: 60. After 60 s: 117 calls/min, 92 on the database arrow, CPU about 2 %.
   - Then "Traffic ended: 118 sent, 118 answered". The numbers fall again over the next minute, because they count the last minute.
   - Say: "Follow one request: browser, Front Door, the origin that serves, the database. The small line next to each number is its trend since this page was opened."
   - During the traffic the page checks every 10 s, although "Interval" still shows "30 s".
7. Point at the web app's tile for the process vitals: "CPU 1.1 % · 154 MB · 3 in flight", "0 errors · 0 exceptions/min", "up 30 h".
   - Say: "Uptime is the cheapest restart detector there is. Under five minutes the page writes 'restarted' and marks it as a warning."
8. Point at the row of eight small marks and "8 checks healthy". Hover one mark for its name and words. The names are on the Health tab: "API", "DataAccess", "Jeffrey", "LlmGateway", "NeedsReboot", "ProcessThreadCount", "Server", "self".
   - Say: "A health check that answers only 'Healthy' tells you nothing when it fails. This one answers entry by entry, so a red tile says which dependency is the reason."
   - If asked what "LlmGateway" is: the app's chat feature. It is not configured in this system (the check's words are "Chat client is not configured"), so the diagram draws no box for it.
9. Point at any underlined name or number. Say: "Each one is a deep link: the web app in the Azure portal, Live Metrics, the release in Octopus. They ask you to sign in." Open one only if you signed in beforehand.

If slow or red: if the numbers do not move, wait for the next check (10 s) or press "Check now". If "Environment" in the Traffic panel shows another environment, set it back to tdd. On the Runtime view the dropdown follows the buttons "tdd", "uat", "prod" (on `#runtime/prod` it says "prod"); on the Health tab it starts on "tdd" on every dashboard.

## 3. Where it comes from (4 min)

1. Stay on the Runtime view and scroll under the diagram to the card "Code" ("ui in tdd"). It is also on the Health tab inside each environment.
   - Point at the first line: "build 2.4.18", "commit 7053d58", "built 41 h ago", "build run".
   - Point at the rest: "82,655 lines in 1,114 files" with the language bar, "Tests 1,560: 1,070 unit, 339 integration, 151 acceptance", "Coverage 90 % of lines, 81.4 % of branches", "Complexity average 2, worst 42 (1,379 methods)", "CRAP worst 6, none over 6", "Qodana 1 problem" (with a warning mark).
   - Say: "The running app answers these numbers itself, at `/_build`. They are the facts of the build that produced this exact binary, not of the latest commit."
   - Below the "Delivery" cards is a second card "Code", "dashboard (this page)": "build 1.0.28", "19,652 lines in 133 files", "Tests 783 unit", "Coverage 75.6 % of lines, 65.9 % of branches". Say: "The dashboard holds itself to the same rule: it publishes the facts of its own build."
2. Click "commit 7053d58". It opens the commit in cmdemo2-workorders on GitHub (no sign-in).
   - It came from pull request 8: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-workorders/pull/8
3. Go back and click "build run". It opens the run "2.4.18 • Build • master" of the workflow "Build".
   - Say: "From a box on a diagram to the commit and the build in two clicks. The build number is the version: 2.4 plus the run number."
   - The run page is public. The log of a job asks for a GitHub sign-in.
4. Open the three repositories and say one sentence each. Use the addresses in the table: the organization's page, https://github.com/clearmeasure-aisf-sample-apps, lists 273 repositories.
   - cmdemo2-system: "The desired state of the whole system: which environments exist, what Azure resources each has, and which version runs where."
   - cmdemo2-workorders: "The app's code. Its build publishes a versioned package."
   - cmdemo2-dashboard: "The page we have been looking at."
5. In cmdemo2-system, open the README's table "What lives where": https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system#what-lives-where
   - Point at the rows `system.json`, `environments/<env>/versions.json`, `infra/`, `octopus/`, `scripts/`, and at the column "Written by".
   - If you open `system.json`, scroll to `deployables` and `environments`. The file also holds the Azure identifiers of the system; skip that block.
6. Open the pin of prod: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/blob/main/environments/prod/versions.json
   - It holds two lines: `"dashboard": "1.0.28"` and `"ui": "2.4.18"`.
   - Say: "This file is the GitOps pin. Git says prod runs ui 2.4.18. The dashboard reads this file and compares it with what the nodes answer: 'Pinned 2.4.18', 'In sync: all 2 nodes run 2.4.18.'"
7. On the Health tab, click "Pin history" in prod's banner. It opens the commits of that file.
   - Point at messages such as "Pin dashboard 1.0.28 in prod (Deployments-…)", all by the machine user.
   - Say: "People never edit this file. Octopus commits the pin as the first step of a deployment, and reverts it if a later step fails. The history of this file is the history of prod."
8. Optional: open the picture of the whole chain, "What depends on what": https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/blob/main/docs/architecture/20-dependencies.png . The README of that folder does not list this picture; open the file itself.

If slow or red: if a card "Code" is missing, the node did not answer `/_build`; use another environment's card, they show the same build while the versions are in sync.

## 4. How a change travels (4 min)

Tell it as one story, with pull request 8 of the app as the example, and show the evidence in this order.

1. The pull request: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-workorders/pull/8 . It is merged: point at the tab "Checks 14" and at the line "14 checks passed" under "merged commit 7053d58 into master".
   - Say: "Nobody pushes to master. The branch rule asks for a pull request and for two checks: 'Build result' and 'secret-scan'."
2. The build and the release: the "build run" link from step 3. After a green "Build" of master, the workflow "Release" pushes the package to Octopus and creates release 2.4.18 of the project cmdemo2-ui.
   - Say: "GitHub builds and tests. Octopus deploys. The hand-over is one versioned package."
3. Octopus (sign-in): https://clearmeasure.octopus.app/app#/Spaces-356/projects/cmdemo2-ui . Point at the release's row: tdd, uat, prod.
   - Say: "The lifecycle is tdd, then uat, then prod. tdd is automatic. uat and prod wait for a promotion, and their first step is 'Sign-off'."
   - Open the release (the version on a "Delivery" card links to it) and one deployment's task log. The steps, by the repository's Terraform: "Sign-off", "Record restore point" (prod only), "Pin version", "Migrate database", "Acceptance tests" (tdd only), "Update deployable", "Verify deployable", and "Revert pin" when a step fails.
   - The exact Octopus screens were not opened here: they need a sign-in.
4. Back on the dashboard, Health tab, the three cards "Delivery" "ui in tdd", "ui in uat", "ui in prod".
   - tdd: "Deployed 2.4.18", "Lead time 24 min from commit 7053d58 to this deployment". There is no "Signed off" line: tdd has no sign-off step.
   - uat: "Signed off by ai-ops", the reason "Build facts carry the Qodana count; proven in tdd", "Lead time 36 min", "Compared same as tdd".
   - prod: the same sign-off and reason, "Lead time 48 min".
   - Say: "Who deployed what, when, with whose sign-off and for what reason, and how long a commit took to reach each environment: 24, 36 and 48 minutes. The page reads this from a file the pipeline publishes, `delivery.json` on the branch `status` of the system repository."
   - Say, if asked who "ai-ops" is: "The operator's automation account. Every sign-off on these cards is automation's, and only with a recorded reason. People who may sign off are named in `system.json` under `octopus.approvers`; it names one today."
5. Point at the other two "Delivery" cards per environment: "the system (infrastructure and pipeline)" and "dashboard".
   - Say: "Infrastructure travels the same road. A merge to cmdemo2-system makes a release of the project cmdemo2-system; Octopus applies `infra/` to tdd, then to uat and prod after a sign-off."
6. Point at "Last 7 days" on the cards of ui: for example "12 deployments, 3 failed" in tdd and "6 deployments, 1 failed" in uat, each with a warning mark, and "5 deployments, none failed" in prod.
   - Say: "Failures show, and most of them are in tdd, where they belong." (The system's card in prod said "17 deployments, 1 failed" on 2026-10-08.)
7. Show a system pull request's checks, for example https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/pull/58 . The required check is "env-checks": a secret scan, the rules of `system.json`, the Bicep build, Terraform format and validate. Its job "preview" shows what would change in Azure per environment and never blocks.

If slow or red: if Octopus is slow to load, stay on the "Delivery" cards; they carry the same facts. The cards are read every five minutes and are as old as the file's last change (hover a card's title).

## 5. Resilience: the failover test (6 min)

Say this first, before anything is started: "I am going to stop the primary web app of uat on purpose. uat's public address should keep answering from the standby. The runbook starts the primary again by itself. Nothing is deleted, and prod is not touched."

The runbook "Failover test" belongs to the Octopus project cmdemo2-system. It stops uat's primary web app, polls the public address every 3 seconds until three answers in a row come from the standby, starts the primary again (always, also after a failure), and waits for the primary to serve again. It measures the web tier only: the database is one for both regions.

Set-up (1 min):

1. Use two windows side by side. The dashboard stops checking while its tab is hidden, so it must stay visible while you are in Octopus.
2. Left window: the uat dashboard, https://gentle-wave-0b8a73410.2.azurestaticapps.net/#runtime/uat . Set "Interval" to "10 s". Do not reload the page until the test is over: a reload empties "What just happened".
3. Point at the line under "What just happened": "Last failover test: uat, 23 h ago: the standby answered after 40 s" (before the rehearsal on 2026-10-08, which measured 47 s). Say: "That is the last measurement. Let us make a new one."
4. Right window: Octopus, project cmdemo2-system, "Operations", "Runbooks", "Failover test", run it in uat (sign-in). The expected address is https://clearmeasure.octopus.app/app#/Spaces-356/projects/cmdemo2-system/operations/runbooks ; it was not opened here.

What the class sees. The times are from the rehearsal on 2026-10-08, watched with this dashboard at a 10 s interval; expect the same order, not the same seconds. "What just happened" puts the newest line on top, with the time and the place ("uat · westus3", "uat · Front Door", "uat · ui") in front of each line.

| When | On the Runtime view of uat | Say |
|---|---|---|
| Start to about 1 or 2 min | Nothing changes. Octopus starts a worker; the script first waits until the address is served by the primary. In the rehearsal on 2026-10-08 the first change came 1 min 28 s after the run was started. | "A good test checks its starting point first." |
| T+0, the first check after the stop | The header turns to "2 of 8 nodes not healthy". The frame "westus3" turns red and dashed, with "not serving". The box "app-cmdemo2-uat-ui" turns dark grey with a dotted edge: "Unreachable", "no answer", "pinned 2.4.18: not compared", "primary: not serving"; its numbers and its row of check marks are gone. The frame "eastus2" turns green: "serving traffic"; its web app says "standby: serves traffic". The arrow "origin, priority 1" turns red and dashed and shows "– calls/min"; "origin, priority 2" turns solid green, and so does the standby's arrow to the database. | "The primary is gone. The page already expects the standby to serve." |
| T+0, same check | The Front Door endpoint also said "Unreachable", "no answer", with "routes to eastus2 (failed over)" and "disagrees with the web apps"; the browser's arrow to it turned red and dashed. "What just happened" lists "Failover: westus3 → eastus2. Primary westus3 is unreachable; eastus2 is expected to serve traffic." and, for westus3 and for Front Door, "ui: Healthy → Unreachable: No answer within 10 s". | "This gap is the outage a user would feel. Front Door has not yet noticed. It probes each origin every 30 seconds." |
| T+11 s, the next check | For Front Door: "ui: Unreachable → Healthy (HTTP 200)". In the rehearsal on 2026-10-08 it stayed healthy; in the run of 2026-10-06 it flipped once more and was steadily healthy 30 s after the first event. The endpoint then says "routes to eastus2 (failed over)" and "agrees with the web apps". The header says "1 of 8 nodes not healthy". | "Same address, other region. Under a minute, and nobody changed DNS or told a client anything." |
| T+2 min 26 s | Three lines in one check: "Failback: eastus2 → westus3. The primary is healthy again." and, for westus3, "ui: Unreachable → Healthy (HTTP 200)" and "ui restarted, up 22 s". The green line is back on priority 1. The header returns to "All 8 nodes healthy". The primary's box says "restarted 22 s ago" where "up … h" was, for five minutes. | "The runbook started the primary again, and Front Door went back to priority 1 on its own." |

No line about a single health check (a named entry such as "DataAccess" changing state) appeared in the rehearsal: the stopped web app showed no check marks at all, and after the restart the first check found "8 checks healthy".

Afterwards (1 min):

1. In Octopus, open the run's task summary. It has two highlighted lines; in the rehearsal on 2026-10-08 they were "Failover of ui in uat: https://cmdemo2-uat-ui-gjfyh5dcf2e2aagg.z02.azurefd.net answered from the standby (eastus2) 47 s after app-cmdemo2-uat-ui stopped; 6 of 8 requests failed meanwhile." and "Failback of ui in uat: served by the primary (westus3) again 106 s after it was started; 0 of 28 requests failed meanwhile." (read from the task's log, not on the Octopus screen).
2. Say: "The claim 'we have a standby' is now a number with a date. This runbook is also scheduled monthly in uat, so the number never gets old."
3. The line "Last failover test" on the dashboard changes only after the workflow "delivery" has run again (hourly). Do not wait for it.
4. Reload the dashboard now. In the rehearsal on 2026-10-08 the page that had watched the test kept two wrong colours after the failback, until it was reloaded: the mark "serving traffic" on westus3 stayed red and "standby: ready" on eastus2 stayed green. The words were right.

Timing: in the rehearsal on 2026-10-08 the runbook measured 47 s to the standby (40 s and 44 s in the two tests before, by the dashboard's line) and 106 s for the way back; the task took 4 minutes in all. On the page the run took 2 min 26 s from the first event to the failback.

Optional: while the primary is down, check that the Traffic panel's "Environment" says "uat" (it does on `#runtime/uat`) and press "Generate traffic". In the rehearsal on 2026-10-08, pressed 15 s after the first event, the counter began "9 sent, 4 answered; 56 s left." and the run ended "Traffic ended: 119 sent, 119 answered". The numbers appear on "origin, priority 2" (19, then up to 116 "calls/min"), and "origin, priority 1" keeps "– calls/min".

If slow or red:

- Nothing changes after three minutes: look at the task in Octopus. If the script says "Nothing was stopped", it refused to start because the address or the standby did not answer; say so and move on.
- The primary stays "Unreachable" after five minutes: the runbook waits up to 10 minutes for the way back. uat's address keeps answering from the standby meanwhile. Tell the system's owner; do not start, stop or apply anything by hand.
- Do not run this in prod in front of a class, and do not run it across the minute of the hourly "Health report": that report would probably count uat as not healthy for that hour. The report starts at 13 minutes past every hour and ends about 75 s later (Octopus's task list on 2026-10-08, and `octopus/runbooks.tf`): start the test between a quarter past and five to the hour. In the rehearsal it was started at 14:35 past, just after the report.

## 6. Operations (3 min)

1. Availability. On the Health tab, under an environment's name: "Availability Healthy in 24 of 24 hourly checks (100 %) in 24 hours", "54 of 55 in 7 days", "last failure 30 h ago", and the sentence "Hourly checks by the pipeline, not continuous monitoring."
   - Say: "An hourly runbook, 'Health report', asks every node and the public address. This line counts its runs. The page says itself that this is not monitoring: an outage between two reports is not counted."
   - If asked about the one failure in uat and prod: it ended on 2026-10-06 at about 18:14 UTC. The page does not say why. Not shown here.
2. Cost. In the header: "Cost of the system $2.97 yesterday · $9.26 in 7 days · $9.26 this month · as of 2026-10-07". Under each environment: "Cost" and "Most this month: …". After prod on the Health tab: "shared", "no environment".
   - Say: "Cost per environment, by the tag `environment` on the resources. It is a day old, and the page says so. What no environment owns is listed apart: mostly Front Door."
   - "yesterday" is the last complete day in UTC (hover the line), so in a US evening it is the day still on the clock.
3. Capability checks. Open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/actions/workflows/capabilities.yml
   - Say: "Every night, and at the end of every system build, one script proves each capability the system claims, read-only, against GitHub, Octopus and Azure: branch rules, pins, deployment stacks, roles, runbook results, clean deployment logs. 51 checks."
   - The last scheduled run on 2026-10-07 was green.
   - What a red check looks like: open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/issues/56 . The title is "Capabilities not proven". The body links the red run and quotes its lines, for example "FAIL CAP-045: stack-cmdemo2-prod lists no web app for ui (a failed or unfinished apply?)" and "3 of 47 capabilities failed."
   - Say: "A red run opens an issue with the label 'capability'. The next green run closes it. Nobody has to read workflow logs to know." This one was open for about an hour. The list: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/issues?q=label%3Acapability
4. Drift. Open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/actions/workflows/drift.yml
   - Say: "Every night a what-if compares Git with every environment. Red means an environment differs from Git, and the job summary lists what. The repair is to deploy the latest cmdemo2-system release there again. Hand changes are mostly impossible anyway: the deployment stacks deny them."
   - The last run was green. No issue with the label "drift" exists in the repository today.
5. One sentence for the other workflows, only if asked: "delivery" publishes `delivery.json` and `cost.json` to the branch `status`; "kit-templates" proposes template updates as a pull request and never merges; "fleet-findings" keeps the system's differences from the fleet's rules as issues.

If slow or red: if the last capabilities or drift run is red on the day, show it. Read the FAIL line aloud and say which capability it names. That is the lesson of this step.

## 7. The fleet page (2 min)

1. Open https://stcmfleetprodw35dyr.z19.web.core.windows.net (the earlier address, red-sand-…azurestaticapps.net, answers "404: Not Found").
   - Point at the counters at the top (on 2026-10-08: "6 SYSTEMS", "0 PRODUCTION AFFECTED", "0 NEED ATTENTION", "6 BEHIND THE STANDARD", "0 BROKEN", "1 GAPS", "122.68 USD THIS MONTH") and at the line "Read 13 min ago." They moved within half an hour of the rehearsal: read them off the page.
   - Say: "This is the same idea one level up: every demo system, what it runs in prod, what it costs, and where it differs from what the fleet declares."
2. Point at "LANDSCAPE" and its legend ("as declared", "needs attention", "behind the standard", "production affected", "not read", "asleep"). Click the box "cmdemo2": it opens the prod dashboard's Runtime view of prod, in the same tab.
3. Go back. Under "SYSTEMS", point at the card "cmdemo2": its state ("AS DECLARED" at one reading on 2026-10-08, "BEHIND THE STANDARD" half an hour later), the three projects with their versions and "44 h in prod", "Standards: 2 behind · 1 not checked · 10 met" and "health: healthy (asked just now)".
   - If the card says "BEHIND THE STANDARD" or "NEEDS ATTENTION", read the reason under it aloud, for example "Kit templates: 2 file(s) behind the kit since 2026-10-08 04:04 UTC, within the 24 hours a system has to follow". It is a finding of the fleet: a template behind the kit, a release that prod does not have yet, a runbook that has not passed in its period.
   - Say: "Healthy and 'behind the standard' can be true at once. The app answers, and a promise is still open. The fleet shows the difference instead of hiding it."
4. Click the line under the card that ends in "every standard · releases, cost, links". It opens a table "STANDARD", "STANDS", "WHAT THE FLEET READ" with one row per standard, the versions per environment, the cost this month ("9.26 USD this month"), "Owner: …", and the links "Runtime view", "Its dashboard", "Repository", "Octopus space", "Findings".
   - "Findings" leads to issues of the kit's repository, which is private: it will not open for the class.
5. Point at "SHARED BY EVERY SYSTEM": the limits, for example "7 of 10 Static Web Apps on the Free plan" and, with "full: the next one cannot be made", "2 of 2 Container Apps environments" in southcentralus.

If slow or red: the page is a snapshot ("Read … ago"). If cmdemo2's card differs from what the dashboard shows, trust the dashboard for the live state and the fleet page for the findings.

## If asked

**What does it cost?** Read the header: on the last complete day shown (2026-10-07), $2.97 for the whole system; per environment $0.33 (tdd), $0.68 (uat), $0.87 (prod) and $1.09 shared, of which Front Door is the largest part. The sizes are on the diagram and in `system.json`: App Service plans B1, databases on the Basic tier. "This month" is small because the system was created on 2026-10-04. The fleet page showed "9.26 USD this month" on 2026-10-08, the same as the dashboard's "this month". A monthly total or a forecast is not shown here.

**What happens if the primary region dies for real?** For the web tier, what the failover test shows: Front Door probes both origins every 30 seconds and sends traffic to the standby in eastus2, which already runs the same version. The test in the rehearsal on 2026-10-08 measured 47 seconds, the one before 40. Three limits, all visible on the diagram. The database is one for both regions and lives in centralus, so losing westus3 leaves it alone, and losing centralus takes it away from both web apps. tdd has no standby. The dashboards are static sites in centralus. A second copy of the database in another region is not shown here. What the repository does prove for the data is a restore: the runbook "Restore test" restores the database to a point in time into a temporary copy, checks it and deletes it. It runs every Sunday; on 2026-10-08 it passed in 19 minutes (12 tables, 113 rows).

**How is a secret rotated?** By a runbook, "Rotate SQL password", scheduled monthly in every environment. By its script: it generates a 32-character password, sets it on the SQL server, writes it and the connection string to the environment's Key Vault, restarts the apps so they read the new value, and checks that every app answers its health path. No secret is in Git: the README says an app gets a secret by name only and reads the value from the vault by reference. When it last ran, and with what result, is in Octopus and is not shown here.

**How do I add an environment?** By pull request to cmdemo2-system, as its README says under "Common changes": append the environment to `environments` in `system.json` with its tier, and add `environments/<env>/versions.json` containing `{}`. After the merge, promote the new cmdemo2-system release to it in Octopus, then the app's release. The pull request's "preview" job shows what Azure would create. The limit a new environment's dashboard counts against is on the fleet page: "7 of 10 Static Web Apps on the Free plan" on 2026-10-08, so there is room for a fourth. Read the kit's `docs/start-a-new-system.md` before creating a whole new system; that page is not public.

**Who can change prod by hand?** By the README: nobody except the deploy identity. Each environment is a deployment stack with deny settings. The nightly drift check would show a difference. The role assignments themselves are not shown here.

**Why is there a warning mark although everything is healthy?** Two places today: "Qodana 1 problem" on the "Code" card, and "Last 7 days … failed" on a "Delivery" card. Neither is a health state. The header counts only nodes that do not answer HTTP 200.

**Where is the LLM gateway?** The app has a health check named "LlmGateway" for its chat feature. The feature is not configured in this system, so the diagram draws no box for it: a box that says "reachable" about something that is switched off would mislead.
