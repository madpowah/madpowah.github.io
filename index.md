---
layout: default
title: cloud's Blog
---
# CVE-2026-45257 (BUMSRAKETE) : Écriture dans le cache de pages FreeBSD via sendfile + kTLS
11/06/2026 - **CVE-2026-45257 (BUMSRAKETE)** est une vulnérabilité d'élévation de privilèges locaux dans le noyau FreeBSD découverte par le chercheur « Bumsrakete ». En enchaînant `sendfile(2)`, kTLS RX et le decrypt AES-GCM logiciel, tout utilisateur non privilégié peut écrire des données arbitraires dans le cache de pages pour obtenir un shell root en ~1,5 secondes. Affecte FreeBSD 13.0 à 15.0. [Lire][bumsrakete]

[bumsrakete]: https://madpowah.github.io/2026/06/11/bumsrakete-cve-2026-45257.html

---

# Navigateur Tesla : contournement des restrictions vidéo en mode conduite
03/06/2026 - Contournement des restrictions vidéo du navigateur Tesla en mode conduite via MPEG1 + JSMpeg (WebGL) et WebSocket binaire — de 15 FPS/3 Mbps à 30 FPS/0.8 Mbps. Exploration technique des API bloquées, des pistes échouées et de la solution finale. [Lire][tesla-video]

[tesla-video]: https://madpowah.github.io/2026/06/03/tesla-browser-video-bypass.html

---

# CVE-2026-45585 (YellowKey) : Bypass BitLocker Windows par Transaction NTFS
28/05/2026 - **CVE-2026-45585 (YellowKey)** est un bypass de BitLocker Windows permettant à un attaquant avec accès physique de contourner le chiffrement et d'accéder aux données sans mot de passe, clé de récupération, ni outil spécialisé, en abusant de Transactional NTFS (TxF) et du boot WinRE. [Lire][yellowkey]

[yellowkey]: https://madpowah.github.io/2026/05/28/cve-2026-45585-yellowkey-bitlocker-bypass.html

---

# CVE-2026-46333 (ssh-keysign-pwn) : Vol de clés SSH hôte via le noyau Linux
16/05/2026 - **CVE-2026-46333** est une vulnérabilité du noyau Linux permettant à tout utilisateur local non privilégié de voler les clés privées SSH de l'hôte et `/etc/shadow` via un contournement `mm-NULL` dans `__ptrace_may_access()`. [Lire][ssh-keysign-pwn]

[ssh-keysign-pwn]: https://madpowah.github.io/2026/05/16/cve-2026-46333-ssh-keysign-pwn.html

---

# Opencode Agent Manager
14/05/2026 - Gérez vos agents et skills OpenCode depuis une interface web : dashboard, CRUD agents, skills, import/export, mode projet. [Lire][oam]

[oam]: https://madpowah.github.io/2026/05/14/opencode-agent-manager.html

---

# Dirty Frag : LPE universel Linux - CVE-2026-43284 / CVE-2026-43500
09/05/2026 - **Dirty Frag** est une classe de vulnérabilités qui chaîne deux failles d'écriture dans le cache de pages Linux (xfrm-ESP + RxRPC) pour obtenir root sur toutes les distributions majeures. [Lire][dirtyfrag]

[dirtyfrag]: https://madpowah.github.io/2026/05/09/dirtyfrag-cve-2026-43284.html

---

# CVE-2026-23918 : Double-Free dans le module HTTP/2 d'Apache HTTPD
07/05/2026 - **CVE-2026-23918** est une vulnérabilité de type double-free dans le module HTTP/2 d'Apache HTTP Server 2.4.66, permettant un DoS fiable et une RCE théorique. [Lire][cve-2026-23918]

[cve-2026-23918]: https://madpowah.github.io/2026/05/07/cve-2026-23918-apache-http2-double-free.html

---

# Copy Fail exploit - CVE-2026-31431
01/05/2026 - **Copy Fail (CVE-2026-31431)** est une vulnérabilité critique de type "container escape" qui permet l'élévation de privilèges depuis un utilisateur standard vers root. [Lire][copyfail]

[copyfail]: https://madpowah.github.io/2026/05/01/copy-fail-exploit.html

---

# Opencode / Hextrike / Ollama
30/04/2026 - Un article pour détailler la mise en oeuvre d'une solution de pentest via IA en full local avec le combo `Opencode / Hextrike / Ollama`. [Lire][hexstrike-opencode]

[hexstrike-opencode]: https://madpowah.github.io/2026/04/30/opencode-hexstrike-ollama.html

