# VirtualBytes VMSP Toolkit

A Photon OS appliance for VMware Cloud Foundation 9.x administrators. Deploy
it as an OVA; it troubleshoots the VCF management platform (VMSP), evaluates
it against the DISA Kubernetes STIG from real node evidence, scans the running
container images, and produces a posture report that states what it could not
check.

This repository carries **releases and documentation only**. The appliance is
distributed as an OVA; the source is not published.

## Download

Releases are on the [Releases page](https://github.com/virtual-bytes/vmsp-toolkit/releases).
Each release ships the OVA and a `.sha256` file. Verify before deploying:

```
sha256sum -c vmsp-toolkit-<version>-assurance.ova.sha256
```

## Deploy

vCenter → *Deploy OVF Template* → select the OVA. The wizard prompts for
hostname, IP settings, root password, and SSH key. The wizard's
Product/Version fields, the OVA filename, and the appliance's own login page
all report the same version and edition — they derive from the same source
line, so if they ever disagree, the build that produced the OVA would have
refused.

After first boot, connect a kubeconfig from the toolkit menu.

## Editions

| Edition | Capabilities |
|---|---|
| **Assurance** | Operations toolkit + STIG evaluation with node evidence, fleet image scanning, node-store scanning, posture report |

## What it will not do unless you ask

- No vulnerability database ships in the OVA. It is downloaded on request and its age is shown in every view that depends on it.
- Nothing scans on a schedule until the timer is switched on from the UI.
- Nothing touches a node without per-session consent; the exact commands are shown first.
- Nothing is created in the cluster; node access is over SSH, read-only.
- No outbound calls beyond the database fetch you initiate.

## Disclaimer

VMSP Toolkit is provided as is, without warranty of any kind, and is intended
for lab and proof-of-concept environments. Production use is at the operator's
own risk.

Independent project. Not affiliated with or endorsed by Broadcom, VMware,
DISA, or the US Department of Defense.

## License

The appliance is distributed under the MIT License — see [LICENSE](LICENSE).
The DISA STIG text carried in the appliance's compliance pack is a US
Government work and is not covered by that grant; the third-party software
inside the appliance (Photon OS, Trivy, kubectl) ships under its own licenses.
Details in [NOTICE](NOTICE).
