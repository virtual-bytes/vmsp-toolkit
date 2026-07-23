# Changelog

All notable changes to the VMSP Toolkit Appliance. Each release ships as a
new OVA on the [Releases](../../releases) page with a SHA256 checksum.

## v1.0.0 — first public release

**Appliance**
- Photon OS 5 base, fully patched at build; inbound firewall allows only
  SSH (22) and the web UI (8443)
- Deploy-time customization via OVF properties: hostname, static IP/CIDR or
  DHCP, gateway, DNS, root password, SSH public key, read-only mode,
  webhook URL
- Forgiving static-IP handling: a bare IP assumes /24, invalid input falls
  back to DHCP instead of leaving the appliance unreachable
- Unique self-signed TLS certificate minted at first boot (replaceable)

**Web UI**
- Session login with disclaimer acceptance; optional server-enforced
  read-only mode
- Health cards, namespace-grouped topology map (pan/zoom), package
  deployment table with pod drill-down
- Known-issues rules engine with confirmation-gated one-click fixes,
  extensible via YAML rules
- Fleet-wide TLS certificate sweep with cert-manager renewal
  (VCF-managed certs are flagged for the official workflow)
- Velero backup status, warning-events viewer, connectivity matrix,
  embedded web terminal
- In-app themed confirmation dialogs for all mutating actions

**CLI**
- `vmsp-doctor`: health sweep aligned to VCF 9.1 package-deployment phase
  vocabulary, namespace log dump, support bundle collection, fluentd
  remediation (VCFMS-HEALTH-002 / KB 438093)
- `toolkit` menu: guarded kubeconfig fetch with input validation and live
  API verification, worker-node detection with control-plane hint,
  fleet-wide cert expiry sweep; shares the active instance with the web UI
- kubectl, k9s, helm, yq, govc, cmctl preinstalled

**Automation**
- Hourly health watch with webhook alerts (Slack / Teams / generic JSON)
  on failure-count change
