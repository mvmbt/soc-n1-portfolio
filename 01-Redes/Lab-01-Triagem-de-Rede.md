# Relatório de Triagem #01 — Análise de Alertas de Rede (Firewall & DNS)

- **Analista:** Marcos Vinicius Morelli Batista
- **Nível:** SOC Tier 1 (Triagem Inicial)
- **Foco Técnico:** Classificação de IP (Público vs. Privado), Portas Críticas (`3389`, `445`, `53`) e Detecção de Exfiltração DNS.

---

## 1. Alerta #01 — Força Bruta e Acesso Remoto via RDP
- **Timestamp:** `2026-11-25 20:41:12`
- **Source IP:** `198.51.100.42` *(Public IP — Externo)*
- **Destination IP:** `10.10.1.50` *(Private IP — Interno)*
- **Porta / Protocolo:** `3389 / TCP (RDP)`
- **Análise Técnica:** Identificadas 420 tentativas de autenticação falhas consecutivas seguidas de uma sessão permitida (`ACTION: ALLOW`), caracterizando ataque de *Brute Force* bem-sucedido contra serviço de Área de Trabalho Remota exposto.
- **Veredito:** **True Positive (Verdadeiro Positivo)**
- **Ação Tomada:** Bloqueio imediato do IP `198.51.100.42` no firewall de borda, isolamento do host `10.10.1.50` e escalonamento para o time **SOC N2** para resposta a incidentes e redefinição de credenciais.

---

## 2. Alerta #02 — Tráfego SMB Interno
- **Timestamp:** `2026-11-25 21:00:03`
- **Source IP:** `10.10.1.12` (`SERVIDOR-BACKUP-TI`)
- **Destination IP:** `10.10.1.50`
- **Porta / Protocolo:** `445 / TCP (SMB)`
- **Análise Técnica:** Conexão única estabelecida (`ESTABLISHED`) exatamente às `21:00` a partir do servidor oficial de backup da TI, em conformidade com a janela programada no inventário de ativos.
- **Veredito:** **False Positive (Falso Positivo)**
- **Ação Tomada:** Alerta encerrado (*Closed*) e documentado na base de conhecimento como comportamento esperado.

---

## 3. Alerta #03 — Suspeita de Exfiltração via DNS Tunneling
- **Timestamp:** `2026-11-25 21:14:55`
- **Source IP:** `10.10.1.77` (`PC-FINANCEIRO-04`)
- **Destination IP:** `8.8.8.8`
- **Porta / Protocolo:** `53 / UDP (DNS)`
- **Análise Técnica:** Volume anômalo de 310 requisições DNS em 90 segundos direcionadas ao domínio suspeito `c2-drop.xyz` contendo subdomínios codificados em hexadecimal (`53656e68615f44697265746f726961`), caracterizando exfiltração de dados via **DNS Tunneling**.
- **Veredito:** **True Positive (Verdadeiro Positivo)**
- **Ação Tomada:** Contenção imediata com isolamento de rede do endpoint `PC-FINANCEIRO-04`, bloqueio do domínio `c2-drop.xyz` no DNS interno/firewall e escalonamento urgente para o **SOC N2**.
