---
layout: post
title:  "Opencode / Hextrike / Ollama"
date:   2026-04-30 22:54:36 +0200
author: cloud
categories: security ai
tags: security ai
permalink: /2026/04/30/opencode-hexstrike-ollama.html
---
Un article pour détailler la mise en oeuvre d'une solution de pentest via IA en full local avec le combo `Opencode / Hexstrike / Ollama`.

# Architecture
L'architecture que j'utilise est la suivante :
```
[Host W11 + WSL + Ollama + MCP server] <--> [VM Kali + Hexstrike + pentest tools]
```


***


# Hexstrike Server
[Hexstrike][hexstrike-github] est un framework IA orienté pentest donnant les capacités à un LLM d'exécuter les outils de pentests nécessaires à un attaquant. J'ai choisi de l'installer dans une VM Kali qui possède uen bonne partie de ces outils et permet d'en ajouter facilement. Il est composé d'un serveur accessible depuis une API et d'un MCP qui va permettre de faire le lien entre les commandes souhaitées par le LLM et le serveur hexstrike qui va les exécuter.
La liste des outils exécutables par hexstrike sont les suivants (src hexstrike github):

```
# Network & Reconnaissance
nmap masscan rustscan amass subfinder nuclei fierce dnsenum
autorecon theharvester responder netexec enum4linux-ng

# Web Application Security
gobuster feroxbuster dirsearch ffuf dirb httpx katana
nikto sqlmap wpscan arjun paramspider dalfox wafw00f

# Password & Authentication
hydra john hashcat medusa patator crackmapexec
evil-winrm hash-identifier ophcrack

# Binary Analysis & Reverse Engineering
gdb radare2 binwalk ghidra checksec strings objdump
volatility3 foremost steghide exiftool
```

Ajoutez donc d'abord les outils que vous utilisez ou qui vous semblent pertinents pour ce que vous souhaitez faire puis installer et exécuter hexstrike via les commandes suivantes :

```bash
# 1. Clone the repository
git clone https://github.com/0x4m4/hexstrike-ai.git
cd hexstrike-ai

# 2. Create virtual environment
python3 -m venv hexstrike-env
source hexstrike-env/bin/activate 

# 3. Install Python dependencies
pip3 install -r requirements.txt
python3 hexstrike_server.py

```

Ma VM est configurée en NAT donc il faut faire une redirection de port dans la conf Virtualbox de la VM pour rendre le port 8888 (hexstrike server) accessible depuis le WSL du host.

![redirection port](/assets/images/2026-04-30-redir-port.jpg)


***


# Ollama
Ollama est installé sur le host W11 et fonctionne en utilisant ma carte graphique. J'ai constaté des soucis avec beaucoup de modèles pour détecter les tools. cela serait dû à la taille du context configuré à 4096. Du coup j'ai trouvé ce workaround qui fonctionne pour ma part et qui consiste à augmenter la taille de ce contexte. Perso j'utilise Qwen3.5 en 9b ou 27b selon mon humeur. C'est le modèle local que je trouve le plus pertinent. J'ai trouvé également deepseek v3.2 via Ollama cloud très efficace également.
Les commandes pour augmenter le context depuis le Powershell

```bash
PS> ollama run qwen3.5:9b
>>> /set parameter num_ctx 100000
Set parameter 'num_ctx' to '100000'
>>> /save qwen3:8b-16k
Created new model 'qwen3.5:9b-100k'
>>> /bye
```


***


# Hexstrike MCP
On va maintenant installer le MCP hexstrike sur le Host WSL. C'est lui qui sera appelé via Opencode pour lancer des commandes via le Hexstrike de la Kali.
On tape donc sur le WSL.

```bash
# 1. Clone the repository
git clone https://github.com/0x4m4/hexstrike-ai.git
cd hexstrike-ai

# 2. Create virtual environment
python3 -m venv hexstrike-env
source hexstrike-env/bin/activate  #

# 3. Install Python dependencies
pip3 install mcp requests
```


***


# Opencode
Il reste maintenant à installer Opencode sur le WSL. Rien de plus simple :

```bash
curl -fsSL https://opencode.ai/install | bash
```

On va ensuite configurer [Opencode][opencode] pour qu'il utilise le MCP d'Hexstrike et qu'il propose les modèles Ollama que l'on souhaite. Pour trouver l'IP pour pouvoir contacter ollama et hexstrike, le plus simple est de taper `route -n` depuis un shell du WSL et de regarder l'IP de la default gateway.

```json
# Fichier: ~/.config/opencode/config.json

{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "hexstrike-ai": {
      "type": "local",
      "command" : ["/path/to/hexstrike-ai/hexstrike-env/bin/python3", "/path/to/hexstrike-ai/hexstrike_mcp.py", "--server", "http://<IP_DEFAULT_GW_WSL>:8888"],
      "enabled": true
    }
  },
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://<IP_DEFAULT_GW_WSL>:11434/v1"

      },
      "models": {
        "qwen3.5:27b-16k": {
                "name": "Qwen3.5-27b",
                "tools": true
        },
        "qwen3.5:9b-100k": {
                "name": "Qwen3.5-9b-100k",
                "tools": true
        },
      }
    }
  }
}
```

Et voilà, il n'y a plus qu'à lancer opencode en tapant opencode dans le WSL et si tout va bien vous devriez voir dans Opencode `MCP Hexstrike-ai Connected` et la console du serveur Hexstrike sur la Kali devrait également s'activer.

Il ne reste plus qu'à discuter via Opencode et lui indiquer votre cible et ce que vous souhaitez faire.

Have fun.

[hexstrike-github]: https://github.com/0x4m4/hexstrike-ai
[opencode]:   https://opencode.ai/

 