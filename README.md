# pfSense LAB

## 📌 Descrição do Projeto
Este repositório documenta a implementação de um laboratório corporativo virtualizado de Cibersegurança (Blue Team), focado em segmentação de rede por departamentos (VLANs), gestão centralizada de identidades via Active Directory (Windows Server 2025) e captura avançada de telemetria em *endpoints* para análise de ameaças.

---

## 📐 Arquitetura de Rede e Topologia

O ambiente foi construído sobre o hipervisor **Proxmox VE**, utilizando o **pfSense** como firewall/roteador principal e *trunking* 802.1Q habilitado na bridge virtual (`vmbr1`).

### Tabela de Segmentação de Redes (VLANs)

| Segmento / Departamento | ID da VLAN | Sub-rede | Gateway (pfSense) | Descrição |
| :--- | :--- | :--- | :--- | :--- |
| **Core / Servidores** | *Nativa (`vmbr1`)* | `192.168.2.0/24` | `192.168.2.1` | Controlador de Domínio (`DC-01`: `192.168.2.10`) |
| **FINANCEIRO** | `VLAN 10` | `192.168.10.0/24` | `192.168.10.1` | Estações de trabalho do setor Financeiro |
| **RH** | `VLAN 20` | `192.168.20.0/24` | `192.168.20.1` | Estações de trabalho do setor de RH |
| **TI** | `VLAN 30` | `192.168.30.0/24` | `192.168.30.1` | Estações de trabalho do setor de TI |

---

## 🚀 Etapas da Implementação

### 1. Infraestrutura Base e Serviços de Domínio
* Implementação do **Windows Server 2025** (`DC-01`) no domínio `lab.local`.
* Criação de Unidades Organizacionais (OUs) organizadas sob a estrutura hierárquica `DEPARTAMENTOS` (`FINANCEIRO`, `RH`, `TI`).
* Integração do cliente **Windows 11** ao domínio Active Directory.

### 2. Segmentação L2/L3 no Proxmox e pfSense
* Habilitação da funcionalidade **VLAN Aware** na bridge `vmbr1` do Proxmox VE.
* Criação de sub-interfaces 802.1Q (VLANs 10, 20 e 30) associadas à interface LAN (`vtnet1`) do pfSense.
* Configuração do **Servidor DHCP** por sub-rede, apontando obrigatoriamente o DNS para o Controlador de Domínio (`192.168.2.10`).
* Aplicação de regras no pfSense para permissão de tráfego inter-VLAN e acesso à Internet.
* Adjustment de infraestrutura virtual: Desativação do *Hardware Offloading* no pfSense para correção de processamento de *tags* 802.1Q sobre drivers VirtIO.

### 3. Elevação de Telemetria e Auditoria
* Criação e aplicação de GPO corporativa (`GPO-BlueTeam-Audit`) para habilitação da **Auditoria Avançada de Processos** e captura da linha de comandos (**Event ID 4688**).
* Implementação do **Sysmon (System Monitor - Sysinternals)** nos *endpoints* utilizando perfil defensivo otimizado para mitigação e deteção de técnicas MITRE ATT&CK.

---

## 🔍 Testes de Validação Efetuados

1. **Validação de Camada 2 / DHCP:** Associação da tag de VLAN `10` na vNIC do cliente Windows 11 no Proxmox, resultando na concessão bem-sucedida de IP na sub-rede `192.168.10.0/24`.
2. **Validação Roteamento Inter-VLAN e DNS:** Confirmação de conectividade via ICMP e consultas DNS (`nslookup lab.local` e `nslookup google.com`) a partir do cliente segmentado até ao `DC-01` e Internet.
3. **Validação de Logs de Segurança:** Verificação da geração de eventos no *Event Viewer* em `Microsoft-Windows-Sysmon/Operational` e `Security` (Event ID 4688 com linha de comando expandida).
