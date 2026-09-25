# 🔵 pfSense Lab

> Laboratório corporativo virtualizado de cibersegurança com segmentação por VLANs, identidade centralizada via Active Directory e telemetria avançada de endpoints.

<img width="1369" height="558" alt="image" src="https://github.com/user-attachments/assets/fb7c50e5-3959-414e-857d-b90035e86234" />
<img width="1291" height="804" alt="image" src="https://github.com/user-attachments/assets/1e0aba98-0b52-4cc4-92cc-4a3164a5c67a" />
<img width="1311" height="815" alt="image" src="https://github.com/user-attachments/assets/8b547796-74bb-4ee6-82ec-aff912b1b475" />
<img width="1697" height="981" alt="image" src="https://github.com/user-attachments/assets/e59e380a-b8a1-4318-bc62-d400a5b8f358" />


---

## Visão Geral

Este repositório documenta a construção de um laboratório de Blue Team do zero, com foco em três pilares:

- **Segmentação de rede** departamental via VLANs (802.1Q)
- **Gestão centralizada de identidades** com Active Directory no Windows Server 2025
- **Telemetria de endpoints** para detecção e análise de ameaças (Sysmon + GPO de Auditoria)

O ambiente roda sobre **Proxmox VE** com o **pfSense** atuando como firewall e roteador principal, simulando uma rede corporativa real em escala de laboratório.

---

## Arquitetura

<img width="1024" height="559" alt="topologia" src="https://github.com/user-attachments/assets/d7301550-d065-4752-9982-2092994b8589" />


### Segmentação de Rede (VLANs)

| Segmento | VLAN | Sub-rede | Gateway | Hosts |
|:---------|:----:|:---------|:--------|:------|
| Core / Servidores | *Nativa* | `192.168.2.0/24` | `192.168.2.1` | DC-01 (`192.168.2.10`) |
| Financeiro | `10` | `192.168.10.0/24` | `192.168.10.1` | Estações do setor Financeiro |
| RH | `20` | `192.168.20.0/24` | `192.168.20.1` | Estações do setor de RH |
| TI | `30` | `192.168.30.0/24` | `192.168.30.1` | Estações do setor de TI |

---

## Stack de Tecnologias

| Camada | Tecnologia |
|:-------|:-----------|
| Hipervisor | Proxmox VE |
| Firewall / Roteador | pfSense (vtnet1 + sub-interfaces 802.1Q) |
| Controlador de Domínio | Windows Server 2025 — domínio `lab.local` |
| Cliente | Windows 11 (ingressado no domínio) |
| Telemetria | Sysmon (Sysinternals) + GPO de Auditoria Avançada |
| Framework de Detecção | MITRE ATT&CK |

---

## Implementação

### 1 — Infraestrutura Base e Active Directory

- Deploy do **Windows Server 2025** como `DC-01` no domínio `lab.local`
- Estrutura de Unidades Organizacionais (OUs) sob `DEPARTAMENTOS`:
  - `OU=FINANCEIRO`
  - `OU=RH`
  - `OU=TI`
- Ingresso do cliente Windows 11 ao domínio

### 2 — Segmentação L2/L3 (Proxmox + pfSense)

- Ativação de **VLAN Aware** na bridge `vmbr1` do Proxmox
- Criação de sub-interfaces 802.1Q (VLANs 10, 20, 30) na interface LAN (`vtnet1`) do pfSense
- Configuração do **servidor DHCP por VLAN**, com DNS apontando para `DC-01` (`192.168.2.10`)
- Regras de firewall para tráfego inter-VLAN e acesso à internet
- **Desativação do Hardware Offloading** no pfSense para processamento correto de tags 802.1Q sobre drivers VirtIO

### 3 — Telemetria e Auditoria de Segurança

- Deploy da **GPO `GPO-BlueTeam-Audit`** com:
  - Auditoria Avançada de Criação de Processos habilitada
  - Captura da linha de comando completa (**Event ID 4688**)
- Implementação do **Sysmon** nos endpoints com perfil defensivo alinhado às técnicas do MITRE ATT&CK

---

## Validação

Cada etapa foi validada com testes objetivos:

**Camada 2 / DHCP**
> Associação da tag VLAN `10` na vNIC do Windows 11 → concessão de IP na sub-rede `192.168.10.0/24` confirmada.

**Roteamento inter-VLAN e DNS**
> Ping e `nslookup lab.local` / `nslookup google.com` a partir do cliente segmentado → conectividade com `DC-01` e internet confirmada.

**Logs de Segurança**
> Geração de eventos verificada no Event Viewer em:
> - `Microsoft-Windows-Sysmon/Operational`
> - `Security` — Event ID 4688 com linha de comando expandida

---
