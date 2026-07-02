# Home SOC Lab: Wazuh SIEM + pfSense + Windows Hardening

## Descrição
Este projeto documenta a implementação de um ecossistema de monitorização de segurança completo. O objetivo foi integrar telemetria avançada de um endpoint Windows (via Sysmon) e logs de rede perimetral (via pfSense) num servidor centralizado Wazuh SIEM.

---
---
### Relatório Detalhado em PDF
 Pode consultar a documentação completa com todos os passos, prints e resoluções de problemas aqui:
 **[Relatorio_Lab_SOC_Diana.pdf.pdf](https://github.com/user-attachments/files/29612035/Relatorio_Lab_SOC_Diana.pdf.pdf)**
---

## Arquitetura e Rede
O laboratório foi isolado num segmento de rede local para simular um ambiente corporativo:
*   **Wazuh Manager:** Ubuntu 24.04 LTS (IP: 10.0.0.101)
*   **Firewall/Gateway:** pfSense (WAN + LAN Segment - IP: 10.0.0.1)
*   **Endpoint:** Windows 11 Enterprise (Wazuh Agent + Sysmon)

---

## Jornada de Implementação

### 1. Servidor Wazuh (SIEM)
*   **Desafio:** Tentativa inicial de instalação no Ubuntu 26.04 (incompatibilidade detectada) e erro de "Low Disk Space".
*   **Solução:** Downgrade para Ubuntu 24.04 e expansão do disco virtual para **70GB**.
*   **Configuração:** Ativação do módulo de Vulnerability Detection e configuração do `ossec.conf` para aceitar logs remotos (Syslog porta 514).

### 2. Windows 11 & Telemetria Sysmon
*   **Hardening:** Instalação do Sysmon com a configuração de referência da **SwiftOnSecurity**.
*   **Bypass de Sistema:** Utilização do Regedit (`LabConfig`) para contornar exigências de TPM e RAM do Windows 11 em ambiente virtual.
*   **Troubleshooting:** Diagnóstico de instabilidade (BSOD) após instalação do Sysmon, resolvido com o aumento de recursos de RAM e CPU na VM.

### 3. Integração pfSense (Network Visibility)
*   Configuração do Syslog para exportar logs de sistema e firewall para o Wazuh.
*   **Lógica de Regras:** Identificação da hierarquia de regras (Regra nativa 87701 vs Regra customizada 100010).

---

## Inteligência de Deteção (Regras Customizadas)

### Deteção de Limpeza de Logs (Windows)
Identifica quando um utilizador tenta apagar o rasto de atividades usando o `wevtutil`.

```xml
<rule id="100002" level="12">
  <if_group>windows</if_group>
  <match>wevtutil</match>
  <description>ALERTA SOC: Ferramenta de limpeza de logs detetada!</description>
  <mitre><id>T1070.001</id></mitre>
</rule>
```
<img width="857" height="296" alt="image" src="https://github.com/user-attachments/assets/cdbe081a-b1b3-42a7-9d20-09fb4318cd6c" />


## Normalização de Logs pfSense
Força a visibilidade de logs que inicialmente eram ignorados pelo motor de análise.
```xml
<group name="pfsense_custom,">
  <rule id="100010" level="5">
    <if_sid>1002</if_sid>
    <match>syslogd|pfsense|filterlog|auth.notice</match>
    <description>pfSense: Atividade detetada agora</description>
  </rule>
</group>
```
<img width="999" height="451" alt="image" src="https://github.com/user-attachments/assets/d56b53d2-858e-49c9-a818-d0d8504b112f" />

## Lições Aprendidas (Troubleshooting)
1. Sincronização de Tempo: A discrepância de relógios entre máquinas descarta logs silenciosamente. O NTP é obrigatório.
2. Mapeamento de Campos: Inicialmente, tentei usar a tag <field> com o prefixo data.win.eventdata, mas a regra falhou. Aprendi que, para fins de laboratório e troubleshooting rápido, a tag <match> é mais eficiente pois realiza uma busca global no log bruto, ignorando falhas de mapeamento de campos .
3. Active Response: A automação de bloqueio de IPs (firewall-drop) transforma o SIEM numa ferramenta reativa potente.

### Evidências do Lab

<img width="1688" height="774" alt="image" src="https://github.com/user-attachments/assets/422d5f8c-bc19-4240-9372-b431192e184e" />

<img width="1714" height="706" alt="image" src="https://github.com/user-attachments/assets/117116e7-0a52-4f1f-bcb1-a89e91d57ddc" />


<img width="1542" height="783" alt="image" src="https://github.com/user-attachments/assets/214ab3ad-bceb-4886-84da-9d5f3a3166cd" />

<img width="1706" height="719" alt="image" src="https://github.com/user-attachments/assets/f848c894-de5c-4132-b0a0-581497f1ebc7" />


