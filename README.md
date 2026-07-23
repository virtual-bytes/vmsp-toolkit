# VMSP Toolkit Appliance

A day-2 troubleshooting appliance for the **VMware Management System Platform**
(VMSP / VCF Management Services) in **VCF 9.x**. Deploy the OVA next to your
management plane, point it at a VMSP control node, and get a live dashboard
and CLI toolkit for the platform's Kubernetes runtime.

**No building, no dependencies, no setup** — the appliance ships as a
ready-to-deploy OVA.

## Quick start

1. Download the OVA + `.sha256` from [Releases](../../releases) and verify:
   `sha256sum -c vmsp-admin-toolkit-<version>.ova.sha256`
2. vCenter → **Deploy OVF Template**, fill in the wizard (IP or DHCP,
   password, done).
3. Browse to `https://<appliance>:8443` and sign in.
4. **+ Connect instance** → your VMSP control node → dashboard goes live.

Full details below.

Part of the VirtualBytes tool family ([tools.virtualbytes.io](https://tools.virtualbytes.io)).
Independent project — **not affiliated with Broadcom or VMware**. Complementary
to William Lam's VIS appliance: VIS provides day-0/1 prerequisite services
(DNS/NTP/DHCP etc.), while this appliance covers day-2 diagnostics of the VMSP
runtime itself.

## Disclaimer

VMSP Toolkit is provided "as is", without warranty of any kind, express or
implied. It is intended for lab and proof-of-concept environments. VirtualBytes
assumes no liability for any damage, outage, or data loss arising from its use.
Use on production systems is at the operator's own risk; administrators and
engineers who do so confirm they understand and accept those risks.

## What's inside

The appliance is a small Photon OS 5 VM exposing exactly two ports: SSH (22)
and the web UI (8443, TLS).

**Web UI** — `https://<appliance>:8443`
- Live platform health cards: nodes, package deployments, pods with issues,
  soonest certificate expiry
- Namespace-grouped topology map of the VMSP runtime (services → workloads,
  pan/zoom)
- Package deployment table with per-namespace pod drill-down
- Known-issues rules engine matched against live cluster state, with
  confirmation-gated one-click fixes (e.g. the post-upgrade fluentd buffer
  issue, VCFMS-HEALTH-002 / KB 438093) — extensible by dropping YAML rules
  into `/opt/toolkit/rules/`
- Fleet-wide TLS certificate sweep across all namespaces, with renewal for
  cert-manager-managed certificates (certs managed by VCF itself are flagged
  for the official SDDC Manager / VCF Operations workflow instead)
- Velero backup status, warning-events viewer, node connectivity matrix
- Embedded web terminal
- Read-only mode (set at deploy time or in `/etc/toolkit/webui.env`) that
  blocks all mutating actions server-side

**CLI** — over SSH
- `vmsp-doctor` — full health sweep (nodes, package deployments, pods,
  StatefulSets/Deployments, PVCs, fleet-wide cert expiry, warning events,
  disk); also `--quick`, `--logs <namespace>`, `--bundle` (support bundle
  tar.gz), `--fix-fluentd`
- `toolkit` — interactive menu: fetch kubeconfigs from VMSP nodes, select the
  active instance, run doctor, collect bundles, launch k9s, watch pods,
  fleet-wide cert sweep
- kubectl, k9s, helm, yq, govc, cmctl preinstalled

**Automation**
- Hourly health watch (`vmsp-doctor --quick`) with webhook alerts
  (Slack / Teams / generic JSON) whenever the failure count changes

## Download & verify

Get the OVA and its checksum from [Releases](../../releases), then:

```bash
sha256sum -c vmsp-admin-toolkit-<version>.ova.sha256
```

## Deploy the OVA

1. vCenter → **Deploy OVF Template** → select the OVA and follow the wizard.
2. On the **Customize template** page, fill in:
   - **Hostname** — FQDN for the appliance
   - **IP Address (CIDR)** — e.g. `10.0.0.50/24`. A bare IP assumes `/24`;
     leave empty for DHCP. Invalid input falls back to DHCP rather than
     leaving the appliance unreachable.
   - **Default Gateway** and **DNS Servers** (comma-separated)
   - **Root password** — leave empty to force an interactive change at first
     login
   - **SSH public key** — optional, appended to root's authorized_keys
   - **Web UI read-only** — deploy the dashboard with all fixes disabled
   - **Webhook URL** — optional, enables the hourly health alerts
3. Power on. First boot regenerates SSH host keys, applies your settings, and
   mints a unique self-signed TLS certificate. SSH and the web UI are up
   within ~2 minutes.

Placement and reachability: put the appliance on a network that can reach the
VMSP control nodes on **22** (one-time kubeconfig fetch) and the cluster API
endpoint — typically the kube-vip VIP — on **6443**. Note the VIP serves only
6443; port 443 being closed there is normal.

## First login

- **Web UI:** browse to `https://<appliance>:8443`, accept the self-signed
  certificate, and sign in with a local appliance account (root works).
  Signing in doubles as accepting the disclaimer.
- **SSH:** log in as root and type `toolkit` for the menu, or run
  `vmsp-doctor` directly.

## Connect to your VMSP runtime

Web UI → **+ Connect instance** → enter a VMSP **control-plane** node IP/FQDN
and the `vmware-system-user` password. The appliance fetches
`/etc/kubernetes/admin.conf` over SSH once, verifies it against the API
server, and stores it locally with 0600 permissions. The password is used for
that single fetch and never stored. Pointing it at a worker node gets you a
hint with the control plane's address instead of a broken config. The same
flow exists in the CLI (`toolkit` → option 1). Multiple instances can be
fetched and switched between; the web UI, CLI, and health watch share one
"active" instance.

## Typical workflow

1. Connect an instance (above) — the dashboard populates immediately.
2. Scan the health cards and topology; drill into any package deployment's
   pods from the table.
3. Run the known-issues scan; apply one-click fixes where offered.
4. Check the certificate sweep for anything expiring; renew
   cert-manager-managed certs from the UI.
5. For a Broadcom support case: SSH in, `toolkit` → option 5 for a support
   bundle.

## Updating / replacing the TLS certificate

Each release is a fresh OVA — deploy the new version and connect your
instances again (kubeconfig fetch takes seconds). To use your own web
certificate, replace the files in `/etc/toolkit/tls/` and reload nginx.

## Security notes

- The appliance holds admin kubeconfigs for your management plane. Treat it
  like a management jump box: tight network scope, and clear
  `/opt/toolkit/kubeconfigs` after incidents.
- Only 22 and 8443 are open inbound; everything else is dropped.
- Web sessions are HttpOnly/Secure cookies with a 12-hour TTL, held in memory
  (a service restart signs everyone out).
- Read-only mode is enforced in the backend, not just hidden in the UI.

## Troubleshooting the appliance itself

```bash
systemctl status vmsp-webui nginx vmsp-ttyd vmsp-watch.timer
journalctl -u vmsp-webui            # dashboard backend
journalctl -u toolkit-firstboot     # deploy-time customization
```

## Issues & feedback

Use the [issue tracker](../../issues). Please include the appliance version,
VCF version, and relevant journal output from above.
