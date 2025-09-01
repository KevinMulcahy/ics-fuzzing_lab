# 🛠 ICS Fuzzing Lab – Roadmap

This roadmap tracks the **features, structure, and improvements** needed to mature the repo into a full production-ready project.  

---

## ✅ Core Deliverables

- [ ] **Protocol Fuzzers**  
  - [ ] Modbus TCP/RTU fuzzer (`src/protocol_modules/modbus_fuzzer.py`)  
  - [ ] DNP3 fuzzer (`src/protocol_modules/dnp3_fuzzer.py`)  
  - [ ] IEC 61850 fuzzer (`src/protocol_modules/iec61850_fuzzer.py`)  
  - [ ] EtherNet/IP fuzzer (`src/protocol_modules/enip_fuzzer.py`)  

- [ ] **Protocol Simulators**  
  - [ ] Modbus simulator  
  - [ ] DNP3 simulator  
  - [ ] Config-driven YAML support (`config/protocols/`)  

- [ ] **Utilities**  
  - [ ] Shared logging helper (`src/utils/logging.py`)  
  - [ ] Packet builder helpers (`src/utils/packets.py`)  

---

## 🧪 Testing & QA

- [ ] Unit Tests (`tests/`)  
  - [ ] `test_modbus_fuzzer.py`  
  - [ ] `test_dnp3_fuzzer.py`  
  - [ ] `test_iec61850_fuzzer.py`  

- [ ] Integration Tests  
  - [ ] Spin up simulator + verify fuzzing traffic  
  - [ ] Validate logs sent to ELK stack  

- [ ] CI/CD Improvements  
  - [x] GitHub Actions workflow (`.github/workflows/ci.yml`)  
  - [ ] Add coverage reporting (`pytest-cov`)  
  - [ ] Upload coverage reports to Codecov or Coveralls  

---

## ⚙️ Infrastructure & Deployment

- [ ] **Scripts (`scripts/`)**  
  - [x] `deploy-minimal.sh` (skeleton exists)  
  - [ ] `run-fuzzer.sh` (launches campaigns easily)  
  - [ ] `test-network.sh` (simple connectivity check)  
  - [ ] `cleanup.sh` (teardown/reset lab)  

- [ ] **Containerization**  
  - [ ] `docker-compose.yml` (ELK + fuzzers + simulators)  
  - [ ] Add prebuilt Docker images  

- [ ] **Infra-as-Code**  
  - [ ] Terraform templates for Proxmox/VMware  
  - [ ] Ansible playbooks for VM provisioning  

---

## 📖 Documentation

- [ ] `docs/implementation-plan.md` (step-by-step deployment)  
- [ ] `docs/protocol-development-guide.md` (how to add new protocols)  
- [ ] `docs/usage-examples.md` (sample fuzzing campaigns)  
- [ ] Network architecture diagram (PNG/SVG)  
- [ ] Example PCAPs (fuzzing traffic captures)  

---

## 🔒 Security & Compliance

- [ ] `SECURITY.md` (vulnerability disclosure process)  
- [ ] SSH key-only config examples  
- [ ] Default-deny firewall policy examples  
- [ ] Role-based access guide  

---

## 🌍 Community & Governance

- [ ] `CONTRIBUTING.md` (PR workflow, coding style)  
- [ ] `CODE_OF_CONDUCT.md`  
- [ ] `LICENSE` (MIT text in repo root)  
- [ ] GitHub Discussions enabled  

---

## 🎯 Future Roadmap

- [ ] v2.0 → Wireless ICS protocol support (WirelessHART, ISA100.11a)  
- [ ] v2.1 → ML-based anomaly detection  
- [ ] v2.2 → Integration with pentest frameworks  
- [ ] v3.0 → Cloud deployment options (AWS, Azure, GCP)  

---

✅ You can **check off items as you go** either by editing `ROADMAP.md` or by copying this into a **GitHub Project board** (Kanban style).  
