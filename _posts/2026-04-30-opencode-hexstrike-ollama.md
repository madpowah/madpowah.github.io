---
layout: post
title:  "Opencode / Hextrike / Ollama"
date:   2026-04-30 22:54:36 +0200
categories: security ai
permalink: /2026/04/30/opencode-hexstrike-ollama.html
---
Un article pour détailler la mise en oeuvre d'une solution de pentest via IA en full local avec le combo `Opencode / Hextrike / Ollama`.

L'architecture que j'utilise est la suivante :
[Host W11 + WSL + Ollama + MCP server] <--> [VM Kali + Hexstrike + pentest tools]

[Hextrike][hexstrike-github] est un framework IA oriennté pentest donnant les capacités à un LLM d'exécuter les outils de pentests nécessaires à un attaquant. J'ai choisi de l'installer dans une VM Kali qui possède uen bonne partie de ces outils et permet d'en ajouter facilement. 
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

Ajouter donc d'abord les outils que vous utilisez ou qui vous semblent pertinents pour ce que vous souhaitez faire puis installer hexstrike via les commandes suivantes :

```
# 1. Clone the repository
git clone https://github.com/0x4m4/hexstrike-ai.git
cd hexstrike-ai

# 2. Create virtual environment
python3 -m venv hexstrike-env
source hexstrike-env/bin/activate  # Linux/Mac
# hexstrike-env\Scripts\activate   # Windows

# 3. Install Python dependencies
pip3 install -r requirements.txt

```

Ma VM est configurée en NAT donc il faut faire une redirection de port dans la conf Virtualbox de la VM pour rendre le port 8888 (hexstrike server) accessible depuis le WSL du host.
![redirection port](/assets/images/2026-04-30-redir-port.jpg)

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[hexstrike-github]: https://github.com/0x4m4/hexstrike-ai
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
 