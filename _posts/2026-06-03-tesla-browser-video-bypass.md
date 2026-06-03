---
layout: post
title:  "Navigateur Tesla : contournement des restrictions vidéo en mode conduite"
date:   2026-06-03 14:00:00 +0200
author: cloud
categories: security hardware
tags: security hardware tesla browser webcodecs websocket mpeg1 jsmpeg streaming
permalink: /2026/06/03/tesla-browser-video-bypass.html
---

**Le navigateur Tesla** (basé sur QtWebEngine, un Chromium embarqué) est accessible en mode Park comme en mode Drive, mais avec une contrainte majeure : dès que le véhicule passe en mode conduite, Tesla verrouille certaines API du navigateur — la balise `<video>` est mise en pause, le micro est inaccessible. Cet article détaille la mise en oeuvre d'une solution de contournement de ces restrictions, depuis le diagnostic initial jusqu'à un flux stable à 30 FPS via MPEG1 + JSMpeg, en passant par les multiples pistes explorées et les optimisations progressives.

Le projet est disponible sur [GitHub — tesla-video-drive](https://github.com/madpowah/tesla-video-drive).

***Disclaimer** : Ce qui est présenté ici vise uniquement à démontrer les limites techniques du blocage mis en place par Tesla sur le navigateur embarqué. Il ne s'agit en aucun cas d'encourager quiconque à regarder des vidéos en conduisant. Regarder un écran en conduisant est dangereux et illégal dans la plupart des pays. Le conducteur reste seul responsable de son attention sur la route. Ce projet est une exploration technique, rien de plus.*

---

## TL;DR

| Champ | Valeur |
|---|---|
| **Cible** | Navigateur Tesla (QtWebEngine / Chromium embarqué) |
| **Restriction** | `<video>` pausé en Drive, `getUserMedia` bloqué |
| **Solution** | MPEG1 via JSMpeg (WebGL) + WebSocket binaire |
| **Résultat** | 30 FPS, ~0.8 Mbps, compatible Twitch et YouTube |
| **Micro** | Définitivement inaccessible (blocage à la compilation) |
| **Projet** | [github.com/madpowah/tesla-video-drive](https://github.com/madpowah/tesla-video-drive) |

---

## 1. Contexte : le navigateur Tesla et ses verrous

Le navigateur Tesla est basé sur **QtWebEngine** (Chromium embarqué). Il est accessible en mode Park comme en mode Drive, mais avec une contrainte de taille : dès que la voiture passe en mode conduite, Tesla verrouille certaines API du navigateur :

- **`<video>`** : mis en pause immédiatement — la dernière image reste figée, l'audio continue
- **`getUserMedia`** : retourne une erreur — le micro est inaccessible
- Certaines fonctionnalités modernes de Chromium sont absentes ou limitées

La question était : jusqu'où peut-on aller pour contourner ces limitations, sans modifier le véhicule, en utilisant uniquement ce que le navigateur met à disposition ?

---

## 2. Phase 1 : Le Probe — cartographier les capacités

Avant toute chose, il fallait cartographier ce dont le navigateur Tesla est capable. J'ai créé une page de diagnostic (`/probe/`) qui teste méthodiquement chaque API :

- User-Agent
- WebSocket
- WebGL / WebGL2
- Canvas 2D
- `createImageBitmap`
- MediaSource Extensions
- VideoDecoder (WebCodecs)
- WebAssembly
- Formats audio/video supportés
- Codecs H.264, H.265, VP9, AV1
- WebRTC
- Service Worker
- Performance API

Résultats clés :

| API | Disponible |
|-----|:----------:|
| WebSocket | ✅ |
| Canvas 2D | ✅ |
| WebGL | ✅ |
| `createImageBitmap` | ✅ |
| MediaSource (MSE) | ⚠️ Oui, mais codecs limités |
| WebCodecs | ❌ |
| WebRTC | ❌ |
| `getUserMedia` | ❌ (bloqué au niveau compilation) |
| WebAssembly | ✅ |

Cette phase de diagnostic a été cruciale : elle a immédiatement éliminé plusieurs pistes (WebCodecs, WebRTC) et orienté les efforts vers les solutions viables — principalement Canvas, WebGL et WebSocket.

---

## 3. Phase 2 : Les tentatives de contournement vidéo

### 3.1 `<video>` natif (échec)

Test le plus simple : balise `<video>` avec une source MP4 directe.

```html
<video src="video.mp4" controls></video>
```

✅ En Park : lecture normale
❌ En Drive : **pause immédiate** — la dernière image reste affichée, l'audio continue. Tesla appelle `video.pause()` au niveau système, impossible de le contrer avec `play()`.

### 3.2 Canvas + `<video>` (échec)

Idée : plutôt que d'afficher directement la `<video>`, la dessiner sur un `<canvas>` via `ctx.drawImage(video, ...)`.

```javascript
var video = document.createElement('video');
video.src = 'video.mp4';
video.play();
function draw() {
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    requestAnimationFrame(draw);
}
```

✅ En Park : fonctionne
❌ En Drive : **canvas noir** — quand la source vidéo est en pause, `drawImage(video)` ne produit rien.

### 3.3 MJPEG (échec)

Le navigateur Tesla ne supporte pas le MJPEG (Motion JPEG). Le support a été retiré de Chromium à partir de la version 82.

### 3.4 Picture-in-Picture (échec)

Tentative de sortir la vidéo du flux principal via PiP. Même problème : la source vidéo est en pause, donc le flux PiP est figé.

### 3.5 MediaSource Extensions + MPEG1 (échec)

Tentative d'utiliser MSE pour envoyer des segments MPEG-TS directement à une balise `<video>`.

```javascript
var ms = new MediaSource();
video.src = URL.createObjectURL(ms);
ms.addEventListener('sourceopen', function() {
    var sb = ms.addSourceBuffer('video/mp2t');
    // envoyer des segments MPEG-TS via WebSocket
});
```

❌ **MSE ne supporte pas MPEG1 sur ce navigateur** — le QtWebEngine de Tesla ne reconnaît pas le format MPEG-TS dans MSE.

---

## 4. Phase 3 : La solution qui marche — Canvas + frames JPEG

### 4.1 Le principe

Puisque `<video>` est bloqué et que `drawImage(video)` produit un canvas noir, il fallait une source que Tesla ne peut pas intercepter. La solution : **des images individuelles**.

Le serveur extrait les frames d'une vidéo via ffmpeg :

```bash
ffmpeg -i video.mp4 -c:v mjpeg -q:v 5 -vf fps=15,scale=640:360 -f image2pipe -
```

Le client reçoit chaque frame comme une image JPEG, la charge via `new Image()`, et la dessine sur le canvas :

```javascript
var img = new Image();
img.onload = function() {
    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
    URL.revokeObjectURL(url);
};
img.src = url; // Blob URL depuis WebSocket
```

✅ **Ça marche en Drive !** — Le canvas reçoit des images individuelles, pas un flux vidéo. Tesla ne peut pas les intercepter.

**Première version** : 15 FPS, 640×360, qualité moyenne.

### 4.2 Passage en WebSocket temps réel

Plutôt que d'extraire toutes les frames d'avance, le serveur lance ffmpeg en temps réel et envoie chaque frame dès qu'elle est produite via WebSocket :

```
ffmpeg → stdout (JPEG) → WebSocket binaire → new Image() → canvas.drawImage()
```

**Gain** : passage de 15 à 25 FPS.

---

## 5. Phase 4 : Optimisations progressives

### 5.1 WebP (5× plus efficace)

Première optimisation : remplacer le JPEG par du WebP. À qualité équivalente, le WebP est ~5× plus petit (6.8 KB vs 35 KB par frame à 854×480).

```bash
# Avant
-c:v mjpeg -q:v 3

# Après
-c:v libwebp -lossless 0 -quality 80
```

**Gain** : débit divisé par 5 (5.5 Mbps → 1.1 Mbps), même FPS, même qualité visuelle.

### 5.2 Compression WebSocket

Activation de la compression `permessage-deflate` sur le serveur WebSocket :

```javascript
const wss = new WebSocketServer({
    noServer: true,
    perMessageDeflate: true  // ← compression activée
});
```

**Gain** : -30% de bande passante supplémentaire, **zéro changement côté client**.

### 5.3 MPEG1 + JSMpeg (le grand saut)

Le problème du JPEG/WebP : chaque frame est une image complète. Pas de compression inter-trames. Pour un flux vidéo, c'est extrêmement inefficace.

Solution : encoder en MPEG1 côté serveur (qui supporte les frames P, beaucoup plus petites) et décoder côté client avec **JSMpeg** — un décodeur MPEG1 pur JavaScript qui utilise WebGL pour le rendu.

```bash
# Encodage MPEG1 (serveur)
ffmpeg -i $STREAM_URL \
  -c:v mpeg1video -q:v 5 -b:v 1500k -bf 0 \
  -vf fps=30,scale=1280:720 \
  -f mpegts -
```

```javascript
// Décodage JSMpeg (client)
var player = new JSMpeg.Player(wsUrl, {
    canvas: canvas,
    audio: false,
    streaming: true,
    maxBufferSize: 1024 * 1024
});
```

**Gain spectaculaire** :

| Métrique | JPEG/WebP | MPEG1 |
|----------|:---------:|:-----:|
| FPS | 25 | **30** |
| Débit | ~1.1 Mbps | **~0.8 Mbps** |
| Taille par frame | 6.8 KB | **~0.5 KB** (frame P) |
| Compression inter-trames | ❌ | ✅ |
| GPU | Non | WebGL |

### 5.4 Gestion des coupures pub

Avec Twitch, les flux live changent de playlist HLS pendant les pauses publicitaires. Sans précautions, ffmpeg décroche et le flux s'arrête. Solution : les options de reconnexion ffmpeg.

```bash
-reconnect 1 -reconnect_at_eof 1 \
-reconnect_streamed 1 -reconnect_delay_max 30
```

Ajoutées à tous les endpoints (audio, vidéo, polling). Le flux survit maintenant aux pubs.

### 5.5 Synchro audio

L'audio est servie par un endpoint HTTP séparé (`/api/live-audio`), ce qui ajoute ~5 secondes de latence par rapport à la vidéo. Solution : la vidéo attend que l'audio soit effectivement en train de jouer avant de démarrer.

```javascript
var syncTimer = setInterval(function() {
    if (audio.readyState >= 2 && audio.currentTime > 0) {
        clearInterval(syncTimer);
        // Lancer JSMpeg maintenant
        player = new JSMpeg.Player(wsUrl, { ... });
    }
}, 200);
```

### 5.6 Piège WebGL : conflit de contexte

JSMpeg utilise WebGL pour le rendu. Si un contexte 2D est pré-alloué sur le même canvas, WebGL ne peut pas s'initialiser. Solution : créer le contexte 2D uniquement quand nécessaire (lazy), et ne jamais le pré-allouer avant JSMpeg.

---

## 6. Architecture finale

```
                      ┌──────────────────────┐
                      │    Navigateur Tesla   │
                      │    (QtWebEngine)       │
                      │                        │
                      │  ┌──────────────────┐  │
                      │  │  JSMpeg (WebGL)   │  │
                      │  │  ↓ canvas 30fps   │  │
                      │  └────────┬─────────┘  │
                      │           │             │
                      │  ┌────────┴─────────┐  │
                      │  │  <audio> MP3      │  │
                      │  └────────┬─────────┘  │
                      └───────────┼────────────┘
                                  │ WSS / HTTPS
                                  │
                     ┌────────────┴────────────┐
                     │      Apache Proxy         │
                     │  app.mydomain.com:443     │
                     └────────────┬──────────────┘
                                  │
                     ┌────────────┴────────────┐
                     │   Node.js (Express)      │
                     │   serveur:8742            │
                     │                           │
                     │  ┌─────────────────────┐ │
                     │  │ yt-dlp → URL        │ │
                     │  │ ffmpeg → MPEG1-TS   │─┼─→ WSS
                     │  │ ffmpeg → MP3 audio  │─┼─→ HTTPS
                     │  └─────────────────────┘ │
                     └──────────────────────────┘
```

---

## 7. Bilan des optimisations

| Étape | FPS | Débit | Qualité |
|-------|:---:|:-----:|:-------:|
| JPEG 15 FPS | 15 | ~3 Mbps | Moyenne |
| JPEG 25 FPS | 25 | ~5.5 Mbps | Correcte |
| WebP 25 FPS | 25 | ~1.4 Mbps | Bonne |
| WebP + compression | 25 | ~1.0 Mbps | Bonne |
| **MPEG1 30 FPS** | **30** | **~0.8 Mbps** | **Très bonne** |

De 15 FPS à 3 Mbps en JPEG basse qualité, on arrive à 30 FPS à 0.8 Mbps en MPEG1 — un gain de ~7× en bande passante et un doublement du framerate.

---

## 8. Pages déployées

| URL | Rôle |
|-----|------|
| `/twitch-client/` | **Client Twitch complet** — JSMpeg 30fps, OAuth, synchro audio |
| `/youtube/` | Client YouTube — recherche, stream 25fps en Drive |
| `/gps-map/` | Navigation GPS — Superchargeurs, radars, itinéraire OSRM |
| `/video-frames/` | VOD — lecture de vidéos par frames extraites |
| `/probe/` | Diagnostic du navigateur Tesla |

---

## 9. Ce qui n'a pas marché

| Technique | Raison |
|-----------|--------|
| `<video>` natif | Tesla pausé au niveau système |
| Canvas + `drawImage(video)` | Source vidéo pausée → canvas noir |
| MJPEG | Chromium 82+ ne supporte plus |
| Picture-in-Picture | Même source vidéo pausée |
| WebCodecs (`VideoDecoder`) | Pas disponible dans ce Chromium |
| MSE + MPEG1 | QtWebEngine ne supporte pas le format |
| `getUserMedia` | Bloqué à la compilation de Chromium |
| Audio MP2 dans JSMpeg | CPU Tesla surchargé (vidéo + audio) |

---

## 10. Le micro : un blocage définitif

Contrairement à la vidéo, le micro est bloqué au niveau de la compilation de Chromium. Le flag `--disable-audio-input` est passé à QtWebEngine, ce qui rend `getUserMedia` totalement inopérant — pas de contournement JavaScript possible.

La solution de contournement déployée utilise un **relay audio via WebSocket** : le téléphone de l'utilisateur envoie l'audio au serveur Node.js, qui le relaie au navigateur Tesla via un endpoint dédié (`/ws/audio-relay`). Ce n'est pas élégant, mais c'est fonctionnel.

---

## Conclusion technique

Le navigateur Tesla, malgré ses limitations, offre assez de capacités (WebSocket, Canvas, WebGL, WebAssembly) pour permettre un contournement élégant de la restriction vidéo en mode conduite. La clé a été de sortir du paradigme `<video>` pour passer à un flux d'images individuelles décodées côté client.

Le passage au MPEG1 via JSMpeg a permis un gain significatif en FPS (25 → 30) et en bande passante (5×), tout en maintenant une compatibilité totale avec le navigateur Tesla.

Le micro, en revanche, reste définitivement inaccessible — le blocage est au niveau compilateur Chromium, pas contournable depuis le JavaScript.

Le code complet est disponible sur [GitHub — tesla-video-drive](https://github.com/madpowah/tesla-video-drive).

---

Have fun.