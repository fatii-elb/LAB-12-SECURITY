# LAB 12 — Bypass de la Détection de Root Android avec Medusa

**Cours :** Sécurité des applications mobiles
**Outil principal :** Medusa (basé sur Frida)
**Application cible :** OWASP MSTG Uncrackable Level 1 (`owasp.mstg.uncrackable1`)
**Environnement :** Émulateur Android 5554 (Android 14, x86_64)

---

## Objectif

Réaliser un bypass de la détection de root sur une application Android en utilisant Medusa, puis valider que l'application ne bloque plus l'accès malgré l'environnement rooté.

---

## Environnement utilisé

| Élément | Détail |
|---|---|
| OS | Windows 10 |
| Python | 3.14.4 |
| pip | 26.0.1 |
| Frida | 17.9.6 |
| ADB | 1.0.41 |
| Appareil | Émulateur Android 5554 (Android 14, x86_64) |
| App cible | owasp.mstg.uncrackable1 |

---

## Étape 1 — Vérification de l'environnement

Vérification que Python, pip et ADB sont correctement installés :

```
python --version
pip --version
adb version
```

Connexion de l'émulateur confirmée :

```
adb devices
```
<img width="1636" height="503" alt="image" src="https://github.com/user-attachments/assets/5b2d9e7e-d6b0-4651-b955-9500220fc4cd" />


---

## Étape 2 — Installation et vérification de Frida

Installation de Frida sur le PC :

```
pip install --upgrade frida frida-tools
frida --version
python -c "import frida; print(frida.__version__)"
```

Les deux commandes affichent la version **17.9.6** — versions synchronisées. ✅

<img width="1714" height="244" alt="image" src="https://github.com/user-attachments/assets/6235cf73-fe4b-455f-a498-ef3ca72ed62f" />

---

## Étape 3 — Démarrage de frida-server sur l'émulateur

Identification de l'architecture CPU de l'émulateur :

```
adb shell getprop ro.product.cpu.abi
```

Résultat : **x86_64**

Démarrage du frida-server (déjà présent sur l'émulateur) :

```
adb shell "/data/local/tmp/frida-server -l 0.0.0.0 &"
```

Vérification que le serveur tourne :

```
adb shell ps | findstr frida
```

Vérification que Frida voit les applications de l'émulateur :

```
frida-ps -Uai
```

<img width="1658" height="238" alt="image" src="https://github.com/user-attachments/assets/e48f6944-fe83-4bb2-9d68-f4e32b1e8e6a" />
<img width="1675" height="1208" alt="image" src="https://github.com/user-attachments/assets/809da74a-e134-4067-a4ba-b49bdbc77388" />

---

## Étape 4 — Installation de Medusa

Clonage du dépôt et installation des dépendances :

```
git clone https://github.com/Ch0pin/medusa.git
cd medusa
pip install -r requirements.txt
python medusa.py --help
```

Medusa charge **124 modules** disponibles. 

<img width="1655" height="437" alt="image" src="https://github.com/user-attachments/assets/6ab53666-7c2c-4f73-939d-91ac4941afb7" />
<img width="1658" height="293" alt="image" src="https://github.com/user-attachments/assets/e8b58d9e-39cf-44f2-a7a6-343debb0a680" />
<img width="1698" height="606" alt="image" src="https://github.com/user-attachments/assets/65a36410-65aa-4be8-adae-a6841ff6b136" />
<img width="1676" height="855" alt="image" src="https://github.com/user-attachments/assets/d3126900-f395-4b29-9824-f4bdb2fdb99b" />

---

## Étape 5 — Bypass avec Medusa

Lancement de Medusa et sélection de l'émulateur :

```
python medusa.py
```

Sélection du device : **emulator-5554** (index 3)

Chargement du module de bypass root :

```
use root_detection/universal_root_detection_bypass
```

Modules disponibles identifiés lors de la recherche :

```
search root
```

```
root_detection/universal_root_detection_bypass
root_detection/rootbeer_detection_bypass
root_detection/rootbeer_detection_bypass_no_obfuscation
root_detection/jailMonkey_react_native
```

Lancement du bypass sur l'application cible (après ouverture de l'app sur l'émulateur) :

```
run owasp.mstg.uncrackable1
```

Résultat dans la console Medusa :

```
Attaching frida session to PID - 5751
---------LOADING ANTI ROOT DETECTION SCRIPT-------------------
Loaded 24885 classes!
Script is compiled
-------------------Application Info--------------------
- Frida version: 17.9.1
- Application Name: android.app.Application
- Package Code Path: .../owasp.mstg.uncrackable1.../base.apk
```

<img width="1653" height="1208" alt="image" src="https://github.com/user-attachments/assets/faccde6a-4463-4d1b-bb91-a779c2259d09" />
<img width="925" height="814" alt="image" src="https://github.com/user-attachments/assets/0a84d2a8-8315-4ce6-836e-f23a1b921823" />

---

## Étape 6 — Validation du bypass

Après injection du module par Medusa, l'application Uncrackable Level 1 s'ouvre et fonctionne normalement sur l'émulateur rooté, sans afficher de popup "Root detected" ni se fermer automatiquement.

<img width="945" height="172" alt="image" src="https://github.com/user-attachments/assets/68b43353-a672-4219-92b7-298cd2039e2c" />
<img width="915" height="143" alt="image" src="https://github.com/user-attachments/assets/3096eef4-3a72-4b8f-82f0-cc9904ed0b88" />
<img width="941" height="125" alt="image" src="https://github.com/user-attachments/assets/aed45f27-5b68-4e1d-a92b-114ad33f1577" />
<img width="930" height="672" alt="image" src="https://github.com/user-attachments/assets/a6cced4c-13e9-442e-aba9-e8bf015008da" />

---

## Explication technique

L'application détecte le root via plusieurs mécanismes Java :

- Lecture de `Build.TAGS` (valeur suspecte : `dev-keys` au lieu de `release-keys`)
- Vérification de l'existence de fichiers comme `/system/xbin/su` via `File.exists()`
- Tentative d'exécution de `su` via `Runtime.exec()`

Le module `universal_root_detection_bypass` de Medusa intercepte ces appels via Frida et renvoie de fausses réponses :

- `Build.TAGS` → retourne `release-keys`
- `File.exists()` sur les chemins suspects → retourne `false`
- `Runtime.exec("su")` → bloqué

L'application reçoit des réponses falsifiées et considère l'environnement comme non rooté.

---

## Résumé des livrables

| # | Livrable | Statut |
|---|---|---|
| 1 | Preuve d'installation (frida, adb) | ✅ |
| 2 | frida-server actif + frida-ps -Uai | ✅ |
| 3 | Medusa installé avec 124 modules | ✅ |
| 4 | Bypass réussi sur Uncrackable 1 via Medusa | ✅ |
| 5 | Application fonctionnelle sur émulateur rooté | ✅ |

---

## Note éthique

Toutes les techniques utilisées dans ce lab ont été appliquées uniquement sur des applications et un environnement de test dédiés à la formation en sécurité mobile (OWASP MSTG). Aucune application réelle n'a été ciblée.
