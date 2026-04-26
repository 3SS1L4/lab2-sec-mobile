# Rapport de Lab 2 : Rooting Android et Analyse de Sécurité

**Date :** 25 Avril 2026  
**Auteur :** AMSOU ISMAIL

---

## 1. Fiche Périmètre
* **Application cible + version :** Pizza recipes (`pizza.apk`)
* **Support de test :** AVD Pixel 3 XL, Android 9.0 (API 28) - Google APIs (x86)
* **Objectif :** Comprendre le rooting et ses impacts sur la sécurité globale.
* **Données :** Fictives uniquement (environnement isolé).
* **Réseau :** Réseau de test isolé.

---

## 2. Définition du Rooting
Obtenir l'accès "root" sur Android signifie acquérir les privilèges super-utilisateur, permettant un contrôle total sur l'appareil. Cette élévation de privilèges modifie les protections natives et altère la chaîne de confiance du système d'exploitation. Bien que cela soit extrêmement utile en laboratoire pour observer les comportements bas niveau ou analyser les données des applications, c'est une pratique très risquée. Par conséquent, toute manipulation de ce type nécessite un isolement strict, une traçabilité rigoureuse et une remise à zéro complète de l'environnement après utilisation.

---

## 3. Schéma de la Chaîne de Confiance (Verified Boot / AVB)
**Objectif :** Garantir que le système qui démarre est intègre et approuvé par le fabricant, sans compromission.

```text
[Matériel Sécurisé (Hardware Root of Trust)] 
       ↓ vérifie la signature de
[Bootloader] 
       ↓ vérifie l'intégrité de
[Partition Boot (Kernel)] 
       ↓ vérifie l'intégrité de
[Partition Système (OS Android)]
```
*Note sur l'AVB (Android Verified Boot) : AVB ajoute une vérification d'intégrité moderne et intègre une protection stricte contre le rollback (anti-retour en arrière) pour empêcher l'installation d'anciennes versions vulnérables du système.*

---

## 4. Matrice des Risques et Mesures Défensives

| # | Risque identifié | Mesure défensive associée |
|---|---|---|
| 1 | **Intégrité non garantie :** Conclusions biaisées sur la sécurité réelle de l'app. | **Journal de configuration :** Documenter l'état exact pour assurer la reproductibilité. |
| 2 | **Surface d'attaque accrue :** Exposition à des menaces externes si l'appareil sort du labo. | **Device/AVD dédié :** Utiliser un support exclusivement réservé aux tests de sécurité. |
| 3 | **Données sensibles exposées :** Violation potentielle de confidentialité. | **Données fictives :** N'utiliser que des comptes de test pour éliminer le risque de fuite. |
| 4 | **Instabilité système :** Tests non reproductibles et résultats incohérents. | **Contrôle strict des APK :** Limiter les installations aux seules applications cibles. |
| 5 | **Mélange comptes perso/test :** Fuite possible d'informations personnelles. | **Aucun compte personnel :** Vérifier l'absence de compte légitime sur le support. |
| 6 | **Mauvais nettoyage fin de séance :** Persistance de données sensibles. | **Reset/Wipe :** Wipe complet de l'AVD ou réinitialisation usine du device en fin de test. |
| 7 | **Réseau non isolé :** Effets involontaires ou fuites vers des systèmes externes. | **Réseau isolé :** Bloquer les communications non contrôlées (sandbox réseau). |
| 8 | **Traçabilité insuffisante :** Impossible de reproduire ou d'auditer les tests a posteriori. | **Horodatage + Captures :** Conserver des preuves visuelles de chaque étape critique. |

---

## 5. Référentiels OWASP Mobile

### OWASP MASVS (Mobile Application Security Verification Standard)
* **STORAGE-1 :** Les données sensibles (clés d'API, mots de passe, tokens) doivent être stockées de manière sécurisée en utilisant des fonctions de chiffrement appropriées plutôt qu'en clair sur le stockage local.
* **NETWORK-1 :** Les communications réseau doivent impérativement utiliser le protocole TLS avec une configuration robuste et une validation stricte des certificats.

### OWASP MASTG (Mobile Application Security Testing Guide)
* **Test 1 (Stockage) :** Avec les accès root, naviguer dans `/data/data/[package_name]/shared_prefs/` pour vérifier manuellement si les fichiers de préférences partagées exposent des données sensibles en clair.
* **Test 2 (Logs) :** Utiliser la commande `adb logcat` pendant l'exécution de l'application pour surveiller et détecter d'éventuelles fuites d'informations confidentielles ou de tokens dans les journaux système.

---

## 6. Fiche Environnement et Traçabilité
<img width="1276" height="573" alt="image" src="https://github.com/user-attachments/assets/ea3a2e51-ca66-418c-baab-802706996f9c" />
* **Version Android / API :** Android 9.0 / API 28 (x86)
* **Résultat `adb devices` :** `emulator-5554 device` 
* **Vérification Root (`adb shell id`) :** `uid=0(root) gid=0(root) groups=0(root),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats),3009(readproc),3011(uhid)`
* **État Verified Boot (`ro.boot.verifiedbootstate`) :** `orange` (Intégrité compromise en raison du déverrouillage pour le root).

### Scénarios de test :
1. **Scénario 1 :** Ouverture de l'écran d'accueil de l'application Pizza recipes. *(Observation : L'application démarre normalement, aucune détection de root au lancement).*
2. **Scénario 2 :** Recherche ou navigation vers une recette spécifique (ex: Pizza Margherita). *(Observation : Les images et la description se chargent sans erreur d'intégrité).*
3. **Scénario 3 :** Ouverture d'une fiche détaillée de la recette et interaction avec les boutons de partage. *(Observation : L'application fonctionne sans planter, prouvant qu'aucune vérification d'environnement d'exécution n'entrave son utilisation).*

* **Limites rencontrées :** Aucune limite technique majeure rencontrée sur l'AVD pour ces manipulations de base.

---

## 7. Checklist Finale de Remise à Zéro (Obligatoire)

- [x] Données de test supprimées.
- [x] Reset effectué (Wipe data AVD via Android Studio Device Manager).
- [x] Aucun compte personnel n'a été utilisé durant la session.
- [x] Rapport et traçabilité sauvegardés.
