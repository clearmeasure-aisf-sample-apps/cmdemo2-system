# CM demo 2 multi-region (cmdemo2): presenter's walkthrough

A 20 to 30 minute tour of a live system, for a class on DevOps and GitOps. Every label in quotes is the page's own wording, read on 2026-10-07 and 2026-10-08 and rehearsed against the live pages on 2026-10-08. A second pass later on 2026-10-08 followed every step again and compared each label with the page; it did not run the failover test of step 5. That test was run again on 2026-10-09, and step 5's timeline is that run's; ui 2.4.22 ran then, and the quotes of sections 2, 5 and 6 are as they were observed on those days. The quotes of sections 1, 3, 4 and 7 that the releases of the following hours changed were read again on 2026-10-09 at about 05:00, after the app release 2.4.24 (pull request 11 of cmdemo2-workorders), which is the worked example. Then every environment ran ui 2.4.24, cmdemo2-system 1.0.51 and cmdemo2-dashboard 1.0.39. Later that morning cmdemo2-system 1.0.52 and cmdemo2-dashboard 1.0.41 gave every system's dashboard the fleet dashboard's look (a dark navy page with the Clear Measure logo in a bar at the top) and made every box of the Runtime view a link; the passages about the look, the frames and the links are from a reading after that release (dates and times in UTC). Numbers are from those days and will differ on the day: read them off the page, do not recite them from here.

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
4. On each environment, read the banner: "Expected to serve traffic: westus3 (primary)", "Front Door agrees: it is healthy.", "Pinned 2.4.24", "In sync: all 2 nodes run 2.4.24." (the version will differ; tdd has one node and says "In sync: westus3 runs 2.4.24.").
5. Use a browser window at least 1400 px wide; at 1300 px the right edge of the diagram is out of view until its box is scrolled sideways (the page fits the diagram to the width, down to 70 % of its size). The Runtime view opens with the diagram fitted to the page. The button at the right says "Actual size": leave it. A press shows the diagram 1880 px wide, which scrolls sideways; the button then says "Fit to width" and a second press brings it back.
6. Leave "Probe" on "Health check". "Liveness" hides the health check marks and the database's state.
7. Open the fleet page and read cmdemo2's card, so that its state is no surprise (a card that says "Behind the standard", or another state than "As declared", names the reason under it).
8. Decide who starts the failover test in step 5: you, signed in to Octopus, or the system's owner on your word. Check that the person can run runbooks in the space (not verified here).
9. Run nothing destructive. No runbook except "Failover test" in uat, and that one only in step 5 and only after saying so. Nothing that removes Front Door or a public address: the system keeps them.
10. Leaving a browser on a dashboard is fine. The page stops checking while its tab is hidden.

The dashboard is a Blazor WebAssembly page: its first load takes a few seconds, and the header says "Checking 8 nodes", then for a moment a count such as "5 of 8 nodes healthy, 3 being checked", and "Last refresh: not yet" until every node has answered.

## 1. What is running (3 min)

1. Open the app in prod: https://cmdemo2-prod-ui-a2b5hkfrchg3ckew.z02.azurefd.net
   - Point at: the browser tab's title "Clear Measure - .NET Bootcamp - Work Order Application", the menu ("Home", "Health Check", "Work Order Management", "Church Bulletin", "Login"), and the last line of the page, which shows the version and the commit, for example `2.4.24 · 8ce8050`. The commit is a link to its page in cmdemo2-workorders on GitHub; it appears a moment after the version.
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

1. On the tdd dashboard, click the tab "Runtime", then the button "prod". Address: https://calm-wave-0b8361f10.6.azurestaticapps.net/#runtime/prod . The diagram is fitted to the page; leave the button "Actual size" alone.
   - Say: "Same checks, drawn as a C4 deployment diagram. The boxes were drawn when the dashboard was deployed. The colours, words and numbers are live."
2. Point at the frames, left to right. Since dashboard 1.0.41 two frames surround them: "Azure subscription [subscription]" around everything but the browser, and "rg-cmdemo2-prod [resource group]" around the three regions.
   - "rg-cmdemo2-edge" with "afd-cmdemo2 [Front Door profile, Standard: global]" and the endpoint "cmdemo2-prod-ui".
   - "westus3 [Azure region: primary]" with the mark "serving traffic", and under it "eastus2 [Azure region: standby]" with "standby: ready". Each holds an App Service plan, size B1, and a web app.
   - "centralus [Azure region: data, static sites]": the database "sqldb-cmdemo2-prod" and the dashboard's Static Web App.
   - Say: "Two regions run the app. One database serves both, and it lives in a third region. Keep that in mind for the failover."
3. Point at the Front Door endpoint's tile: "routes to westus3 (priority 1)" and "agrees with the web apps".
   - Then the two arrows out of it: "origin, priority 1" with "first, while healthy" (solid green: it carries the traffic) and "origin, priority 2" with "when priority 1 is down" (dotted grey: idle). Each line ends with Front Door's health probes of the last minute, for example "· 68 probes".
   - Say: "Front Door probes both origins and sends traffic to the healthy one with the lowest priority. A failover is the green line moving from priority 1 to priority 2."
4. Point at the numbers on the arrows: "0 calls/min" from the browser, "0 calls/min" to each origin, and a few "calls/min" on "reads and writes [TCP 1433]" with "app queries · 54 background" (2 to 9 at the second pass on 2026-10-08; 20 to 30 at the first rehearsal).
   - Say: "Each web app counts its own last minute and publishes it at `/_telemetry`. Nobody is using the app, so requests are zero. The database still gets queries: the health checks, and a message bus that polls in the background."
5. Switch to "tdd" (button "tdd"; address `#runtime/tdd`). Scroll to the bottom: "Traffic". Check that "Environment" says "tdd". Press "Generate traffic" once.
   - What it does, in the page's words: "2 requests a second for 60 s from this browser to ui at cmdemo2-tdd-ui-…azurefd.net: the start page and the app's representative reads".
   - The button becomes "Stop traffic". Under it a counter runs, for example "3 sent, 2 answered, 1 on its way; 59 s left." and later "118 sent, 118 answered; 1 s left.", and it ends "Done: 120 sent, 120 answered." The numbers always add up: a request is answered, failed, on its way, or given up "with no answer within 10 s"
   - "What just happened" gets a line: "Traffic started: 2 requests a second for 60 s to ui at …".
6. Scroll back up to the diagram and watch for one minute. Observed here on 2026-10-07:
   - After 20 s: "21 calls/min" on browser to Front Door and on "origin, priority 1"; the web app's tile says "21 req/min · p95 65 ms"; the database arrow rose from 17 to 33.
   - After 40 s: 60. After 60 s: 117 calls/min, 92 on the database arrow, CPU about 2 %.
   - Then "Traffic ended: 118 sent, 118 answered" (120 and 120 on 2026-10-09). The numbers fall again over the next minute, because they count the last minute.
   - Say: "Follow one request: browser, Front Door, the origin that serves, the database. The small line next to each number is its trend since this page was opened."
   - During the traffic the page checks every 10 s, although "Interval" still shows "30 s".
7. Point at the web app's tile for the process vitals: "CPU 1.1 % · 154 MB · 3 in flight", "0 errors · 0 exceptions/min", "up 30 h".
   - Say: "Uptime is the cheapest restart detector there is. Under five minutes the page writes 'restarted' and marks it as a warning."
8. Point at the row of eight small marks and "8 checks healthy". Hover one mark for its name and words. The names are on the Health tab: "API", "DataAccess", "Jeffrey", "LlmGateway", "NeedsReboot", "ProcessThreadCount", "Server", "self".
   - Say: "A health check that answers only 'Healthy' tells you nothing when it fails. This one answers entry by entry, so a red tile says which dependency is the reason."
   - If asked what "LlmGateway" is: the app's chat feature. It is not configured in this system (the check's words are "Chat client is not configured"), so the diagram draws no box for it.
9. Point at any underlined name or number, and at any box. Say: "Each one is a deep link: the web app in the Azure portal, Live Metrics, the release in Octopus. And every box opens what it stands for when you click anywhere on it: the subscription, a resource group, the Front Door profile, an App Service plan, the Static Web App; a region opens a query for the resources in it. They open in a new tab and ask you to sign in." Open one only if you signed in beforehand; the portal addresses behind the boxes were not followed past the sign-in here.

If slow or red: if the numbers do not move, wait for the next check (10 s) or press "Check now". If answers are slow, the counter says so instead of losing them: after the minute it reads, for example, "120 sent, 110 answered, 10 on their way; waiting for their answers." and ends when the last one has arrived, or names how many got "no answer within 10 s". (Until dashboard 1.0.36 a run with slow answers ended "119 sent, 99 answered" although the app had received every request; 1.0.38 counts them.) If "Environment" in the Traffic panel shows another environment, set it back to tdd. On the Runtime view the dropdown follows the buttons "tdd", "uat", "prod" (on `#runtime/prod` it says "prod"); on the Health tab it starts on "tdd" on every dashboard.

## 3. Where it comes from (4 min)

1. Stay on the Runtime view and scroll under the diagram to the card "Code" ("ui in tdd"). It is also on the Health tab inside each environment.
   - Point at the first line: "build 2.4.24", "commit 8ce8050", "built 98 min ago", "build run".
   - Point at the rest: "82,963 lines in 1,116 files" with the language bar, "Tests 1,594: 1,103 unit, 339 integration, 152 acceptance", "Coverage 90.1 % of lines, 81.9 % of branches", "Complexity average 2, worst 42 (1,386 methods)", "CRAP worst 6, none over 6", "Qodana 1 problem" (with a warning mark).
   - Say: "The running app answers these numbers itself, at `/_build`. They are the facts of the build that produced this exact binary, not of the latest commit."
   - Below the "Delivery" cards is a second card "Code", "dashboard (this page)": "build 1.0.41", "21,713 lines in 141 files", "Tests 882 unit", "Coverage 76.5 % of lines, 67 % of branches". Say: "The dashboard holds itself to the same rule: it publishes the facts of its own build."
2. Click "commit 8ce8050". It opens the commit in cmdemo2-workorders on GitHub (no sign-in).
   - It came from pull request 11: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-workorders/pull/11
3. Go back and click "build run". It opens the run "2.4.24 • Build • master" of the workflow "Build".
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
   - It holds two lines: `"dashboard": "1.0.41"` and `"ui": "2.4.24"`.
   - Say: "This file is the GitOps pin. Git says prod runs ui 2.4.24. The dashboard reads this file and compares it with what the nodes answer: 'Pinned 2.4.24', 'In sync: all 2 nodes run 2.4.24.'"
7. On the Health tab, click "Pin history" in prod's banner. It opens the commits of that file.
   - Point at messages such as "Pin dashboard 1.0.41 in prod (Deployments-…)" by `cmdemo2-system[bot]`, the system's own GitHub App, which commits the pins since 2026-10-09 (the first at 04:16 UTC, in tdd). "Pin ui 2.4.24 in prod (Deployments-…)", further down, and everything older are by the machine user `cm-ai-ops-bot`.
   - Say: "People never edit this file. Octopus commits the pin as the first step of a deployment, and reverts it if a later step fails. The history of this file is the history of prod."
8. Optional: open the picture of the whole chain, "What depends on what": https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/blob/main/docs/architecture/20-dependencies.png . The README of that folder shows it too, as its last picture, under the heading "What depends on what: the system, its GitOps repositories and its DevOps pipeline (2026-10-06)".

If slow or red: if a card "Code" is missing, the node did not answer `/_build`; use another environment's card, they show the same build while the versions are in sync.

## 4. How a change travels (4 min)

Tell it as one story, with pull request 11 of the app as the example, and show the evidence in this order.

1. The pull request: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-workorders/pull/11 . It is merged: point at the tab "Checks 14" and at the line "14 checks passed" under "merged commit 8ce8050 into master".
   - Say: "Nobody pushes to master. The branch rule asks for a pull request and for two checks: 'Build result' and 'secret-scan'."
2. The build and the release: the "build run" link from step 3. After a green "Build" of master, the workflow "Release" pushes the package to Octopus and creates release 2.4.24 of the project cmdemo2-ui.
   - Say: "GitHub builds and tests. Octopus deploys. The hand-over is one versioned package."
3. Octopus (sign-in): https://clearmeasure.octopus.app/app#/Spaces-356/projects/cmdemo2-ui . Point at the release's row: tdd, uat, prod.
   - Say: "The lifecycle is tdd, then uat, then prod. tdd is automatic. uat and prod wait for a promotion, and their first step is 'Sign-off'."
   - Open the release (the version on a "Delivery" card links to it) and one deployment's task log. The steps, in the order of the task log of release 2.4.24: "Sign-off" (uat and prod), "Record restore point" (prod only), "Pin version", "Migrate database", "Prepare test runner" (tdd only), "Seed demo employees" (uat and prod), "Update deployable", "Verify deployable", "Revert pin" (skipped unless a step failed), and in tdd then "Open test database", "Acceptance tests" and "Close test database".
   - The exact Octopus screens were not opened here: they need a sign-in. The step names were read from Octopus through its API on 2026-10-08, and for release 2.4.24 again on 2026-10-09.
4. Back on the dashboard, Health tab, the three cards "Delivery" "ui in tdd", "ui in uat", "ui in prod".
   - tdd: "Deployed 2.4.24", "Lead time 24 min from commit 8ce8050 to this deployment". There is no "Signed off" line: tdd has no sign-off step.
   - uat: "Signed off by ai-ops", the reason "Footer and login page: the version never shows the commit the SDK appends to it (cmdemo2-workorders 11)", "Lead time 36 min", "Compared same as tdd".
   - prod: the same sign-off and reason, "Lead time 54 min".
   - Say: "Who deployed what, when, with whose sign-off and for what reason, and how long a commit took to reach each environment: 24, 36 and 54 minutes. The page reads this from a file the pipeline publishes, `delivery.json` on the branch `status` of the system repository."
   - Say, if asked who "ai-ops" is: "The operator's automation account. It signs off only with a recorded reason. People who may sign off are named in `system.json` under `octopus.approvers`; it names one today, and he signed off one promotion to prod himself: cmdemo2-dashboard 1.0.30 on 2026-10-08, which waited sixteen hours for him. While it waited, the dashboards said 'cmdemo2-dashboard 1.0.30 waits for a sign-off in prod' under the environment buttons. The cards show the newest deployment of each environment; the card 'dashboard in prod' keeps his sign-off in a line of its own: 'Last by a person 1.0.30 by jeffreypalermo'."
5. Point at the other two "Delivery" cards per environment: "the system (infrastructure and pipeline)" and "dashboard".
   - Say: "Infrastructure travels the same road. A merge to cmdemo2-system makes a release of the project cmdemo2-system; Octopus applies `infra/` to tdd, then to uat and prod after a sign-off."
6. Point at "Last 7 days" on the cards of ui: for example "14 deployments, 3 failed" in tdd and "8 deployments, 1 failed" in uat, each with a warning mark, and "7 deployments, none failed" in prod.
   - Say: "Failures show, and most of them are in tdd, where they belong." (The system's card in prod said "24 deployments, 1 failed" on 2026-10-09.)
7. Show a system pull request's checks, for example https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/pull/58 . The required check is "env-checks": a secret scan, the rules of `system.json`, the Bicep build, Terraform format and validate. Its job "preview" shows what would change in Azure per environment and never blocks.

If slow or red: if Octopus is slow to load, stay on the "Delivery" cards; they carry the same facts. The cards are read every five minutes and are as old as the file's last change (hover a card's title).

## 5. Resilience: the failover test (6 min)

Say this first, before anything is started: "I am going to stop the primary web app of uat on purpose. uat's public address should keep answering from the standby. The runbook starts the primary again by itself. Nothing is deleted, and prod is not touched."

The runbook "Failover test" belongs to the Octopus project cmdemo2-system. It stops uat's primary web app, polls the public address every 3 seconds until three answers in a row come from the standby, starts the primary again (always, also after a failure), and waits for the primary to serve again. It measures the web tier only: the database is one for both regions.

Set-up (1 min):

1. Use two windows side by side. The dashboard stops checking while its tab is hidden, so it must stay visible while you are in Octopus.
2. Left window: the uat dashboard, https://gentle-wave-0b8a73410.2.azurestaticapps.net/#runtime/uat . Set "Interval" to "10 s". Do not reload the page until the test is over: a reload empties "What just happened".
3. Point at the line under "What just happened": "Last failover test: uat, 22 h ago: the standby answered after 47 s" (before the rehearsal on 2026-10-09, which measured 62 s). Say: "That is the last measurement. Let us make a new one."
4. Right window: Octopus, project cmdemo2-system, "Operations", "Runbooks", "Failover test", run it in uat (sign-in). The expected address is https://clearmeasure.octopus.app/app#/Spaces-356/projects/cmdemo2-system/operations/runbooks ; it was not opened here.

What the class sees. The times are from the rehearsal on 2026-10-09 (the runbook was started at 02:21:05 UTC), watched with this dashboard at a 10 s interval; expect the same order, not the same seconds: the rehearsal a day earlier went through the same steps faster (11 s to the first healthy answer through Front Door, 2 min 26 s to the failback). "What just happened" puts the newest line on top, with the time and the place ("uat · westus3", "uat · Front Door", "uat · ui") in front of each line.

| When | On the Runtime view of uat | Say |
|---|---|---|
| Start to about 1 or 2 min | Nothing changes. Octopus starts a worker; the script first waits until the address is served by the primary. In the rehearsal on 2026-10-09 the first change came 1 min 30 s after the run was started (1 min 28 s the day before). | "A good test checks its starting point first." |
| T+0, the first check after the stop | The header turns to "2 of 8 nodes not healthy". The frame "westus3" turns red and dashed, with "not serving". The box "app-cmdemo2-uat-ui" turns dark grey with a dotted edge: "Unreachable", "no answer", "pinned 2.4.22: not compared" (2.4.22 was the pinned version at the rehearsal; the box names the version pinned on the day), "primary: not serving"; its numbers and its row of check marks are gone. The frame "eastus2" turns green: "serving traffic"; its web app says "standby: serves traffic" and keeps "8 checks healthy". The arrow "origin, priority 1" turns red and dashed and shows "– calls/min"; "origin, priority 2" turns solid green, and so does the standby's arrow to the database. | "The primary is gone. The page already expects the standby to serve." |
| T+0, same check | The Front Door endpoint also said "Unreachable", "no answer", with "routes to eastus2 (failed over)" and "disagrees with the web apps"; the browser's arrow to it turned red and dashed. "What just happened" lists "Failover: westus3 → eastus2. Primary westus3 is unreachable; eastus2 is expected to serve traffic." and two more lines: for westus3 "ui: Healthy → Unreachable: No answer within 10 s", and for Front Door "ui: Healthy → Unreachable: The browser could not read an answer: the network failed, or the node does not allow this origin (CORS)" (Front Door answers with its own error page while it still routes to the stopped origin; on 2026-10-08 that line also read "No answer within 10 s"). | "This gap is the outage a user would feel. Front Door has not yet noticed. It probes each origin every 10 seconds." |
| T+41 s (T+11 s on 2026-10-08) | For Front Door: "ui: Unreachable → Healthy (HTTP 200)". In both rehearsals it then stayed healthy; in the run of 2026-10-07 at 04:44 UTC it flipped once more and was steadily healthy 30 s after the first event. The endpoint then says "routes to eastus2 (failed over)" and "agrees with the web apps". The header says "1 of 8 nodes not healthy". | "Same address, other region. Under a minute, and nobody changed DNS or told a client anything." |
| T+2 min 44 s (T+2 min 26 s on 2026-10-08) | Three lines in one check: "Failback: eastus2 → westus3. The primary is healthy again." and, for westus3, "ui: Unreachable → Healthy (HTTP 200)" and "ui restarted, up 30 s". The green line is back on priority 1. The header returns to "All 8 nodes healthy". The primary's box says "restarted 30 s ago" where "up … h" was, for five minutes. | "The runbook started the primary again, and Front Door went back to priority 1 on its own." |

No line about a single health check (a named entry such as "DataAccess" changing state) appeared in either rehearsal: the stopped web app showed no check marks at all, and after the restart the first check found "8 checks healthy".

Afterwards (1 min):

1. In Octopus, open the run's task summary. It has two highlighted lines; in the rehearsal on 2026-10-09 they were "Failover of ui in uat: https://cmdemo2-uat-ui-gjfyh5dcf2e2aagg.z02.azurefd.net answered from the standby (eastus2) 62 s after app-cmdemo2-uat-ui stopped; 6 of 11 requests failed meanwhile." and "Failback of ui in uat: served by the primary (westus3) again 114 s after it was started; 4 of 19 requests failed meanwhile." (read from the task's log, not on the Octopus screen). The day before they read 47 s with "6 of 8 requests failed" and 106 s with "0 of 28 requests failed": the way back is not always free of failed requests, and the line says so.
2. Say: "The claim 'we have a standby' is now a number with a date. This runbook is also scheduled monthly in uat, so the number never gets old."
3. The line "Last failover test" on the dashboard changes only after the workflow "delivery" has run again (hourly). Do not wait for it.

Timing: in the rehearsal on 2026-10-09 the runbook measured 62 s to the standby (47 s, 40 s and 44 s in the three tests before) and 114 s for the way back; the task took 5 minutes in all. On the page the run took 2 min 44 s from the first event to the failback.

Optional (not repeated on 2026-10-09): while the primary is down, check that the Traffic panel's "Environment" says "uat" (it does on `#runtime/uat`) and press "Generate traffic". In the rehearsal on 2026-10-08, pressed 15 s after the first event, the counter began "9 sent, 4 answered; 56 s left." and the run ended "Traffic ended: 119 sent, 119 answered". The numbers appear on "origin, priority 2" (19, then up to 116 "calls/min"), and "origin, priority 1" keeps "– calls/min".

If slow or red:

- Nothing changes after three minutes: look at the task in Octopus. If the script says "Nothing was stopped", it refused to start because the address or the standby did not answer; say so and move on.
- The primary stays "Unreachable" after five minutes: the runbook waits up to 10 minutes for the way back. uat's address keeps answering from the standby meanwhile. Tell the system's owner; do not start, stop or apply anything by hand.
- Do not run this in prod in front of a class, and do not run it across the minute of the hourly "Health report": that report would probably count uat as not healthy for that hour. The report starts at 13 minutes past every hour and ends about 75 s later (Octopus's task list on 2026-10-08, and `octopus/runbooks.tf`): start the test between a quarter past and five to the hour. The rehearsal on 2026-10-09 was started at 21 minutes past, seven minutes after the report had ended.

## 6. Operations (3 min)

1. Availability. On the Health tab, under an environment's name: "Availability Healthy in 24 of 24 hourly checks (100 %) in 24 hours", "54 of 55 in 7 days", "last failure 30 h ago", and the sentence "Hourly checks by the pipeline, not continuous monitoring."
   - Say: "An hourly runbook, 'Health report', asks every node and the public address. This line counts its runs. The page says itself that this is not monitoring: an outage between two reports is not counted."
   - If asked about the one failure in uat and prod: it ended on 2026-10-06 at about 18:14 UTC. The page does not say why. Not shown here.
2. Cost. In the header: "Cost of the system $3.26 yesterday · $9.54 in 7 days · $9.54 this month · as of 2026-10-07 (UTC)". Under each environment: "Cost" and "Most this month: …". After prod on the Health tab: "shared", "no environment".
   - Say: "Cost per environment, by the tag `environment` on the resources. It is a day old, and the page says so. What no environment owns is listed apart: mostly Front Door."
   - "yesterday" is the last complete day in UTC (hover the line), so in a US evening it is the day still on the clock.
3. Capability checks. Open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/actions/workflows/capabilities.yml
   - Say: "Every night, and at the end of every system build, one script proves each capability the system claims, read-only, against GitHub, Octopus and Azure: branch rules, pins, deployment stacks, roles, runbook results, clean deployment logs. 50 checks, and two more that wait for their preconditions."
   - The last scheduled run, on 2026-10-08 at 07:06 UTC, was red and opened issue 71 ("3 of 51 capabilities failed (2 skipped: their preconditions do not exist yet)."); a green run closed it 35 minutes later. The newest run, of 2026-10-08 at 21:23 UTC, is green: "All 50 checked capabilities are proven (2 skipped: their preconditions do not exist yet)." (in the job's log, which asks for a GitHub sign-in).
   - What a red check looks like: open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/issues/56 . The title is "Capabilities not proven". The body links the red run and quotes its lines, for example "FAIL CAP-045: stack-cmdemo2-prod lists no web app for ui (a failed or unfinished apply?)" and "3 of 47 capabilities failed."
   - Say: "A red run opens an issue with the label 'capability'. The next green run closes it. Nobody has to read workflow logs to know." This one was open for about an hour. The list: https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/issues?q=label%3Acapability
4. Drift. Open https://github.com/clearmeasure-aisf-sample-apps/cmdemo2-system/actions/workflows/drift.yml
   - Say: "Every night a what-if compares Git with every environment. Red means an environment differs from Git, and the job summary lists what. The repair is to deploy the latest cmdemo2-system release there again. Hand changes are mostly impossible anyway: the deployment stacks deny them."
   - The last run was green. No issue with the label "drift" exists in the repository today.
5. One sentence for the other workflows, only if asked: "delivery" publishes `delivery.json` and `cost.json` to the branch `status`; "deployments" publishes `deployments.json`, the deployments in flight, to the branch `deployments`, from which the dashboards mark what is being deployed; "kit-templates" proposes template updates as a pull request and never merges; "fleet-findings" keeps the system's differences from the fleet's rules as issues.

If slow or red: if the last capabilities or drift run is red on the day, show it. Read the FAIL line aloud and say which capability it names. That is the lesson of this step.

## 7. The fleet page (2 min)

1. Open https://stcmfleetprodw35dyr.z19.web.core.windows.net (the earlier address, red-sand-…azurestaticapps.net, answers "404: Not Found").
   - Point at the counters at the top (on 2026-10-09 at 05:00 UTC: "6 systems", "0 production affected", "0 need attention", "5 behind the standard", "0 broken", "0 gaps", "0 deploying", "153.60 USD this month") and at the line "Read 18 min ago." They move within half an hour: read them off the page.
   - Say: "This is the same idea one level up: every demo system, what it runs in prod, what it costs, and where it differs from what the fleet declares."
2. Click "Landscape" in the menu at the left: the section is below the systems' cards. Point at its legend ("as declared", "needs attention", "behind the standard", "production affected", "not read", "asleep", "deploying", "answers healthy now"). Click the box "cmdemo2": the fleet page shows the prod dashboard's Runtime view of prod inside itself (its address then ends in `#cmdemo2`), under a bar with "← Fleet", "cmdemo2" and "Open the dashboard itself".
3. Press "← Fleet". Under "Systems", point at the card "cmdemo2": its state at the right ("As declared" at one reading on 2026-10-08, "Behind the standard" at the later ones and on 2026-10-09), the three projects, each with its version and the time of its deployment under "TDD", "UAT" and "PROD", a line such as "Standards: 1 behind · 15 met" and "health: healthy (asked just now)". While a deployment runs, and for some minutes after it, the card also has a line such as "deploying cmdemo2-system 1.0.49 to tdd" or "cmdemo2-dashboard 1.0.36 reached prod 4 min ago".
   - If the card says "Behind the standard", or another state than "As declared", read the reason under it aloud, for example "Kit templates: 2 file(s) behind the kit since 2026-10-08 04:04 UTC, within the 24 hours a system has to follow" or, on 2026-10-09, "Private build: not yet: ui names no private build". (At the second pass on 2026-10-08 the reason was "GitHub identity: its workflows and its Octopus steps act in GitHub with the machine user's token: system.json names no App of its own yet (set-system-github-app.ps1)"; the system has its own GitHub App since 2026-10-09, and that standard now stands as met.) It is a finding of the fleet: a template behind the kit, a release that prod does not have yet, a runbook that has not passed in its period, an application that names no private build, a GitHub identity that is not yet an App of the system's own.
   - Say: "Healthy and 'behind the standard' can be true at once. The app answers, and a promise is still open. The fleet shows the difference instead of hiding it."
4. Click the last line of the card, which ends in "every standard · cost, links". It opens a table "STANDARD", "STANDS", "WHAT THE FLEET READ" with one row per standard, then the cost this month ("12.32 USD this month" on 2026-10-09, "11.80 USD this month" at the second pass on 2026-10-08, "9.26 USD this month" at the first rehearsal), "5 parts with a fee running", "Owner: …", and the links "Runtime view", "Its dashboard", "Repository", "Octopus space", "Findings".
   - "Findings" leads to issues of the kit's repository, which is private: it will not open for the class.
5. Click "Limits" in the menu at the left, under "Shared by every system": the limits, for example "10 of 10 Static Web Apps on the Free plan" in centralus and "2 of 2 Container Apps environments" in southcentralus, each with "full: the next one cannot be made" (the first read "7 of 10" at the first rehearsal on 2026-10-08).

If slow or red: the page is a snapshot ("Read … ago"). If cmdemo2's card differs from what the dashboard shows, trust the dashboard for the live state and the fleet page for the findings.

## If asked

**What does it cost?** Read the header: on the last complete day shown (2026-10-07), $3.26 for the whole system; per environment $0.37 (tdd), $0.77 (uat), $0.98 (prod) and $1.14 shared, of which Front Door is the largest part. Azure amends a day's cost for a day or two (hover the line), so the same day read $2.97 earlier. The sizes are on the diagram and in `system.json`: App Service plans B1, databases on the Basic tier. "This month" is small because the system was created on 2026-10-04. The fleet page reads the month to date at its own time, so its number can differ from the dashboard's "this month": "9.26 USD this month" at the first rehearsal on 2026-10-08, and "11.80 USD this month" at the second pass, when the dashboard said "$9.54 this month" as of 2026-10-07. A monthly total or a forecast is not shown here.

**What happens if the primary region dies for real?** For the web tier, what the failover test shows: Front Door probes both origins every 10 seconds and sends traffic to the standby in eastus2, which already runs the same version. The test in the rehearsal on 2026-10-09 measured 62 seconds, the two before 47 and 40. Three limits, all visible on the diagram. The database is one for both regions and lives in centralus, so losing westus3 leaves it alone, and losing centralus takes it away from both web apps. tdd has no standby. The dashboards are static sites in centralus. A second copy of the database in another region is not shown here. What the repository does prove for the data is a restore: the runbook "Restore test" restores the database to a point in time into a temporary copy, checks it and deletes it. It runs every Sunday; on 2026-10-08 it passed in 19 minutes (12 tables, 113 rows).

**How is a secret rotated?** By a runbook, "Rotate SQL password", scheduled monthly in every environment. By its script: it generates a 32-character password, sets it on the SQL server, writes it and the connection string to the environment's Key Vault, restarts each container app so it reads the new value, and checks that it answers its health path. This system has no container app: its web apps sign in to the database with a login of their own, so the rotation restarts nothing here, and since cmdemo2-system 1.0.50 the runbook's last line says so ("SQL administrator password of tdd rotated; no app uses it, so none was restarted."). Say it plainly if asked: what rotates is the administrator's password, which only the pipeline's steps use. The web apps' own login got its password when each environment was created and nothing rotates it yet; and the runbook has run once so far, in tdd on 2026-10-05, with its first scheduled runs on 2026-11-01. No secret is in Git: the README says an app gets a secret by name only and reads the value from the vault by reference. When it last ran, and with what result, is in Octopus and is not shown here.

**How do I add an environment?** By pull request to cmdemo2-system, as its README says under "Common changes": append the environment to `environments` in `system.json` with its tier, and add `environments/<env>/versions.json` containing `{}`. After the merge, promote the new cmdemo2-system release to it in Octopus, then the app's release. The pull request's "preview" job shows what Azure would create. The limit a new environment's dashboard counts against is on the fleet page: "7 of 10 Static Web Apps on the Free plan" at the first rehearsal on 2026-10-08, and "10 of 10 Static Web Apps on the Free plan" with "full: the next one cannot be made" at the second pass that day. Read it on the day: while it says "full", a fourth environment's dashboard has no room on the Free plan. Read the kit's `docs/start-a-new-system.md` before creating a whole new system; that page is not public.

**Who can change prod by hand?** By the README: nobody except the deploy identity. Each environment is a deployment stack with deny settings. The nightly drift check would show a difference. The role assignments themselves are not shown here.

**Why is there a warning mark although everything is healthy?** Two places today: "Qodana 1 problem" on the "Code" card, and "Last 7 days … failed" on a "Delivery" card. Neither is a health state. The header counts only nodes that do not answer HTTP 200.

**Where is the LLM gateway?** The app has a health check named "LlmGateway" for its chat feature. The feature is not configured in this system, so the diagram draws no box for it: a box that says "reachable" about something that is switched off would mislead.
