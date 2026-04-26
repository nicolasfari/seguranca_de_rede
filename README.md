# seguranca_de_rede
# Segurança de Redes — Perímetro, Visibilidade e Monitoramento de Logs
---
> Este documento é baseado no módulo **Pre-Security / Network Security** da plataforma [TryHackMe] 
> Os exemplos de logs e cenários de ataque foram extraídos do material original da plataforma.  
> O objetivo é consolidar o aprendizado com minhas próprias anotações e análises.

## 1. Componentes de uma rede corporativa

Uma rede não é uma coleção aleatória de dispositivos — é uma estrutura organizada onde cada componente tem uma função específica. Do ponto de vista de segurança, entender o que cada um faz é o primeiro passo para identificar atividades suspeitas.

### Estações de trabalho (Endpoints)
São onde os funcionários trabalham no dia a dia. Também são o ponto de entrada mais comum para ataques — geralmente via phishing ou downloads maliciosos. Os logs de endpoint podem revelar processos maliciosos, mas os logs de rede costumam ser os primeiros a mostrar conexões C2 (Command & Control).

### Servidores de arquivos e bancos de dados
Armazenam o ativo mais valioso da empresa: os dados. São alvos prioritários de operadores de ransomware e campanhas de exfiltração. Comprometer um servidor de arquivos significa acesso em massa a informações críticas.

### Servidores de aplicação (Web, Email, VPN)
Por estarem expostos externamente, são alvos de alto valor. Devem ser monitorados constantemente para:
- Tentativas de exploração (ex: SQL Injection em aplicações web)
- Ataques de força bruta em serviços de email ou VPN
- IPs externos suspeitos interagindo com aplicações sensíveis

### Active Directory (AD)
É a espinha dorsal da identidade corporativa — gerencia usuários, grupos, computadores e permissões. Uma conta de administrador de domínio comprometida pode derrubar toda a organização. Monitorar o AD significa ficar de olho em:
- Múltiplas tentativas de login falhas (brute force)
- Logins de IPs externos incomuns ou em horários atípicos
- Contas acessando sistemas que normalmente não deveriam

### Roteadores e switches
São o sistema circulatório da rede. Se comprometidos, permitem ao atacante interceptar tráfego (MITM), redirecionar pacotes e abrir canais ocultos para a internet.

### Firewall / Dispositivos de perímetro
Principal gateway de segurança entre a rede interna e a internet. Inspeciona pacotes, aplica regras, e registra todas as tentativas de conexão — bloqueadas ou permitidas. Esses logs são frequentemente os **primeiros indicadores de ataque**.

---

## 2. Visibilidade de rede

> *"Você não pode defender o que não consegue ver."*

Visibilidade de rede é a capacidade de monitorar e entender o que está acontecendo em toda a rede. Sem ela, atividades maliciosas como infecções por malware, acesso não autorizado e exfiltração de dados passam completamente despercebidas.

Existem duas fontes principais de logs, e entender a diferença entre elas é essencial para reconstruir a linha do tempo de um ataque.

### Logs centrados no host
Gerados por dispositivos individuais (servidores, estações de trabalho). Fornecem visão detalhada do que aconteceu **dentro de uma máquina específica**.

Fontes principais:
- **Logs do SO**: Windows Event Logs, syslog (Linux) — registram logins, criação de processos, falhas de autenticação
- **Logs de aplicação**: servidores web (Apache, Nginx), bancos de dados (MySQL, MSSQL)
- **Ferramentas de segurança**: antivírus, EDR, HIDS

Úteis para responder perguntas como:
- Quais arquivos foram acessados, modificados ou deletados?
- Um processo malicioso foi criado?
- Um script não autorizado (ex: PowerShell) foi executado?
- O malware foi de fato executado, ou apenas recebido?

### Logs centrados na rede
Gerados por equipamentos de rede que monitoram o tráfego entre dispositivos. Mostram **o que aconteceu entre as máquinas** — origem, destino, portas, protocolos e ação tomada.

Fontes principais:
- **Firewalls**: histórico de conexões permitidas ou negadas
- **IDS/IPS**: alertas de ataques conhecidos (por assinatura) ou comportamentos anômalos
- **Roteadores/Switches**: dados de fluxo — quem falou com quem, por quanto tempo, quanta data
- **Web Proxies**: cada site visitado por cada usuário — ótimo para detectar exfiltração ou C2 via HTTP
- **VPN**: quem conectou à rede corporativa, de onde e quando

Úteis para:
- Detectar tentativas de reconhecimento antes de comprometer um endpoint
- Identificar conexões C2 de um host comprometido para um servidor externo
- Rastrear movimento lateral entre máquinas internas
- Detectar exfiltração de dados por volume anômalo de tráfego de saída

### A regra de ouro
> Os logs de host dizem o que aconteceu **dentro de uma sala**.  
> Os logs de rede dizem quem **entrou e saiu do prédio**.  
> Correlacionar os dois é o que constrói uma investigação completa.

---

## 3. Perímetro de rede

O perímetro é o limite entre a rede interna (zona confiável) e a internet (zona não confiável). Todo tráfego vindo de fora precisa passar por ele — e todo tráfego saindo também. É a primeira e mais importante linha de defesa.

### Componentes do perímetro

| Componente | Função |
|---|---|
| Firewall | Filtra e inspeciona tráfego entre redes interna e externa |
| Roteador/Gateway | Encaminha tráfego e aplica regras de acesso |
| DMZ | Zona buffer onde ficam servidores públicos (web, email, VPN) |
| VPN Gateway | Ponto de entrada seguro para acesso remoto |

### Por que o perímetro é crítico
Atacantes sempre começam sondando pelo lado de fora. Um perímetro mal configurado permite:
- Explorar serviços expostos desnecessariamente (RDP, MySQL, SMB abertos para a internet)
- Realizar port scanning e reconhecimento da rede
- Ataques de força bruta contra serviços de autenticação
- Exfiltração de dados por canais não monitorados

---

## 4. Monitoramento do perímetro na prática

Monitorar o perímetro significa usar firewalls, IDS/IPS e controle de acesso para examinar o tráfego e aplicar regras de segurança. O objetivo do analista é **separar o tráfego normal da atividade suspeita**.

### Cenário 1 — Port Scanning
  2025-09-22 08:30:05 BLOCK TCP 203.0.113.10:50001 -> 10.0.0.20:21
  2025-09-22 08:30:06 BLOCK TCP 203.0.113.10:50002 -> 10.0.0.20:22
  2025-09-22 08:30:08 BLOCK TCP 203.0.113.10:50003 -> 10.0.0.20:23
  2025-09-22 08:30:09 BLOCK TCP 203.0.113.10:50004 -> 10.0.0.20:25
  2025-09-22 08:30:11 BLOCK TCP 203.0.113.10:50005 -> 10.0.0.20:53

**O que está acontecendo:** O mesmo IP externo (`203.0.113.10`) tenta se conectar rapidamente a várias portas diferentes na mesma máquina interna.
**Análise:** Ataque clássico de port scanning. O atacante está mapeando quais serviços estão abertos para identificar um vetor de entrada.
**Indicador-chave:** mesmo IP de origem → múltiplos destinos em sequência rápida.

---

### Cenário 2 — Ataques a aplicação web (WAF Logs)

```
2025-09-22T09:14:46Z src_ip=[REDACTED] action=BLOCK request="GET /search.php?q=<script>alert('XSS')</script>" attack_type="XSS"
2025-09-22T09:15:42Z src_ip=[REDACTED] action=BLOCK request="GET /../../../../etc/passwd" attack_type="Directory Traversal"
```
**O que está acontecendo:** O WAF está bloqueando tentativas de XSS e Directory Traversal contra a aplicação web.
**Análise:** Atacante testando diferentes vetores contra o mesmo alvo. Cada `action=BLOCK` com um `attack_type` é um alerta de alta confiabilidade — o WAF não só bloqueou como explicou o motivo.
**Indicador-chave:** múltiplos `action=BLOCK` de mesmo IP com `attack_type` variados = reconhecimento ativo de vulnerabilidades.
---

### Cenário 3 — Força bruta em VPN
  2025-09-22 10:12:08 FAILED_AUTH TCP [REDACTED]:31249 -> 10.0.0.1:443 (user 'guest')
  2025-09-22 10:12:09 FAILED_AUTH TCP [REDACTED]:31250 -> 10.0.0.1:443 (user 'user')
  2025-09-22 10:12:11 FAILED_AUTH TCP [REDACTED]:31245 -> 10.0.0.1:443 (user 'admin')
  2025-09-22 10:12:15 FAILED_AUTH TCP [REDACTED]:31248 -> 10.0.0.1:443 (user 'admin')
  2025-09-22 10:12:21 SUCCESS_AUTH TCP 198.51.100.88:41233 -> 10.0.0.1:443 (user 'b.jones')
  
**O que está acontecendo:** Volume alto de `FAILED_AUTH` de um IP suspeito tentando usernames comuns (`admin`, `guest`, `user`). O `SUCCESS_AUTH` isolado é tráfego legítimo de funcionário.
**Análise:** Ataque de força bruta contra o gateway VPN usando uma lista de credenciais comuns.
**Indicador-chave:** mesmo IP → mesmo destino → alto volume de falhas em curto espaço de tempo
---

## 5. Padrões que todo analista SOC precisa reconhecer

| Padrão nos logs | Significado provável |
|---|---|
| Mesmo IP → várias portas em sequência | Port scanning (reconhecimento) |
| Mesmo IP → mesmo destino → muitas falhas | Força bruta (autenticação) |
| Tráfego em intervalos regulares e perfeitos | Malware em beacon (C2) |
| Volume alto de dados saindo para IP externo incomum | Exfiltração de dados |
| Conta acessando sistemas fora do padrão habitual | Movimento lateral ou comprometimento |

> **Contexto é tudo.** Um alerta de IDS que diz *por que* algo foi bloqueado vale muito mais do que um simples `BLOCK` no firewall sem explicação.


