# Fiche de trace — Analyse de trafic mobile en laboratoire

---

## 1. Périmètre

| Champ | Valeur |
|---|---|
| **Environnement** | Émulateur Android de labo (Android Studio AVD) |
| **Cible autorisée** | `testphp.vulnweb.com` (cible d'entraînement publique) |
| **Objectif** | Valider la chaîne de capture HTTP/HTTPS via proxy d'interception |
| **Hors périmètre** | Tout autre domaine, toute modification de requête |

---

## 2. Configuration

| Paramètre | Valeur |
|---|---|
| **Outil** | Burp Suite Community Edition v2024.5.5 |
| **IP_HOTE** | `192.168.96.137` |
| **PORT_PROXY** | `8080` |
| **Listener** | `*:8080` (All interfaces) |
| **OS hôte proxy** | Kali Linux (VM VMware) |
| **OS émulateur** | Android 8.1.0 x86 (AVD Android Studio) |


<img width="1225" height="763" alt="image" src="https://github.com/user-attachments/assets/8f1db164-ddf7-4ea7-949f-3ea4dc7e843f" />


---

## 3. Preuves

### 3.1 Capture de l'historique HTTP
<img width="1600" height="275" alt="image" src="https://github.com/user-attachments/assets/f57861f8-d372-4f3c-9678-98f4b016d237" />

---

### 3.2 Détails d'une requête observée

<img width="1522" height="586" alt="image" src="https://github.com/user-attachments/assets/bb19ba84-3478-46a9-84be-9a554edb03e1" />


**Exemple de structure observée :**

```
GET /index.php HTTP/1.1
Host: testphp.vulnweb.com
User-Agent: Mozilla/5.0 (Linux; Android 8.1.0; ...)
                          Chrome/69.0.3497.100 Mobile Safari/537.36
Accept: text/html,application/xhtml+xml,...
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

---

## 4. Analyse

### 4.1 Données observées en transit

| Type de donnée | Présence | Observation |
|---|---|---|
| Paramètres en URL | À vérifier | Potentiellement visibles en clair (HTTP) |
| Cookies de session | À vérifier | Attributs `Secure` / `HttpOnly` à contrôler |
| Tokens en URL | À vérifier | Risque d'exposition dans les logs serveur |
| En-têtes de sécurité | À vérifier | Présence de `X-Frame-Options`, `CSP`, etc. |

### 4.2 Risques potentiels identifiés

| Risque | Niveau | Justification |
|---|---|---|
| Trafic HTTP non chiffré | ⚠️ Moyen | Données lisibles sans déchiffrement |
| Absence d'attribut `Secure` sur cookies | ⚠️ Moyen | Cookie transmissible sur HTTP |
| Tokens/paramètres sensibles en URL | ⚠️ Moyen | Exposés dans l'historique proxy et logs |
| Trafic système Android visible | ℹ️ Info | Requêtes de mise à jour interceptées |

> **Distinction importante :**
> - **Observé** : requêtes HTTP visibles en clair dans Burp
> - **Supposé** : risques déduits de l'absence de mécanismes de protection
> - **Non confirmé** : exploitation réelle — hors périmètre de ce lab

---

## 5. Interception contrôlée (mode actif)

### 5.1 Procédure appliquée

1. Activation de l'interception dans Burp : **Proxy → Intercept → "Intercept is on"**
2. Rafraîchissement d'une page dans le navigateur de l'émulateur
3. Observation de la requête bloquée dans l'onglet **Intercept**
4. Désactivation immédiate : **"Intercept is off"**

### 5.2 Requête interceptée observée

<img width="1342" height="756" alt="image" src="https://github.com/user-attachments/assets/dee8ae0e-42dd-40fa-9760-5c140eace082" />

<img width="1600" height="607" alt="image" src="https://github.com/user-attachments/assets/2f45b712-12a0-4fe7-8a8a-0c8a33e92ffd" />


**Exemple observé lors du test :**

```
POST /service/update2?cup2key=...&cup2hreq=... HTTP/1.1
Host: update.googleapis.com
Content-Length: 830
X-Goog-Update-AppId: gcmjkmgdlgnkkcocmoeiminaijmmjnii,...
X-Goog-Update-Interactivity: bg
Content-Type: application/xml
User-Agent: Mozilla/5.0 (Linux; Android 8.1.0; Android SDK built for x86)
            AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

### 5.3 Différence passif / actif

| Mode | Description | Risque si mal utilisé |
|---|---|---|
| **Passif** (HTTP history) | Enregistrement silencieux du trafic | Faible |
| **Actif** (Intercept on) | Blocage et lecture avant envoi | Bloquer tout le trafic si oublié |

> ⚠️ **Point de vigilance** : laisser l'intercept activé bloque tout le trafic de l'émulateur.
> L'interception doit être courte, ciblée, et désactivée immédiatement après observation.

---

## 6. Analyse des données observées

### 6.1 Données observées en transit

| Type de donnée | Présence | Observation |
|---|---|---|
| Paramètres en URL | À vérifier | Potentiellement visibles en clair (HTTP) |
| Cookies de session | À vérifier | Attributs `Secure` / `HttpOnly` à contrôler |
| Tokens en URL | À vérifier | Risque d'exposition dans les logs serveur |
| En-têtes de sécurité | À vérifier | Présence de `X-Frame-Options`, `CSP`, etc. |

### 6.2 Risques potentiels identifiés

| Risque | Niveau | Justification |
|---|---|---|
| Trafic HTTP non chiffré | ⚠️ Moyen | Données lisibles sans déchiffrement |
| Absence d'attribut `Secure` sur cookies | ⚠️ Moyen | Cookie transmissible sur HTTP |
| Tokens/paramètres sensibles en URL | ⚠️ Moyen | Exposés dans l'historique proxy et logs |
| Trafic système Android visible | ℹ️ Info | Requêtes de mise à jour interceptées |

> **Distinction importante :**
> - **Observé** : requêtes HTTP visibles en clair dans Burp
> - **Supposé** : risques déduits de l'absence de mécanismes de protection
> - **Non confirmé** : exploitation réelle — hors périmètre de ce lab

---

## 7. Recommandations défensives

### Côté application mobile
- Utiliser **HTTPS exclusivement** pour toutes les communications réseau
- Implémenter le **Certificate Pinning** pour résister à l'interception proxy
- Ne jamais transmettre de données sensibles dans les paramètres URL

### Côté gestion des cookies
- Toujours définir les attributs **`Secure`** et **`HttpOnly`** sur les cookies de session
- Définir une durée de vie courte (`Max-Age`) pour les tokens de session
- Utiliser `SameSite=Strict` pour limiter les requêtes cross-site

### Côté bonnes pratiques Android
- Déclarer `android:usesCleartextTraffic="false"` dans le manifest
- Configurer une **Network Security Policy** stricte
- Ne pas stocker de données sensibles dans les SharedPreferences non chiffrées

---

## 8. Reproductibilité

Pour reproduire ce test dans le même lab :

```
1. VM Kali avec Burp Suite Community Edition
2. Listener Burp : *:8080 (All interfaces)
3. Émulateur Android : AVD Android 8.1.0 x86
4. Proxy Android configuré : 192.168.96.137:8080
5. Certificat CA Burp installé (USER store émulateur)
6. Cible : http://testphp.vulnweb.com
```

---

## 9. Nettoyage post-lab

- [ ] Certificat CA Burp supprimé de l'émulateur
- [ ] Proxy Android remis sur "None"
- [ ] Listener Burp remis sur `127.0.0.1` (loopback)
- [ ] Captures archivées dans un dossier sécurisé

---

