# VMSP Toolkit

A ready-to-run virtual appliance for VCF 9.x administrators who look after
VCF Management Services (the VMSP Kubernetes runtime). Deploy it next to your
management components, sign in to its web UI, and you get health checks,
troubleshooting tools and VCF infrastructure checks in one place.

By Virtual Bytes · [tools.virtualbytes.io](https://tools.virtualbytes.io)

> **Independent project.** Not affiliated with, endorsed by or supported by
> Broadcom or VMware.

This repository carries **releases and documentation only**. The appliance is
distributed as an OVA; the source is not published.

## Download

The current release is **[VMSP Toolkit 3.0.1](https://github.com/virtual-bytes/vmsp-toolkit/releases/tag/v3.0.1)**.
Each release ships the OVA and a `.sha256` file. Verify before deploying:

```bash
sha256sum -c vmsp-toolkit-3.0.1.ova.sha256
```

On Windows (PowerShell):

```powershell
Get-FileHash .\vmsp-toolkit-3.0.1.ova -Algorithm SHA256
```

SHA-256 of `vmsp-toolkit-3.0.1.ova`:
`c0ab3161a8e857e8a5989cbe4ef59a264a56807e69e97dba22257ed0eef49509`

## What's new in 3.0.1

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

Full notes: [RELEASE-NOTES-3.0.1.md](RELEASE-NOTES-3.0.1.md) · history: [CHANGELOG.md](CHANGELOG.md)

## Requirements

- vSphere 9 (vCenter and ESXi 9.0 or later). The VM uses virtual hardware
  version 22, so it does not deploy on ESXi 8.
- 2 vCPU, 4 GB memory, 20 GB disk (thin provisioning is fine)
- A port group that can reach your VMSP nodes (SSH and 6443) and, for the VCF
  Infrastructure pages, SDDC Manager, vCenter, NSX and ESXi over HTTPS
- No internet access is needed to run it. Vulnerability database updates use
  the internet or an internal mirror.

## Deploy

1. In vCenter, choose **Deploy OVF Template** and select the OVA.
2. On the customization page, fill in:
   - **Hostname**
   - **IP address (CIDR)**, **default gateway** and **DNS servers**. Leave
     them blank to use DHCP.
   - **Root password**
   - **SSH public key** (optional). With a key and no password, SSH accepts
     only the key.
   - **Web UI read-only mode** and **alert webhook URL** (both optional)
3. Power it on. On first boot the appliance applies these settings and
   creates its own SSH host keys and web certificate, unique to your
   deployment.

## First sign-in

Browse to `https://<appliance>:8443` and sign in as `root` with the password
you set during deployment. The certificate is self-signed until you replace
it. Logins are throttled after repeated failures.

On the console or over SSH, type `toolkit` for the menu or `vmsp-doctor` for a
health check.

## Everything in the toolkit

- **Overview:** what needs attention, live tiles, a node map and recent
  activity.
- **Health:** unhealthy pods with one-click diagnose and logs, package
  deployments, a known-issues rulebook with fixes, and connectivity probes.
- **Topology:** services and workloads per namespace (see above).
- **Certificates:** expiry across the platform, with cert-manager reissue
  where it applies.
- **Worker rightsizing:** a live view of worker rollouts (see above).
- **Compliance:** DISA Kubernetes STIG scan with node evidence and `.ckl`
  export for STIG Viewer.
- **Vulnerabilities:** Trivy scans across the platform's images, risk
  acceptances, trends, drift and SBOM export.
- **VCF Infrastructure** (read-only): SDDC Manager, vCenter, NSX and ESXi.
  Time and DNS, certificate and password expiry, backups, platform health,
  upgrade readiness with SDDC Manager prechecks, and a hardening baseline.
- **Command-line tools** on the appliance: `kubectl`, `k9s`, `helm`, `yq`,
  `jq`, `govc`, `trivy`, plus network tools (`tcpdump`, `dig`, `traceroute`,
  `ncat`).

Everything runs on the appliance. The web UI loads nothing from the internet.

## Good to know

- The appliance holds admin kubeconfigs for your management plane. Keep its
  network access tight.
- Every page has its own link (`#/health`, `#/topology` …). Press Ctrl+K for
  the command palette.

## Disclaimer

VMSP Toolkit is provided "as is", without warranty of any kind, express or
implied. It is intended for **lab and proof-of-concept environments**.
Virtual Bytes assumes no liability for any damage, outage or data loss
arising from its use. Use on production systems is at your own risk.

Independent project. Not affiliated with or endorsed by Broadcom, VMware,
DISA, or the US Department of Defense.

## Support

This is a community tool with no official support. For VCF or VMSP problems,
open a case with Broadcom support.

## License

The appliance is distributed under the MIT License — see [LICENSE](LICENSE).
The DISA STIG text carried in the appliance's compliance pack is a US
Government work and is not covered by that grant; the third-party software
inside the appliance (Photon OS, Trivy, kubectl and others) ships under its
own licenses. Details in [NOTICE](NOTICE).
