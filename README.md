# DVWA-WazuhAgent

A small security monitoring PoC that connects **DVWA**, **Wazuh**, and a **local LLM** to turn security alerts into a short SOC-style analysis.

The project was built to explore how AI can assist with the first step of security alert triage without sending logs to an external AI service.

## How it works

<img width="1757" height="949" alt="shapes at 26-10-08 02 28 58" src="https://github.com/user-attachments/assets/d60b1b51-8e69-443c-a417-b11c4255da72" />


## What I built

**Monitored Endpoint**
- DVWA runs in Docker as the vulnerable application.
- Wazuh Agent monitors the Apache access log.
- Optional **Suricata** and **Zeek** integrations provide network security logs.

**Wazuh Server**
- Wazuh Manager, Indexer, and Dashboard run with Docker Compose.
- A custom Wazuh integration forwards JSON alerts to the AI bridge.

**AI Bridge**
- Small **FastAPI** service that receives Wazuh alerts.
- Extracts the important alert fields and sends them to **Ollama**.
- Prompts the model to separate observed evidence from assumptions.
- Returns a concise report containing severity, evidence, possible impact, recommended actions, and confidence.

**Display Output**
- Lightweight web service that acts as a mock notification destination.
- It is only for the PoC and is **not a real LINE/notification integration**.

## Example workflow

1. Perform an attack against DVWA.
2. Apache records the request.
3. Wazuh Agent collects the log.
4. Wazuh analyzes the event and generates an alert.
5. Alerts at level **5+** are sent to the custom AI bridge.
6. The AI bridge asks the local Ollama model to analyze the alert.
7. The analysis is shown in the display service.

## Tech Stack

**Security:** Wazuh · DVWA · Suricata · Zeek  
**AI:** Ollama · Qwen  
**Backend:** Python · FastAPI  
**Infrastructure:** Docker · Docker Compose  
**Monitoring:** Wazuh Dashboard · OpenSearch

## Running the PoC

The project is designed to be deployed as several components/VMs.

### 1. Wazuh Server

```bash
cd wazuh-server

docker compose -f generate-indexer-certs.yml run --rm generator
docker compose up -d
```

### 2. Monitored Endpoint

Copy the environment template and configure the Wazuh Manager address:

```bash
cd monitored-endpoint

cp .env.template .env
```

Then start DVWA:

```bash
cd docker
docker compose up -d
```

Install/configure the Wazuh Agent:

```bash
cd ../wazuh-agent
sudo ./install.sh
```

Suricata and Zeek can be installed from their respective directories when needed.

### 3. AI Bridge

Configure the Ollama endpoint:

```bash
cd wazuh-server

cp .env.template .env
```

Set:

```env
OLLAMA_BASE_URL=http://YOUR_OLLAMA_VM_IP:11434
OLLAMA_MODEL=qwen2.5:0.5b
DISPLAY_OUTPUT_URL=http://YOUR_DISPLAY_VM_IP:9000/messages
```

The AI bridge is started together with the Wazuh server Compose environment.

## Notes

This is a **proof of concept**, not a production SOC platform.
