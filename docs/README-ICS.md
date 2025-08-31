# ICS Protocol Fuzzing Laboratory

![Build Status](https://github.com/your-org/ics-fuzzing-lab/workflows/CI/badge.svg)
![Security Scan](https://github.com/your-org/ics-fuzzing-lab/workflows/Security/badge.svg)
![License](https://img.shields.io/github/license/your-org/ics-fuzzing-lab)

A **production-grade ICS cybersecurity research lab** for protocol vulnerability analysis, fuzzing, and security testing in a fully isolated environment.

---

## 🚀 Features

- 🔒 **Complete Network Isolation** (VLAN segmentation, pfSense firewall)  
- 🛠 **Protocol Simulators** (Modbus, DNP3, IEC 61850, EtherNet/IP)  
- 📊 **Advanced Monitoring** (ELK + Suricata IDS, Zeek)  
- 🤖 **Automated Fuzzing** with custom protocol support  
- ⚡ **High Availability** with clustered infrastructure  
- 🏢 **Enterprise Security**: Centralized logging + default-deny firewall  

---

## 📦 Quick Start

### Single Node Evaluation

```bash
git clone https://github.com/your-org/ics-fuzzing-lab.git
cd ics-fuzzing-lab

# Deploy minimal lab
./scripts/deploy-minimal.sh

# Test setup
./scripts/test-minimal-lab.sh
```

**Access Points:**  
- Lab Management → https://192.168.40.50  
- Firewall Admin → https://192.168.40.1  
- Fuzzing Interface → `ssh ubuntu@192.168.30.10`  

### Docker Development Mode

```bash
docker-compose up -d
docker exec -it ics-fuzzing-dev bash

# Run protocol test
./test-modbus-simulator.py --target 192.168.30.20
```

---

## 🏗 Full Installation

For production-grade deployment, follow the [Implementation Plan](docs/implementation-plan.md).  
Covers **hardware setup**, **Proxmox cluster config**, **Terraform/Ansible automation**, and **security hardening**.

---

## 🌐 Network Architecture

| VLAN | Network        | Purpose   | Access Policy              |
|------|----------------|-----------|---------------------------|
| 30   | 192.168.30.0/24| ICS Lab   | Isolated from production   |
| 40   | 192.168.40.0/24| Management| Administrative access      |
| 50   | 192.168.50.0/24| Logging   | Centralized monitoring     |
| 10   | 192.168.10.0/24| LAN       | Simulated corporate network|
| 20   | 192.168.20.0/24| DMZ       | External services          |
| 60   | 192.168.60.0/24| Guest     | Internet-only access       |

```
Internet
    ↓
[WAN Redundancy] ← Raspberry Pi VRRP
    ↓
[pfSense Firewall] ← Default-deny
    ↓
[OVS Switch] ← VLAN trunk + port mirroring
    ↓
[ICS Lab VLAN 30] ← Isolated fuzzing/testing
```

---

## 🔒 Security Highlights

- SSH key-only authentication (no passwords)  
- Role-based access control + audit logging  
- Dedicated mirror ports for traffic monitoring  
- Encrypted storage for logs & test artifacts  

---

## 🧑‍💻 Development

```bash
cd src/protocol-modules
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Create new protocol module
./create-protocol-module.py --protocol iec104 --template base

# Run tests
pytest tests/test_iec104_module.py
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for details.  

---

## ❓ FAQ

**Q: Can I test wireless ICS protocols?**  
A: Planned in **v2.0** (WirelessHART, ISA100.11a).  

**Q: What’s the hardware cost?**  
A: ~$15k–$25k depending on servers and networking gear.  

**Q: Is this for production testing?**  
A: 🚫 No — for **isolated research only**. Never connect to live ICS networks.  

---

## 📅 Roadmap

- [ ] v2.0 → Wireless ICS protocol support  
- [ ] v2.1 → ML-based anomaly detection  
- [ ] v2.2 → Integration with pentest frameworks  
- [ ] v3.0 → Cloud deployment (AWS, Azure, GCP)  

---

## 🔗 Related Projects

- [OVS Layer 2 Switch](https://github.com/your-org/ovs-layer2-switch)  
- [Pi WiFi Bridge](https://github.com/KevinMulcahy/pi-wifi-bridge)  
- [Industrial Security Framework](https://github.com/dark-lbp/isf)  

---

## 📜 License

MIT License – see [LICENSE](LICENSE)  

⚠️ **Disclaimer:** Use for authorized research only. Authors assume no liability for misuse.

---

## 🙌 Acknowledgments

- ICS security research community  
- Open source ICS protocol implementations  
- Enterprise virtualization & networking projects  

---

## 📬 Support

- 📖 Docs → [docs/](docs/)  
- 🐞 Issues → GitHub Issues tab  
- 🔐 Security → security@your-domain.com  
- 💬 Community → GitHub Discussions  

