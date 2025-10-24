# Information-Stealer

---

## Important — Safety & Legal Notice

I can’t help create or provide materials for tools intended to steal information or otherwise perform unauthorized access. Building or distributing an "information stealer" is harmful and illegal in many jurisdictions.

However, I *can* help with **defensive, educational**, and **authorized** projects that teach the same concepts safely — for example, a lab that simulates data-exfiltration techniques at a high level so defenders can learn detection, prevention, and incident response. The rest of this README is a complete, safe alternative you can use for classroom exercises or red-team/blue-team training in a controlled environment.

---

## Project Overview

This repository contains materials for a **Data Leak Simulation & Detection Lab**. The goal is to let students and security teams practice detecting and responding to simulated data exfiltration attempts in a legal, controlled environment. The project focuses on detection engineering, logging, SIEM rules, network monitoring, and incident response — not on developing malware or exfiltration tools.

Use this lab only on isolated networks, virtual machines, or containerized environments you control. Obtain written authorization for any testing on shared infrastructure.

---

## Learning Objectives

Participants will learn to:

* Understand common data exfiltration patterns (beacons, large file transfers, covert channels) at a conceptual level.
* Instrument hosts and services to generate relevant logs for detection (syslog, OS audit logs, application logs).
* Create detection rules and alerts in a SIEM (example rules for suspicious outbound connections, anomalous file access patterns, and unusual DNS queries).
* Develop basic playbooks for incident response and containment.
* Use network monitoring tools (e.g., Zeek, Suricata, tcpdump) to capture and analyze suspicious traffic.
* Build benign, scripted “simulations” that generate realistic-but-safe behaviors for training (e.g., scripted file copy + upload to a lab-controlled FTP/S3 endpoint), with strict safeguards and rate limits.

---

## Components

* `lab-scripts/` — contains **safe** simulation scripts that *only* generate observable behaviors (file read events, outbound HTTP POSTs to a lab server) using non-sensitive test data. Scripts are intentionally basic and include explicit safeguards (rate limits, size limits, and require an explicit lab mode flag).
* `siem-rules/` — example detection rules for common SIEM platforms (pseudocode and mapping to Splunk/Kibana/Elastic SIEM concepts).
* `network-monitoring/` — Zeek/Suricata example configs and recommended pipelines for PCAP capture and analysis.
* `playbooks/` — incident response playbooks for detection, containment, eradication, and lessons learned.
* `lab-setup/` — instructions and `docker-compose` to spin up an isolated lab: ELK stack (Elastic / Kibana), Zeek, a benign file-receiver service (lab-controlled HTTP/SFTP endpoint), and a target VM.
* `test-data/` — synthetic, non-sensitive files used by simulations (e.g., lorem ipsum documents, dummy CSVs); no real PII or secrets.

---

## Requirements

* Docker & Docker Compose (for the isolated lab)
* Python 3.8+ (for simulation scripts and helpers)
* Optional: Elastic Stack / Splunk / Graylog (can be run via Docker images in `docker-compose`)

---

## Safe Simulation Scripts (High-level)

All provided scripts are intentionally non-malicious and include safety checks. They are meant to create observable signals (file access, outbound connections) for defenders to detect.

Key safety features in scripts:

* Require `--lab-mode` flag to run (prevents accidental execution)
* Use only synthetic test data from `test-data/`
* Enforce rate limits and maximum upload size
* Target only lab-controlled endpoints defined in `lab-config.yml`

Example behaviors implemented by scripts (safe):

* `simulate_file_read.py` — opens a set of test files, reads them to generate file access logs, and timestamps activity.
* `simulate_upload.py` — uploads small, synthetic files to a lab-controlled HTTP/SFTP receiver using short, randomized intervals.
* `simulate_dns_beacon.py` — generates a controlled pattern of DNS queries to a lab DNS server to mimic beaconing (no external DNS traffic).

These scripts are high-level and well-commented; they avoid techniques that would enable real-world data theft.

---

## Detection Rules (Examples)

Provided as pseudocode and mappings to SIEM rule languages. Examples include:

* **Unusual outbound volume:** Alert on a host sending a sudden spike in outbound bytes to an external IP outside a whitelist.
* **Anomalous file access + network:** Alert when a non-privileged process reads many files in quick succession followed by outbound network connections.
* **DNS tunneling indicators:** Excessive DNS TXT queries or unusually long subdomain lengths to external DNS servers (lab-only patterns).
* **Use of uncommon protocols:** Alert on outbound FTP/SCP from endpoints that don’t normally use them.

Each rule includes recommended thresholds, rationale, and suggested enrichment fields (tags, geo-lookup, asset owner).

---

## Lab Setup (Quick Start)

1. Clone repository and enter directory:

```bash
git clone https://github.com/<your-username>/data-leak-sim-lab.git
cd data-leak-sim-lab
```

2. Review `lab-config.yml` and set all endpoints to `localhost` or the isolated Docker network. **Do not** point to external services.

3. Start the lab (Docker Compose):

```bash
docker-compose up --build
```

This brings up Elastic/Kibana (or your chosen SIEM), Zeek, and a benign receiver service. Wait for services to become healthy.

4. Run a safe simulation script (requires `--lab-mode`):

```bash
python3 lab-scripts/simulate_upload.py --lab-mode --endpoint http://lab-receiver:8080 --files test-data/sample1.csv --rate 1
```

5. Observe generated logs in Kibana / SIEM and test detection rules.

---

## Incident Response Playbook (Summary)

1. **Detect & Triage:** Confirm alert, gather host and process context, collect logs and PCAPs.
2. **Contain:** Isolate the host from external network access while preserving volatile evidence.
3. **Eradicate:** Remove unauthorized tools or scripts, rotate credentials if necessary.
4. **Recover:** Restore services from trusted backups and monitor for reoccurrence.
5. **Lessons Learned:** Update detection rules, harden controls, and, if appropriate, adjust training exercises.

---

## Ethical Guidance

* Only run simulations in lab environments or where you have explicit written authorization.
* Use synthetic, non-sensitive test data only. Never include real PII, credentials, or secrets in the lab.
* Treat the lab like a safe learning space: document experiments and ensure reproducibility.

---

## Extending the Lab

Ideas for classroom expansions:

* Add log enrichment pipelines that map hosts to business units and risk scores.
* Implement automated red-team vs blue-team scoring for exercises.
* Integrate a SOAR playbook to practice automated containment.

---

## Contributing

Contributions that improve safety, detection fidelity, and teaching clarity are welcome. Please open issues or PRs and include test cases and documentation.

---

## License

MIT License. See `LICENSE` for details.

---

