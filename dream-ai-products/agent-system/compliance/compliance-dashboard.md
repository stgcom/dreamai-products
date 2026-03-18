# Dream AI — Enterprise Compliance Dashboard

> For B2B Enterprise Sales ($1,497+ deployments)
> Last updated: 2026-03-18

---

## 1. Compliance Requirements

### SOC 2 Type II Readiness Checklist

| Control Area | Status | Owner | Notes |
|---|---|---|---|
| Security | 🟡 In Progress | — | Access controls, encryption verified |
| Availability | 🟡 In Progress | — | 99.9% uptime target, monitoring active |
| Processing Integrity | 🔴 Not Started | — | Data validation controls needed |
| Confidentiality | 🟡 In Progress | — | Encryption at rest + transit configured |
| Privacy | 🔴 Not Started | — | Privacy policy, consent flows pending |

**Timeline:** 6-month readiness track. Target audit Q3 2026.

### Data Encryption

| Layer | Method | Standard | Status |
|---|---|---|---|
| At Rest | AES-256 | FIPS 140-2 | ✅ Configured |
| In Transit | TLS 1.3 | — | ✅ Enforced |
| API Keys | Encrypted vault | — | ✅ Stored encrypted |
| Backups | AES-256 | — | ✅ Encrypted |

### Access Logging & Audit Trails

- **Scope:** All API endpoints, database queries, admin actions, file access
- **Retention:** 365 days (exceeds 90-day minimum)
- **Format:** Structured JSON logs with timestamps, user IDs, IP addresses
- **Storage:** Immutable log store (append-only)
- **Alerting:** Anomaly detection on access patterns

### Data Retention Policies

| Data Type | Default Retention | Archive After | Delete After |
|---|---|---|---|
| Customer PII | Active account + 90d | — | 90 days post-closure |
| Transaction records | 7 years | 1 year | Per legal req |
| Access logs | 365 days | 90 days | 365 days |
| Chat transcripts | 180 days | 30 days | 180 days |
| Analytics data | Indefinite | Anonymize at 1yr | — |

### Right to Deletion (GDPR-lite)

1. **Request intake:** Customer submits deletion request via support portal
2. **Verification:** Confirm identity, verify account ownership
3. **Scope check:** Identify all data stores (DB, backups, logs, 3rd-party)
4. **Execution:** Delete or anonymize within 30 days
5. **Confirmation:** Send written confirmation to customer
6. **Exceptions:** Legal holds, financial records (retain as required)

### Business Associate Agreement (HIPAA — MedSpa Clients)

- **Trigger:** Any client handling PHI (patient records, treatment history, health data)
- **BAA scope:** Cloud hosting, AI processing, data storage
- **Key terms:**
  - Minimum necessary standard
  - Breach notification within 60 days
  - Subcontractor flow-down provisions
  - Right to audit
  - Data return/destruction on termination
- **Execution:** BAA signed before deployment, renewed annually
- **Status:** Template drafted, legal review pending

---

## 2. Security Controls

### API Key Rotation Schedule

| Key Type | Rotation Period | Auto-Rotate | Alert Before Expiry |
|---|---|---|---|
| Production API keys | 90 days | ✅ Yes | 14 days |
| Service account keys | 90 days | ✅ Yes | 14 days |
| Third-party integrations | Per vendor policy | ❌ Manual | 30 days |
| Database credentials | 180 days | ✅ Yes | 30 days |

### Access Control Matrix

| Role | User Data | API Config | Billing | Audit Logs | Admin Panel |
|---|---|---|---|---|---|
| Customer | R (own) | — | R (own) | — | — |
| Support | R (assigned) | — | R (assigned) | R | — |
| Admin | RW | R | RW | R | R |
| Security | R | R | — | RW | R |
| Super Admin | RW | RW | RW | RW | RW |

*R = Read, W = Write, — = No Access*

### Incident Response Playbook

**Severity Levels:**

| Level | Description | Response Time | Example |
|---|---|---|---|
| P1 — Critical | Data breach, service down | 15 min | Customer data exposed |
| P2 — High | Security vulnerability active | 1 hour | Unauth API access detected |
| P3 — Medium | Potential issue, contained | 4 hours | Suspicious login pattern |
| P4 — Low | Minor issue, no impact | 24 hours | Log anomaly |

**Response Steps:**
1. **Detect** — Automated alerting + manual report
2. **Assess** — Classify severity, identify scope
3. **Contain** — Isolate affected systems, revoke access
4. **Remediate** — Fix root cause, patch vulnerability
5. **Notify** — Customer notification if PII affected (within 72h)
6. **Document** — Post-incident report within 5 business days
7. **Review** — Lessons learned, update controls

### Backup Procedures

| System | Frequency | Method | Retention | Test |
|---|---|---|---|---|
| Primary database | Continuous | WAL replication | 30 days | Weekly restore |
| Application data | Daily | Incremental | 90 days | Monthly |
| Configuration | On change | Git + snapshot | Indefinite | Weekly |
| Logs | Real-time | Streaming | 365 days | — |

### Disaster Recovery

| Metric | Target | Actual |
|---|---|---|
| **RPO** (Recovery Point Objective) | 24 hours | ~1 hour (WAL replication) |
| **RTO** (Recovery Time Objective) | 4 hours | ~2 hours (automated failover) |

**DR Strategy:**
- Primary: Zeabur managed hosting (US-West)
- Replica: Automated backup in separate AZ
- Failover: Manual trigger with 2-hour target
- DR Test: Quarterly tabletop exercise
- Communication: Status page + email alerts

---

## 3. Dashboard Requirements

### Real-Time Security Posture Score

**Scoring Model (0–100):**

| Factor | Weight | Score Source |
|---|---|---|
| Encryption status | 20% | System check |
| API key age compliance | 15% | Key management audit |
| Access control coverage | 15% | Role audit |
| Vulnerability count | 20% | Security scanner |
| Backup freshness | 15% | Backup monitoring |
| Audit trail completeness | 15% | Log coverage check |

**Target:** ≥ 85/100 for enterprise certification

### Open Vulnerabilities Count

| Severity | Current | Target | SLA |
|---|---|---|---|
| Critical | 0 | 0 | Fix within 24h |
| High | 0 | < 3 | Fix within 7 days |
| Medium | — | < 10 | Fix within 30 days |
| Low | — | < 25 | Fix within 90 days |

### Compliance Checklist Progress

| Category | Items | Complete | % |
|---|---|---|---|
| SOC 2 Security | 12 | 7 | 58% |
| SOC 2 Availability | 8 | 5 | 63% |
| SOC 2 Confidentiality | 6 | 4 | 67% |
| Data Protection | 10 | 6 | 60% |
| Incident Response | 5 | 4 | 80% |
| **Overall** | **41** | **26** | **63%** |

### Audit Dates

| Audit Type | Last Completed | Next Scheduled | Status |
|---|---|---|---|
| Internal security review | 2026-02-15 | 2026-03-15 | ⏰ Due |
| Access control audit | 2026-02-01 | 2026-03-01 | ⏰ Due |
| Backup restoration test | 2026-02-20 | 2026-03-20 | ✅ On track |
| Full compliance review | 2025-12-01 | 2026-03-01 | ⏰ Due |
| SOC 2 Type II | — | Q3 2026 | 🔴 Not started |

---

## 4. N8N Compliance Monitoring Workflows

### Weekly: API Key Age Check

**Schedule:** Every Monday, 09:00 UTC

**Workflow:**
1. Query key management system for all active keys
2. Calculate age of each key (created_at → now)
3. Flag keys > 60 days old (30-day warning)
4. Flag keys > 90 days old (rotation required)
5. Send digest to security channel:
   - Keys OK: count
   - Keys expiring soon: count + list
   - Keys expired: count + list (urgent)
6. Create Jira/Notion ticket for expired keys

### Monthly: Access Log Audit

**Schedule:** 1st of each month, 10:00 UTC

**Workflow:**
1. Export access logs for previous month
2. Analyze patterns:
   - Login anomalies (unusual hours, locations)
   - Failed authentication attempts (> 5)
   - Privilege escalation events
   - Data export events
3. Generate audit report with findings
4. Flag anomalies for review
5. Store report in compliance folder
6. Send summary to security@dreamai

### Quarterly: Full Compliance Review

**Schedule:** 1st of Jan/Apr/Jul/Oct, 11:00 UTC

**Workflow:**
1. Run full security posture score calculation
2. Check all encryption configurations
3. Verify backup freshness + run restoration test
4. Review access control matrix (remove stale roles)
5. Check API key rotation compliance
6. Review incident response readiness (run drill)
7. Update compliance checklist progress
8. Generate quarterly compliance report
9. Schedule next quarter's audit dates
10. Present findings to leadership

### N8N Implementation Notes

- Use **Cron Trigger** for scheduling
- **HTTP Request** nodes for API calls to monitoring systems
- **Postgres** nodes for database queries
- **Slack/Telegram** nodes for notifications
- **Code** nodes for scoring calculations
- Store results in compliance tracking table

---

## Appendix: Enterprise Sales Checklist

Before deploying to an enterprise client ($1,497+):

- [ ] Security posture score ≥ 85
- [ ] No critical or high open vulnerabilities
- [ ] BAA signed (if MedSpa/healthcare)
- [ ] Data retention policy acknowledged by client
- [ ] SLA terms agreed (uptime, response times)
- [ ] Access controls configured for client's team
- [ ] Integration credentials rotated (fresh keys)
- [ ] Backup procedures confirmed
- [ ] Incident response contacts exchanged
- [ ] Compliance dashboard access provisioned
