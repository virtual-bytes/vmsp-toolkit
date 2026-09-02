# VMSP Toolkit 2.0.0-rc.1 — Assurance Edition

The 2.x line adds a security-assurance layer to the VMSP Toolkit appliance:
STIG compliance evaluation with node-level evidence collection, fleet
container-vulnerability scanning, and a one-click posture report — alongside
everything the 1.x operations toolkit did. It is a major version because the
privilege model changed: where 1.x only read a kubeconfig, 2.x can (with
explicit per-session consent) SSH to VMSP nodes, and can be told to scan the
platform registry on a schedule. Every such capability is opt-in, printed
before it runs, and reviewable.

**This is a release candidate.** Promotion to 2.0.0 happens when the built
OVA passes verification. Lab/PoC use only, per the disclaimer.

## Highlights

### STIG compliance with node evidence (new)
- Scans the connected VMSP instance against the DISA Kubernetes STIG (V2R6,
  92 requirements) and produces a STIG Viewer `.ckl`.
- API-scope checks run agentlessly. Node-local checks (kubelet config,
  manifest flags, file modes) are evaluated from **evidence bundles**
  collected over SSH as `vmware-system-user` — a fixed, printable
  cat/stat/ps plan shown before you consent; the password is per-session and
  never stored; kubeconfig contents and private keys are never read, and a
  redaction guard keeps key material out of bundles.
- With node evidence, up to 25 of the 92 requirements evaluate automatically
  today (14 node-local CAT I checks plus the API-scope set). Everything the
  appliance cannot answer is reported **Not_Reviewed — never guessed**.
- Node evidence is embedded in the `.ckl` FINDING_DETAILS for auditability.

### Container vulnerability scanning (new)
- Inventories every image running in the cluster and scans the platform
  registry with Trivy, entirely on the appliance. **No vulnerability database
  ships in the OVA** — one baked in would be stale on arrival; the appliance
  downloads it on demand and reports its age in every view.
- **Scan fleet** from the UI with live progress; CLI and button share one
  code path. Trend tracking across runs, base-layer drift, exposed-secret
  detection, risk-acceptance workflow with expiry dates.
- **Not-in-registry detection:** images that run in the cluster but are
  absent from the registry they claim to come from are flagged as
  supply-chain findings, not scan errors — such an image cannot be re-pulled
  and will fail on a node rebuild.
- **Node-store scanning (new):** those images can still be scanned — exported
  from a node's containerd store over SSH (no cluster mutation, byte-count
  verified, temporary files removed on every exit path) and scanned with the
  appliance's database. The remote command plan is shown before consent.
- Optional **weekly scheduled scan**: ships OFF; one reversible toggle in the
  UI. The appliance never pulls images unasked.

### Posture report (new)
- One click produces a self-contained HTML report (print-to-PDF): STIG state,
  open findings, node-evidence list, severity totals, top open CVEs, drift,
  secrets, coverage, trend, and a provenance block (cluster, benchmark, scan
  run, scanner and DB versions, appliance version and edition). Absent data
  is reported as absent, never implied clean.

### Multi-cluster awareness (new)
- Scan runs, node evidence, and checklists are strictly per-cluster, keyed by
  kubeconfig. Switching clusters clears and reloads every cluster-scoped view.

### Operations (improved)
- Live topology gains focus mode for large fleets (unlinked healthy workloads
  collapse to per-namespace tiles; nothing degraded or linked is ever hidden).
- Coverage is stated in words: "N running images: X via registry, Y via node
  store, Z unscanned" — image coverage and STIG requirement counts can no
  longer be confused.
- Light theme fully re-tuned: every status colour now meets WCAG AA contrast
  in both themes (several light-mode severity badges were previously near
  1.5:1 against white).

### Build & identity
- The appliance version and edition live in exactly one place
  (`scripts/install-tools.sh` → `/etc/toolkit/version`). The build derives
  the OVA filename, the OVF ProductSection (what the vCenter deploy wizard
  shows), and the wizard's VM-name prefill from it, and **fails** if anything
  disagrees or the payload tree is stale. `/etc/toolkit/build-info` records
  the build date, open-vm-tools version, kernel, and Photon release.

## Upgrading from 1.x
- **Recommended: deploy the 2.0.0-rc.1 OVA fresh.** Kubeconfigs can be
  re-fetched via the toolkit menu.
- In-place upgrades of a 1.x appliance are possible but manual (new Python
  modules, a service restart, and a scan-directory migration for
  multi-cluster: `cd /var/lib/toolkit/trivy/scans && mkdir -p <cluster> &&
  mv 20*Z <cluster>/`). Pre-existing runs remain readable either way,
  labelled as predating multi-cluster and excluded from per-cluster trends.

## Known limitations
- 66 STIG requirements (largely CAT II) are not yet automated and report
  Not_Reviewed; automation is the next increment.
- Node evidence collection and node-store scanning require SSH reachability
  to the VMSP nodes and the `vmware-system-user` password per session.
- The scheduled-scan timer is off by default by design.
- The OVA is unsigned ("No certificate present" in the deploy wizard is
  expected).

## Verification
After download:
```
sha256sum -c vmsp-toolkit-2.0.0-rc.1-assurance.ova.sha256
```
After deploy: the login page and footer should read
**2.0.0-rc.1 · Assurance Edition**, matching the OVA filename and the deploy
wizard — all three derive from the same source line, so agreement here
verifies the artifact end to end.

## Attribution
STIG requirement identifiers and guidance text derive from the DISA
Kubernetes Security Technical Implementation Guide (V2R6), published by the
US Defense Information Systems Agency at public.cyber.mil. This project is
not affiliated with or endorsed by DISA, DoD, Broadcom, or VMware.

## Disclaimer
VMSP Toolkit is provided "as is", without warranty of any kind. Intended for
lab and proof-of-concept environments; production use at the operator's own
risk.
