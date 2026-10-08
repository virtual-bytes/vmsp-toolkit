# VMSP Toolkit 3.0.1

The first public release of the 3.0 line. The product is now simply
**VMSP Toolkit**; the "Assurance Edition" name is gone.

**Download:** [v3.0.1 release](https://github.com/virtual-bytes/vmsp-toolkit/releases/tag/v3.0.1)
· SHA-256 `c0ab3161a8e857e8a5989cbe4ef59a264a56807e69e97dba22257ed0eef49509`

### Container topology, redesigned

The topology page now shows how traffic reaches your workloads, one
namespace at a time:

- **One card per namespace.** Services sit on the left, the workloads they
  select on the right, joined by drawn links.
- **Problems come first.** Degraded workloads are red, and their namespace
  moves to the top.
- **Trace a path.** Hover any service or workload to highlight its links.
- **Search** across every namespace.
- **No clutter.** Healthy rows without links fold into one line per
  namespace, and CronJob runs collapse into one row per CronJob.
- **Readable at any size.** Labels never overlap; long names end in … with
  the full name on hover.
- **Broken services stay visible.** A service whose selector matches no pod
  is marked "no pods".

### Worker rollouts, live

The worker rightsizing page follows a worker rollout as it happens:

- the current stage and a plain sentence on what is happening right now;
- old and new workers side by side as each one is replaced;
- drains, and the pods holding them up behind a PodDisruptionBudget;
- the time left before Cluster API's drain timeout;
- the command's output, the rollout timeline and progress in the browser tab.

A blocked pod can be released only while Cluster API is draining its node.
A release never cordons a node or changes a PodDisruptionBudget. A rollout
that fails, or changes nothing, is reported as such.

### VCF Infrastructure

- **SDDC Manager is the source of truth for passwords.** Managed accounts
  show SDDC Manager's expiry, status and rotation schedule. **Re-verify with
  SDDC Manager** refreshes every expiry on demand.
- **NSX password expiry fixed.** NSX root no longer shows as expired.
  Accounts VCF doesn't manage are shown for information only.
- **SoS health check results.** GREEN, YELLOW and RED results appear, each
  with its area, check and message. Run it through the SDDC Manager API with
  your SSO login, no SSH needed, or load the newest finished run, including
  VCF Operations' daily collection.
- **Clearer VCF 9 sign-in.** "User is not authorized" from SDDC Manager is
  explained: a wrong password or a missing SDDC Manager role. An optional
  separate SDDC Manager login is available. The toolkit never retries enough
  to trigger SDDC Manager's 24-hour IP lockout.
- **vSAN datastores and ESXi lockdown mode** are read correctly again.

### Health and troubleshooting

- **Known issues show the lines that matched**, not the first few lines of
  output. A check that times out reads Unknown, never clear.
- **New rule** for a PackageDeployment that has failed.
- **Diagnose explains a pod that has already gone** (for example a finished
  CronJob run) and shows the CronJob's recent runs and logs.

### A cleaner interface

Both light and dark themes were redrawn: neutral colours, flat panels,
sentence-case labels and one accent colour. The product is now simply
**VMSP Toolkit**.

## Requirements

- vSphere 9 (vCenter and ESXi 9.0 or later). The VM uses virtual hardware
  version 22, so it does not deploy on ESXi 8.
- 2 vCPU, 4 GB memory, 20 GB disk (thin provisioning is fine)
- A port group that can reach your VMSP nodes (SSH and 6443) and, for the VCF
  Infrastructure pages, SDDC Manager, vCenter, NSX and ESXi over HTTPS
- No internet access is needed to run it. Vulnerability database updates use
  the internet or an internal mirror.

Lab and proof-of-concept use only. Not affiliated with or endorsed by
Broadcom or VMware.
