# Zabbix Templates — Omnissa Horizon & UAG

Raccolta di template Zabbix per il monitoraggio completo di ambienti **Omnissa Horizon 8** (versione 2503+) e **Omnissa Unified Access Gateway (UAG)**.

---

## Indice

- [Template disponibili](#template-disponibili)
- [Prerequisiti generali](#prerequisiti-generali)
- [Installazione](#installazione)
- [Template 1 — Omnissa Horizon Connection Server (REST API)](#template-1--omnissa-horizon-connection-server-rest-api)
- [Template 2 — Omnissa Horizon Connection Server with Graphs](#template-2--omnissa-horizon-connection-server-with-graphs)
- [Template 3 — Omnissa UAG System by SNMP](#template-3--omnissa-uag-system-by-snmp)
- [Compatibilità](#compatibilità)
- [Note e limitazioni](#note-e-limitazioni)

---

## Template disponibili

| File | Descrizione | Metodo |
|------|-------------|--------|
| `omnissa_horizon_connection_server.yaml` | Monitoraggio Horizon via REST API | HTTP (Script items) |
| `omnissa_horizon_connection_server_GRAPH.yaml` | Come sopra + grafici preconfigurati | HTTP (Script items) |
| `omnissa_uag_system_snmp.yaml` | Monitoraggio sistema UAG (OS) | SNMP v2c |

---

## Prerequisiti generali

- **Zabbix** versione **7.4** o superiore
- **Zabbix Server** o **Proxy** con accesso di rete agli host monitorati

---

## Installazione

1. In Zabbix aprire **Data collection → Templates**
2. Cliccare **Import** in alto a destra
3. Selezionare il file YAML
4. Cliccare **Import**
5. Collegare il template all'host e configurare le macro come descritto nelle sezioni seguenti

> **Consiglio:** usare `omnissa_horizon_connection_server_GRAPH.yaml` al posto della versione base se si desidera avere i grafici già pronti.

---

## Template 1 — Omnissa Horizon Connection Server (REST API)

**File:** `omnissa_horizon_connection_server.yaml`

### Descrizione

Monitora un pod Omnissa Horizon 8 tramite le **Horizon Server REST API** (versione 2503+). Ogni item esegue autonomamente il ciclo di autenticazione (login → chiamata API → logout) tramite JavaScript `HttpRequest`, senza dipendenze esterne.

### Cosa monitora

#### Endpoint API interrogati

| Item master | Endpoint | Intervallo |
|-------------|----------|-----------|
| Connection Servers | `GET /rest/monitor/v2/connection-servers` | 1 min |
| Virtual Centers | `GET /rest/monitor/v4/virtual-centers` | 2 min |
| Farms RDSH | `GET /rest/monitor/v2/farms` | 2 min |
| RDS Servers | `GET /rest/monitor/v2/rds-servers` | 2 min |
| RDS Servers count metrics | `GET /rest/monitor/v1/rds-servers/count-metrics` | 1 min |
| Gateways (UAG) | `GET /rest/monitor/v5/gateways` | 2 min |
| AD Domains | `GET /rest/monitor/v4/ad-domains` | 5 min |
| Event Database | `GET /rest/monitor/v2/event-database` | 5 min |
| Desktop Pools | `GET /rest/monitor/v3/desktop-pools/metrics` | 3 min |
| Health Metrics | `GET /rest/monitor/v1/health-metrics` | 1 min |
| Sessions Metrics | `GET /rest/monitor/v1/sessions/metrics` | 1 min |
| License Usage | `GET /rest/monitor/v1/licenses/usage-metrics` | 5 min |
| Machines Count | `GET /rest/monitor/v1/machines/count-metrics` | 1 min |
| True SSO | `GET /rest/monitor/v2/true-sso` | 5 min |
| SAML Authenticators | `GET /rest/monitor/v4/saml-authenticators` | 5 min |

#### Metriche per Connection Server (LLD)

Per ogni CS scoperto nel pod vengono creati automaticamente:

| Metrica | Descrizione |
|---------|-------------|
| Status | OK / ERROR / NOT_RESPONDING / RESTART_REQUIRED |
| Connection count | Connessioni attive |
| Tunnel connection count | Connessioni in tunnel |
| Session threshold | Limite massimo configurato |
| BLAST sessions | Sessioni attive con protocollo BLAST Extreme |
| PCoIP sessions | Sessioni attive con protocollo PCoIP |
| Connection utilization % | Utilizzo % del threshold |
| Replication status | Stato replica LDAP con i peer |
| Certificate valid | Validità certificato SSL (1/0) |
| Certificate expiry (days) | Giorni alla scadenza del certificato |
| Default certificate | Segnala se è ancora in uso il certificato self-signed Horizon |
| Version | Versione e build Horizon installata |

#### Metriche aggregate (pod)

| Metrica | Descrizione |
|---------|-------------|
| CS count / CS OK count | Conteggio CS totali e in stato OK |
| Total connections | Somma connessioni su tutti i CS |
| Sessions local / remote / total | Sessioni per tipo da `/sessions/metrics` |
| License concurrent usage | Licenze utilizzate attualmente |
| License limit | Limite totale licenze |
| License utilization % | Percentuale di utilizzo |
| License highest usage | Picco storico di utilizzo |
| Machines available / connected / disconnected | Conteggi macchine VDI per stato (solo desktop virtuali) |
| Machines error count | Macchine VDI in stato ERROR |
| Machines agent unreachable | Macchine VDI con agent non raggiungibile |
| RDS Servers available / connected / disconnected | Conteggi server RDSH per stato |
| RDS Servers error count | Server RDSH in stato ERROR |
| RDS Servers agent unreachable | Server RDSH con agent non raggiungibile |
| RDS Servers maintenance count | Server RDSH in manutenzione |
| RDS Servers total count | Totale server RDSH calcolato |
| Desktop Pools count | Numero pool configurati |
| Event Database status | CONNECTED / DISCONNECTED / NOT_CONFIGURED |

#### LLD — Componenti scoperti automaticamente

| Discovery Rule | Cosa crea |
|---------------|-----------|
| Connection Servers | Item + trigger per ogni CS del pod |
| Virtual Centers | Status e certificati per ogni vCenter |
| AD Domains | Status raggiungibilità per ogni dominio |
| Gateways (UAG) | Status, connessioni e certificati per ogni UAG |
| Farms RDSH | Health, RDS count, sessioni per ogni farm |
| RDS Servers | Status, sessioni, agent version per ogni server RDS |
| Desktop Pools | Health, macchine available/connected/error per ogni pool |
| True SSO | Status per ogni connettore True SSO |
| SAML Authenticators | Status e certificati per ogni authenticator SAML |

#### Copertura certificati PKI

| Componente | Non valido | Scadenza Warning | Scadenza Critico | Self-signed default |
|-----------|:---------:|:----------------:|:----------------:|:-------------------:|
| Connection Server | HIGH ✅ | WARNING ✅ | HIGH ✅ | INFO ✅ |
| Virtual Center | HIGH ✅ | WARNING ✅ | HIGH ✅ | — |
| Gateway (UAG) | HIGH ✅ | WARNING ✅ | HIGH ✅ | — |
| SAML Authenticator | HIGH ✅ | WARNING ✅ | HIGH ✅ | — |

Le soglie di scadenza sono configurabili tramite macro `{$HORIZON.CERT.EXPIRY.WARN}` e `{$HORIZON.CERT.EXPIRY.CRIT}`.

#### Trigger

**Trigger globali (pod):**

| Trigger | Severità | Condizione |
|---------|----------|------------|
| TUTTI i CS in errore — SERVIZIO INDISPONIBILE | DISASTER | CS OK = 0 |
| Uno o più CS in errore | HIGH | CS OK < CS totali |
| Event Database disconnesso | WARNING | Status ≠ CONNECTED e ≠ NOT_CONFIGURED |
| Sessioni totali elevate | WARNING | Totale > `{$HORIZON.SESSION.WARN}` |
| Macchine VDI in stato ERROR | AVERAGE | VDI error count > `{$HORIZON.MACHINE.ERR.WARN}` |
| Macchine VDI agent non raggiungibili | WARNING | VDI unreachable count > 0 |
| RDS Server in stato ERROR | AVERAGE | RDSH error count > 0 |
| RDS Server agent non raggiungibile | WARNING | RDSH unreachable count > 0 |
| Licenze quasi esaurite | HIGH | Utilizzo > `{$HORIZON.LICENSE.CRIT.PCT}` |
| Utilizzo licenze elevato | WARNING | Utilizzo > `{$HORIZON.LICENSE.WARN.PCT}` |

**Trigger LLD per componente** (si generano per ogni istanza scoperta):
- CS: status, certificato invalido/in scadenza, utilizzo connessioni, replica LDAP, certificato default
- vCenter: status, certificato
- AD Domain: status
- Gateway: status, certificato
- Farm: health
- RDS Server: status (stati normali: AVAILABLE, OK), sessioni, agent version per server (LLD)
- RDS Server globali: error count e agent unreachable (da `/rds-servers/count-metrics`)
- Desktop Pool: health, errori provisioning, agent unreachable, nessuna macchina available
- True SSO: status
- SAML: status, certificato

### Prerequisiti specifici

- Accesso **TCP 443** dal Zabbix Server/Proxy al Connection Server
- Account AD con ruolo **Read-only Administrators** in Horizon Console:
  `Horizon Console → Settings → Administrators → Add`
- Se il CS usa un **certificato self-signed**, aggiungere il certificato al trust store del Zabbix Server:

```bash
openssl s_client -connect <IP-CS>:443 </dev/null 2>/dev/null \
  | openssl x509 -outform PEM \
  > /usr/local/share/ca-certificates/horizon-cs.crt
update-ca-certificates
systemctl restart zabbix-server
```

### Configurazione host Zabbix

1. Creare un host con **interfaccia Agent** puntando all'IP/FQDN del Connection Server (l'interfaccia serve solo come riferimento, non viene usata per la raccolta dati)
2. Collegare il template
3. Impostare le macro nella scheda **Macros** dell'host

### Macro

| Macro | Default | Tipo | Descrizione |
|-------|---------|------|-------------|
| `{$HORIZON.URL}` | `https://horizon.example.com` | Text | URL base del CS. Accetta: `https://IP`, `https://IP/rest`, `https://IP/rest/` |
| `{$HORIZON.DOMAIN}` | `DOMAIN` | Text | **NetBIOS short name** del dominio AD (es. `CONTOSO`, non `contoso.local`) |
| `{$HORIZON.USER}` | `svc-zabbix` | Text | Username senza prefisso dominio (es. `svc-zabbix`, non `CONTOSO\svc-zabbix`) |
| `{$HORIZON.PASSWORD}` | _(vuoto)_ | **Secret** | Password account di servizio |
| `{$HORIZON.CERT.EXPIRY.WARN}` | `30` | Text | Giorni alla scadenza certificato per trigger WARNING |
| `{$HORIZON.CERT.EXPIRY.CRIT}` | `7` | Text | Giorni alla scadenza certificato per trigger HIGH |
| `{$HORIZON.CONN.WARN.PCT}` | `80` | Text | Soglia % utilizzo connection threshold per CS |
| `{$HORIZON.SESSION.WARN}` | `2000` | Text | Soglia sessioni totali per trigger WARNING |
| `{$HORIZON.LICENSE.WARN.PCT}` | `80` | Text | Soglia % utilizzo licenze concurrent per WARNING |
| `{$HORIZON.LICENSE.CRIT.PCT}` | `95` | Text | Soglia % utilizzo licenze concurrent per HIGH |
| `{$HORIZON.MACHINE.ERR.WARN}` | `0` | Text | Numero macchine in errore per trigger AVERAGE (0 = al primo errore) |
| `{$HORIZON.LLD.POOL.FILTER}` | `.*` | Text | Regex per filtrare pool nella LLD (default: tutti) |
| `{$HORIZON.LLD.RDS.FILTER}` | `.*` | Text | Regex per filtrare RDS server nella LLD (default: tutti) |

> ⚠️ Impostare `{$HORIZON.PASSWORD}` come tipo **Secret text** per evitare che la password sia visibile nel frontend Zabbix.

### Verifica rapida

Dopo il collegamento del template, controllare in **Monitoring → Latest data** (filtrando per l'host) che l'item `Horizon: Connection Servers raw data` mostri un array JSON entro 1 minuto. Se compare un errore, cliccare sull'ultimo valore per leggere il messaggio diagnostico.

Test manuale con curl dal server Zabbix:
```bash
curl -k -s -X POST https://<IP-CS>/rest/login \
  -H "Content-Type: application/json" \
  -d '{"domain":"DOMAIN","username":"svc-zabbix","password":"PASSWORD"}' \
  | python3 -m json.tool
```
La risposta deve contenere `access_token` e `refresh_token`.

---

## Template 2 — Omnissa Horizon Connection Server with Graphs

**File:** `omnissa_horizon_connection_server_GRAPH.yaml`

### Descrizione

Versione identica al Template 1 con l'aggiunta di **grafici preconfigurati**. Stesse macro, stessi trigger, stessa logica di monitoraggio.

### Grafici inclusi

**Grafici globali** (visibili in Monitoring → Hosts → Graphs):

| Grafico | Descrizione |
|---------|-------------|
| Horizon: Sessioni attive | Sessioni locali, remote e totali nel tempo |
| Horizon: Connection Servers - connessioni | CS OK/totali + connessioni aggregate (asse Y destra) |
| Horizon: Stato macchine VDI | Available / Connected / Disconnected / Error / Unreachable (solo desktop virtuali) |
| Horizon: Stato RDS Servers (RDSH) | Available / Connected / Disconnected / Error / Unreachable / Maintenance per server RDSH |
| Horizon: Utilizzo licenze concurrent | Utilizzo corrente, limite e percentuale (asse Y destra) |

**Graph prototypes LLD** (si generano automaticamente per ogni istanza scoperta):

| Graph prototype | LLD | Descrizione |
|----------------|-----|-------------|
| `CS [{#CS.NAME}]: Sessioni per protocollo` | Connection Servers | BLAST e PCoIP per ogni CS |
| `CS [{#CS.NAME}]: Scadenza certificato (giorni)` | Connection Servers | Andamento giorni alla scadenza cert |
| `Pool [{#POOL.NAME}]: Macchine per stato` | Desktop Pools | Available / Connected / Error / Unreachable per pool |

### Prerequisiti, macro e configurazione

Identici al Template 1. Fare riferimento alla sezione precedente.

---

## Template 3 — Omnissa UAG System by SNMP

**File:** `omnissa_uag_system_snmp.yaml`

### Descrizione

Monitora le **risorse di sistema** di un nodo Omnissa Unified Access Gateway tramite **SNMP v2c**. UAG è basato su Photon OS con Net-SNMP standard; non espone MIB proprietarie per le funzioni di gateway — queste vengono monitorate tramite il Template 1/2 (Horizon REST API). Questo template copre la parte infrastrutturale del nodo UAG: CPU, RAM, swap, load average, interfacce di rete e filesystem.

### Cosa monitora

#### Item standalone

| Metrica | OID / MIB | Descrizione |
|---------|-----------|-------------|
| CPU utilization | UCD-SNMP-MIB | Utilizzo CPU totale % |
| CPU idle | UCD-SNMP-MIB | Percentuale CPU idle |
| CPU user / system / iowait | UCD-SNMP-MIB | Breakdown utilizzo CPU |
| Load average 1 / 5 / 15 min | UCD-SNMP-MIB | Load average di sistema |
| Memory total / available / used | HOST-RESOURCES-MIB | RAM in byte |
| Memory utilization % | Calcolato | Percentuale RAM usata |
| Swap total / free | UCD-SNMP-MIB | Swap in byte |
| Swap utilization % | Calcolato | Percentuale swap usata |
| System uptime | SNMP standard | Uptime del nodo |
| SNMP availability | Internal | Raggiungibilità SNMP |

#### LLD — Interfacce di rete

Per ogni interfaccia corrispondente al filtro `{$NET.IF.IFNAME.MATCHES}`:

| Metrica | Descrizione |
|---------|-------------|
| Inbound traffic (bps) | Traffico in ingresso |
| Outbound traffic (bps) | Traffico in uscita |
| Interface errors in/out | Errori per secondo |
| Interface utilization % | Utilizzo banda |
| Operational status | UP / DOWN |

#### LLD — Filesystem

Per ogni filesystem non escluso da `{$VFS.FS.FSNAME.NOT_MATCHES}`:

| Metrica | Descrizione |
|---------|-------------|
| Space used / total | Spazio disco in byte |
| Space utilization % | Percentuale spazio usato |
| Inode utilization % | Percentuale inode usati |

#### Trigger

| Trigger | Severità | Condizione |
|---------|----------|------------|
| CPU utilizzo critico | HIGH | CPU util > `{$CPU.UTIL.CRIT}` |
| Load average elevato | WARNING | Load/vCPU > `{$LOAD.AVG.MAX}` |
| RAM utilizzo elevato | WARNING | RAM util > `{$MEMORY.UTIL.MAX}` |
| Swap quasi esaurito | WARNING | Swap libera < `{$SWAP.PFREE.MIN}`% |
| Interfaccia DOWN | AVERAGE | Status = DOWN |
| Errori interfaccia | WARNING | Errors/s > `{$IF.ERRORS.WARN}` |
| Filesystem quasi pieno (warning) | WARNING | Usage > `{$VFS.FS.UTIL.WARN}`% |
| Filesystem quasi pieno (critico) | HIGH | Usage > `{$VFS.FS.UTIL.CRIT}`% |
| Inode quasi esauriti | WARNING | Inode liberi < `{$VFS.FS.INODE.PFREE.MIN}`% |
| SNMP non raggiungibile | HIGH | No SNMP response |

### Prerequisiti specifici

- SNMP v2c **abilitato** sull'UAG (Photon OS — Net-SNMP)
- Accesso **UDP 161** dal Zabbix Server/Proxy all'UAG
- **Configurare SNMP sull'UAG** (se non già fatto):

```bash
# Su Photon OS (UAG)
tdnf install net-snmp -y
cat > /etc/snmp/snmpd.conf << 'EOF'
rocommunity public 0.0.0.0/0
syslocation UAG
syscontact admin@example.com
EOF
systemctl enable snmpd
systemctl start snmpd
```

### Configurazione host Zabbix

1. Creare un host con **interfaccia SNMP** puntando all'IP dell'UAG, porta **161**, versione **SNMPv2**
2. Collegare il template
3. Impostare le macro nella scheda **Macros** dell'host (almeno `{$SNMP_COMMUNITY}`)

### Macro

| Macro | Default | Tipo | Descrizione |
|-------|---------|------|-------------|
| `{$SNMP_COMMUNITY}` | `public` | Text | Community string SNMP v2c |
| `{$CPU.UTIL.CRIT}` | `90` | Text | Soglia critica utilizzo CPU (%) |
| `{$MEMORY.UTIL.MAX}` | `90` | Text | Soglia warning utilizzo RAM (%) |
| `{$SWAP.PFREE.MIN}` | `20` | Text | Soglia minima swap libera (%) |
| `{$LOAD.AVG.MAX}` | `1.5` | Text | Moltiplicatore load average 5min / vCPU |
| `{$IF.ERRORS.WARN}` | `2` | Text | Soglia errori/s su interfaccia di rete |
| `{$IF.UTIL.MAX}` | `90` | Text | Soglia utilizzo banda interfaccia (%) |
| `{$VFS.FS.UTIL.WARN}` | `80` | Text | Soglia warning utilizzo filesystem (%) |
| `{$VFS.FS.UTIL.CRIT}` | `90` | Text | Soglia critica utilizzo filesystem (%) |
| `{$VFS.FS.INODE.PFREE.MIN}` | `20` | Text | Soglia minima inode liberi (%) |
| `{$NET.IF.IFNAME.MATCHES}` | `^eth` | Text | Regex interfacce da includere nella LLD |
| `{$NET.IF.IFNAME.NOT_MATCHES}` | `^lo` | Text | Regex interfacce da escludere dalla LLD |
| `{$VFS.FS.FSNAME.NOT_MATCHES}` | `^(/dev\|/sys\|/run\|/proc\|tmpfs\|...)` | Text | Regex filesystem da escludere dalla LLD |

---

## Compatibilità

| Template | Zabbix | Horizon | Note |
|----------|--------|---------|------|
| Horizon REST API | 7.4+ | 2503+ | API path verificati su Swagger 2503 |
| Horizon REST API + Graphs | 7.4+ | 2503+ | |
| UAG SNMP | 7.4+ | Qualsiasi | Richiede Net-SNMP su Photon OS |

---

## Note e limitazioni

- **Horizon REST API:** ogni item master esegue un login e un logout separati. Con 14 endpoint e intervalli da 1-5 minuti si generano circa 30-50 autenticazioni/minuto sul CS. L'account di servizio non deve avere restrizioni sul numero di sessioni simultanee.

- **Certificati self-signed:** se il CS usa certificati non firmati da una CA trusted, il metodo `setSSLVerifyPeer(false)` potrebbe non essere disponibile sullo script item a seconda della build di Zabbix. In questo caso è necessario importare il certificato nel trust store del Zabbix Server/Proxy (vedi sezione Prerequisiti del Template 1).

- **VDI vs RDSH:** `/monitor/v1/machines/count-metrics` conta solo le macchine desktop virtuali (VDI). In ambienti puramente RDSH questo item restituisce sempre 0 — è normale. I conteggi degli RDS server sono invece monitorati separatamente tramite `/monitor/v1/rds-servers/count-metrics` (item `horizon.rds_metrics.*`) con trigger e grafico dedicati.

- **Endpoint 404:** se un endpoint non esiste nell'ambiente (es. nessuna Farm RDSH, nessun True SSO), il relativo item restituisce `[]` o `{}` silenziosamente senza generare errori. La LLD scoprirà semplicemente zero elementi.

- **Versioni API:** i path sono basati sullo Swagger del CS Horizon 2503. In versioni precedenti alcuni endpoint potrebbero non esistere (es. `/monitor/v5/gateways` richiede una versione recente). In caso di 404 su endpoint critici, verificare la versione disponibile su `https://<CS>/rest/swagger-ui/index.html` e aggiornare la macro dell'URL o modificare il path nello script item.

- **UAG SNMP:** questo template monitora solo le risorse di sistema del nodo UAG. Le metriche funzionali del gateway (sessioni, connessioni, status) sono già incluse nel Template 1/2 tramite Horizon REST API (`/monitor/v5/gateways`).

---

## Licenza

MIT — libero utilizzo, modifica e distribuzione con attribuzione.
