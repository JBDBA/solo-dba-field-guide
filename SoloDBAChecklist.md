# The (Solo) SQL Server DBA's First 30/60/90 Days

Jordan Brennan - dbafieldguide.com

A practical checklist and guide for succeeding in a new database administrator role.
Your actual timeline and progress rate will vary - and that is ok.

| Timeline    | Focus                    | Primary Goals                                                                |
| ----------- | ------------------------ | ---------------------------------------------------------------------------- |
| Days 1-30   | Orientation & Assessment | Learn the environment, stop the bleeding, protect the data                   |
| Days 31-60  | Stabilize & Observe      | Monitor, close maintenance gaps, validate recoverability against RPO/RTO     |
| Days 61-90  | Optimize & Build Value   | Tune configuration, consolidate tooling, tighten security, build the roadmap |

---

## Days 1-30: Orientation & Assessment

### Discovery

- [ ] Identify SQL instances via onboarding handoff, or if needed - via discovery (`Find-DbaInstance`)
  - [ ] Ask about cloud/PaaS databases (Azure SQL Database, Managed Instance, Amazon RDS) - discovery scans won't find these
  - [ ] Look for less obvious instances: Express editions on application servers, instances on non-default ports
- [ ] Confirm you have sufficient access to connect to each SQL instance
- [ ] Clarify your responsibilities for each instance with your manager, and align expectations
- [ ] Confirm environment tier (Production/Staging/Dev etc) for each instance
- [ ] Identify HA/DR configurations for each instance - Availability Groups, Failover Cluster Instances, log shipping, mirroring, replication - and note the configuration (AGs affect where backups run and how Agent jobs behave)
- [ ] Ask about change management policies, as well as how to request a change and when prior approval is needed
- [ ] Ask about maintenance windows
- [ ] Ask about compliance requirements (e.g. HIPAA, PCI, SOX) - these shape how you handle backups, security, and change requests
- [ ] Save your instances in SSMS Registered Servers/Central Management Server
- [ ] If there is no existing DBA server, request one now - provisioning can take weeks

### Install your toolkit & initial scripts

- [ ] SQL Server Management Studio (SSMS)
- [ ] VS Code (or preferred code editor)
- [ ] Adam Machanic's sp_WhoIsActive: [whoisactive.com](https://whoisactive.com)
- [ ] Ola Hallengren's Maintenance Solution: [ola.hallengren.com](https://ola.hallengren.com)
- [ ] PowerShell modules dbatools & dbachecks:
  - [dbatools.io](https://dbatools.io)
  - [dbachecks.readthedocs.io/en/latest](https://dbachecks.readthedocs.io/en/latest)
- [ ] Brent Ozar's First Responder Kit: [brentozar.com/first-aid](https://www.brentozar.com/first-aid)
- [ ] Glenn Berry's diagnostic scripts: [glennsqlperformance.com/resources](https://glennsqlperformance.com/resources)
- [ ] Plan Explorer: [solarwinds.com/free-tools/plan-explorer](https://www.solarwinds.com/free-tools/plan-explorer)
- [ ] mRemoteNG (or preferred remote connection tool)

### Confirm you can recover data

- [ ] Check each SQL instance in your inventory for existing backup processes - including backups taken outside of SQL Agent (Veeam, Commvault, Rubrik, VM snapshots, etc.)
  - [ ] Review `msdb.dbo.backupset` (`is_snapshot`, `device_type`, `is_copy_only`) before adding any backups of your own - native backups layered on top of a third-party tool can break its differential base or split the log chain
- [ ] Check recovery models against backups:
  - [ ] Databases in FULL recovery with no log backups (unbounded log growth, no point-in-time restore)
  - [ ] Databases in SIMPLE recovery whose RPO requires point-in-time restore
- [ ] Create & schedule backup jobs for any instances without backups
- [ ] Confirm that the existing backups are running and succeeding
- [ ] Take inventory of your backups, where they live, and any existing redundancies - backups stored only on the same server (or same disk) as the data need to be moved off-server
- [ ] If TDE is in use, back up the service master key, database master keys, and TDE certificates with their private keys - without them, those backups cannot be restored on any other server
- [ ] Restore the most critical database at least once, and time it - this is your first real RTO number
- [ ] Check each SQL instance for existing DBCC checks
  - [ ] Check the results, not just whether a job exists: last known good CHECKDB (`Get-DbaLastGoodCheckDb`) and `msdb.dbo.suspect_pages`
- [ ] Create & schedule DBCC jobs for any instances without integrity checks
- [ ] Set up an Operator & Database Mail, and turn on SQL Agent job failure notifications - so the backup and DBCC jobs you depend on can't fail silently
- [ ] Turn on alerts for severity 19-25 and errors 823, 824, and 825 (early warning signs of corruption)

### Identify urgent risks

- [ ] Check free space on data, log, and backup drives
- [ ] Check for tempdb, user database files, or the default backup location on the C: (OS) drive
- [ ] Check SQL Agent job owners and database owners - jobs owned by a former DBA's login will start failing as soon as that account is disabled
- [ ] Flag Evaluation edition instances (the service stops starting after 180 days) and Developer edition in production (a licensing violation)
- [ ] Flag versions that are out of support (SQL Server 2016 reached end of extended support on July 14, 2026, joining 2014 and 2012 - SQL Server 2017 is next)
- [ ] Flag any glaring security problems now, even if the fix comes later (e.g. blank/weak sa password, instance exposed to the internet, application logins with sysadmin)

### Meet your users

- [ ] Set up meetings with application/business owners, learn their goals and any current known problems
- [ ] Identify & document the most critical applications
- [ ] Identify & document any existing RPO/RTO requirements

### Document your findings and progress

- [ ] Create a spreadsheet of your SQL instance inventory, where you can organize and note any related users/applications/connection details - include at minimum server name, instance name, SQL Server version/edition/patch level, environment tier, and HA/DR configuration
- [ ] Collect backup schedules & retention details for each instance
- [ ] Create a journal to document tasks you completed as well as new findings made - update this every day

### Day 30 checkpoint

- [ ] Summarize your progress and findings from your first 30 days, highlighting any risks or recoverability gaps
- [ ] Send your 30-day summary and your SQL inventory data to your manager in a status report

---

## Days 31-60: Stabilize & Observe

### Monitoring

- [ ] Set up basic monitoring/alerting for critical SQL jobs beyond failures (e.g. backups & DBA maintenance jobs that didn't run, or ran long)
- [ ] Set up careful email rules for any excessive alert noise - route noisy alerts to a folder rather than deleting them, so nothing critical disappears unseen - and note where thresholds could be tuned in the future to prevent alert fatigue
- [ ] Run dbachecks against your instances daily
- [ ] Capture configuration baselines for each instance (sp_configure, tempdb, database settings)
- [ ] Capture performance baselines for each instance (e.g. wait stats, by logging `sp_BlitzFirst` to a table)

### Maintenance

- [ ] Keep your SQL inventory notes organized and up to date
- [ ] Check each SQL instance for existing routine database maintenance
- [ ] Create & schedule maintenance jobs for any instances without routine maintenance (index/statistics)
- [ ] Test restoring several of your production databases, and time each one
- [ ] Set up version control for your scripts

### Check in with users

- [ ] Verify that the intended/assumed backup & retention policies match the actual backup & retention processes
- [ ] Confirm existing backup procedures & schedules can meet existing RPO/RTO goals - use your timed restore tests to validate RTO
- [ ] Inform stakeholders of current RPO/RTO capability
- [ ] Check with your manager and other IT teams/coworkers for existing alerts relevant to you, note what is covered, and get access to any important alert distros (e.g. disk space, CPU, network)

### Documentation

- [ ] Create a running To-Do list (or build it into your journal) to keep future tasks and operational requests organized

### Day 60 checkpoint

- [ ] Summarize your progress and findings from days 31-60
- [ ] Send your summary and any changes to SQL inventory data to your manager in a status report

---

## Days 61-90: Optimize & Build Value

### Optimizations

- [ ] Fix any 'low-hanging fruit' performance and configuration issues - via change management, off-hours:
  - [ ] Max server memory
  - [ ] MAXDOP/CTP
  - [ ] Autogrowth settings (avoid percent growth and tiny fixed growth increments)
  - [ ] Number of tempdb data files
  - [ ] Auto-shrink and auto-close turned off
  - [ ] Page verify set to CHECKSUM
  - [ ] Instant file initialization
  - [ ] High VLF counts
- [ ] Observe monitored metrics (dbachecks, SQL alerts, performance baselines) and plan for tuning

### Expanded Monitoring

- [ ] Set up a SQL instance on your DBA server to store dbachecks results
- [ ] Schedule automated dbatools/dbachecks scripts to run against your SQL instances from your DBA server
- [ ] Review your dbachecks default Power BI report, and confirm data collection is working
- [ ] Identify decentralized or proprietary monitoring/alerting, start planning to centralize/consolidate
- [ ] Tune any benign noisy alert thresholds, or mute them

### Continued Maintenance

- [ ] Automate critical production backup testing - take backups `WITH CHECKSUM` and run `RESTORE VERIFYONLY` at minimum; the goal is a real restore plus CHECKDB (`Test-DbaLastBackup` does both)
- [ ] Identify unique or outdated backup strategies, start planning to centralize/consolidate (e.g. swap out maintenance plans for Ola Hallengren's maintenance scripts for all SQL backups)
- [ ] Build redundancy into any backup strategies that are weak (e.g. local-only backups)

### Security

- [ ] Check your SQL instances for any glaring security configurations (e.g. application end users with sysadmin, shared sa account)
- [ ] Review & note:
  - [ ] Sysadmin role membership
  - [ ] Logins belonging to people who have left
  - [ ] Orphaned users
  - [ ] Linked servers that map to sa
  - [ ] SQL authentication vs. Windows authentication usage
  - [ ] TLS/force encryption
- [ ] If there are no existing policies for granting database access, draft one, and start a conversation early about best practices and minimum viable permissions

### Ongoing Documentation

- [ ] Use your to-do list and findings in your journal to put together a roadmap for clearly identified future work, including:
  - [ ] SQL migrations/upgrades for instances out of support - or Extended Security Updates if the business needs to buy time
  - [ ] A patching cadence: Microsoft ships Cumulative Updates only (no Service Packs since SQL Server 2017), so plan how CUs will be tested and deployed

### Day 90 checkpoint

- [ ] Summarize your progress and findings from days 61-90
- [ ] Send your summary, roadmap, and any changes to SQL inventory data to your manager in a status report

---

## Days 90+

- [ ] Review with your manager what status items are useful, and set up a cadence for future status reports
- [ ] Set up an automated morning email to yourself, with consolidated data about your entire SQL inventory that is useful to you and that you will read - e.g. failed jobs, missed backups, low disk space, top 10 processes to tune - in a clean format that highlights anything needing your attention urgently
- [ ] Document escalation contacts, vendor support contacts, and on-call expectations
- [ ] Write your first runbooks, and set up DR testing
- [ ] Continue to automate repeated tasks
- [ ] Find training & development opportunities in the upcoming year, and submit training requests in advance
