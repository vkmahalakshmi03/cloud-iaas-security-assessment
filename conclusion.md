# Conclusion

---

## What This Assessment Found

Neither Azure nor GCP defaults to a secure configuration. Both require deliberate post-provisioning hardening before running any production workload.

The risk profiles are different by platform:

**GCP's biggest problem is IAM.** The default Editor service account is attached to every VM automatically. The metadata endpoint exploit path is well-documented and requires no special tooling — just HTTP access from inside the VM. Any environment running production workloads without addressing GCP-001 is one VM compromise away from losing the entire project.

**Azure's biggest problem is data exposure and visibility.** Public blob access on by default, combined with no storage access logging, means data can be exfiltrated with zero trace. NSG flow logs are off, so there's also no network-layer visibility to detect anything else.

**Both platforms fail at logging.** 7 of 12 findings are detection gaps. An attacker operating in either default environment — from initial access through exfiltration — generates almost no audit trail. This is the most consistent finding across the assessment.

---

## Takeaways

**Default ≠ secure.** Both platforms provide the tools to secure an environment. None of those tools are on by default. Organizations that don't actively configure security after provisioning are running blind.

**Logging gaps are the most consequential finding class.** A compromised environment with no logs cannot be properly investigated. Dwell time is unknown. Scope is unknown. The fix is not complex — enabling flow logs and audit logs is a configuration change, not an architectural one. It should be day-one.

**GCP's default SA binding is the most exploitable single misconfiguration.** The path from VM compromise to project-wide access is straightforward and well-documented. It takes less than a minute to exploit in a default environment.

**IAM defaults differ significantly between the two platforms.** Azure requires explicit RBAC assignments — GCP auto-attaches a broadly privileged service account. These differences matter when teams are operating in multi-cloud environments and making assumptions based on one platform's behavior.

---

## What I'd Extend If Continuing This Assessment

- Run Prowler (supports both Azure and GCP) to surface additional findings at scale across IAM and logging configurations that manual review may miss
- Document a before/after comparison showing Defender for Cloud and SCC scores after applying the hardening checklists
- Expand scope to PaaS defaults — managed databases, Cloud Run, Azure App Service have their own misconfiguration patterns and are increasingly the primary attack surface
