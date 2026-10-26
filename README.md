# LAB-WAZUH-W10-Sysmon-Kali
🛡️ Blue Team Lab: Wazuh SIEM + Sysmon para monitoramento em tempo real de um endpoint Windows 10 e detecção de ataques de força bruta (SSH) simulados via Kali Linux. 
# 🛡️ Laboratório de SIEM: Monitoramento de Atividades e Detecção de Ataques com Wazuh e Windows 10

Este projeto documenta a criação de um ambiente de laboratório de Segurança Cibernética (Blue Team) focado em centralização de logs, monitoramento em tempo real e detecção de ameaças. O objetivo principal foi configurar o **Wazuh SIEM** para monitorar um endpoint **Windows 10 Pro**, integrar logs avançados do **Sysmon** e detectar ataques simulados através do **Kali Linux**.
<img width="1915" height="837" alt="W10 Active" src="https://github.com/user-attachments/assets/04c9ccb3-c575-446f-a8da-83f08fe2ce64" />
<img width="1157" height="801" alt="Captura de tela 2026-10-25 235814" src="https://github.com/user-attachments/assets/57a1c3af-5aa0-4fd7-ba64-1b90d6e750db" />

---

## 🏗️ Arquitetura do Laboratório

O ambiente foi construído utilizando virtualização e conta com os seguintes componentes na mesma rede local:
*   **Virtuallização: ** Virtualbox. 
*   **Servidor SIEM:** VM Wazuh OVA (Centralizador de eventos e inteligência de regras).
*   **Endpoint Monitorado (Vítima):** Windows 10 Pro com Agente Wazuh ativo.
*   **Atacante:** Kali Linux (Utilizado para simular ataques).
<img width="1143" height="908" alt="VirtualBox" src="https://github.com/user-attachments/assets/bd6442c8-1e71-4649-8f89-e847ad3f3bad" />

---

## 🚀 Etapa 1: Instalação e Pareamento Seguros

1.  **Instalação do Agente:** O agente do Wazuh foi implantado no Windows 10 utilizando a interface visual `Wazuh Agent Manager`.
2.  **Geração da Chave de Autenticação:** Para garantir a segurança e criptografia do canal de comunicação (impedindo interceptação de dados ou spoofing na rede), foi extraída uma chave de autenticação exclusiva do Wazuh Server.
3.  **Configuração Visual:** O endereço IP do gerenciador e a chave de autenticação criptografada foram inseridos diretamente no painel do agente no Windows 10, estabelecendo o status de comunicação **Active**.

---

## 📊 Etapa 2: Endurecimento da Auditoria do Host (Hardening)

Por padrão, o Windows não gera logs detalhados de erros e ferramentas avançadas. Foram realizadas duas configurações críticas de defesa:

### 1. Ativação da Auditoria de Logon (`secpol.msc`)
Para permitir que o SIEM detectasse tentativas de invasão, foram ativadas as diretivas locais de monitoramento:
*   **Auditar eventos de logon:** Configurado para registrar *Sucesso* e *Falha*.
*   **Auditar eventos de logon de conta:** Configurado para registrar *Sucesso* e *Falha*.
*   *Resultado:* O Windows passou a gerar o **Event ID 4625** (Falha de Logon) a cada erro de senha.

### 2. Integração com o Microsoft Sysmon
O Sysmon foi instalado no Windows para fornecer visibilidade sobre criação de processos e comandos executados. O arquivo `ossec.conf` do agente Wazuh foi modificado para ler esse canal moderno de eventos:
```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

---

## ⚔️ Etapa 3: Simulação de Ataque e Detecção 

Foi simulado um ataque de **Força Bruta (Brute Force)** via protocolo **SSH (Porta 22)** utilizando a ferramenta **Hydra** integrada no Kali Linux.

### Comando de Ataque Executado:
```bash
hydra -l Duda -P /usr/share/wordlists/fasttrack.txt 10.0.2.5 ssh -t 4
```
<img width="1152" height="851" alt="Ataque do kali" src="https://github.com/user-attachments/assets/55ac9ce3-9afb-47f4-bb78-627f64035bc5" />

### 🛡️ Resposta do Host e Mitigação Ativa
Durante o ataque, o servidor OpenSSH do Windows 10 identificou a agressividade das requisições paralelas e mitigou a ameaça ativamente, derrubando o socket com o erro:
`[ERROR] all children were disabled due too many connection errors`

### 🎯 Correlação de Eventos no Wazuh SIEM
Apesar do bloqueio físico no host, o agente Wazuh capturou com sucesso as falhas de autenticação (**Event ID 4625**) geradas nos primeiros segundos do ataque. 
<img width="1157" height="731" alt="Logon Failure" src="https://github.com/user-attachments/assets/51e246f4-2861-4dcc-b962-68605a4a650d" />
<img width="1156" height="727" alt="Serviço Pri" src="https://github.com/user-attachments/assets/b4fca5f4-6ed3-4535-af93-b67cb7cc5bb4" />

---
## 🗺️ Mapeamento do Ataque na Matriz MITRE ATT&CK®

Para garantir que o laboratório reflita a engenharia de detecção usada no mercado corporativo, os alertas gerados pelo Wazuh foram correlacionados diretamente com a matriz **MITRE ATT&CK**:

| Tática Adversária | Técnica Utilizada | Identificador | Indicador de Comprometimento (IoC) | Regra de Detecção (Wazuh SIEM) |
| :--- | :--- | :--- | :--- | :--- |
| **Acesso Inicial** *(Initial Access)* | Força Bruta via SSH| `T1110` / `T1110.001` | Múltiplas falhas consecutivas de autenticação (Event ID 4625) originadas do mesmo IP de rede externa. | 
| **Reconhecimento** *(Discovery)* | Descoberta de Conta Local | `T1087` / `T1087.001` | 
<img width="1175" height="807" alt="Captura de tela 2026-10-25 235442" src="https://github.com/user-attachments/assets/cc6efd75-03dc-45a2-abd4-4ea62b810c06" />
<img width="1157" height="803" alt="Mitre 01" src="https://github.com/user-attachments/assets/35ee8a76-6595-4171-b00a-c3115f95f9e8" />

## 🛠️ Ferramentas e Conceitos Absorvidos
*   **SIEM/XDR:** Gerenciamento, análise e correlação de logs com Wazuh.
*   **Windows Security:** Auditoria de eventos nativos (IDs 4624, 4625) e políticas locais.
*   **Sysmon:** Monitoramento avançado de telemetria de processos no endpoint.
*   **Pentesting:** Simulação adversária controlada (Hydra, Crowbar, Nmap).
*   **Hardening:** Ajustes de segurança em sistemas operacionais para visibilidade de incidentes.
*   Estudante ativo de cibersegurança aberto a dicas e conselhos.
