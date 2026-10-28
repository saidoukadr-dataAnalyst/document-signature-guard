# Document Signature Guard

Prototype Python de système anti-falsification documentaire, consacré à la signature numérique des PDF et à la traçabilité des opérations.

Le projet associe une empreinte SHA-256, une signature ECDSA, un tampon avec QR code et un module de signature PDF PAdES. Il propose une interface de bureau CustomTkinter et une architecture API FastAPI avec traitement asynchrone Celery / Redis.

**Statut : prototype en développement.** La présence des modules ne constitue pas une validation de sécurité ni une garantie d'intégrité de bout en bout. Les limites connues sont détaillées ci-dessous.

## Fonctionnement

1. Sélectionner un PDF ; l'analyse OCR et la détection de données sensibles (DLP) sont optionnelles.
2. Calculer l'empreinte SHA-256 du document original.
3. Signer cette empreinte avec une clé ECDSA P-256.
4. Insérer un tampon contenant un QR code avec l'identifiant du signataire, l'empreinte et la signature.
5. Appliquer la couche de signature PDF via pyHanko.
6. En option, enregistrer l'empreinte dans le registre local ou l'ancrer sur Ethereum Sepolia si l'intégration est configurée.

La vérification actuelle du QR contrôle la signature de l'empreinte embarquée avec la clé publique chargée. **Elle ne compare pas cette empreinte au contenu du PDF présenté et ne valide pas sa signature PAdES.** Un QR valide recopié dans un autre document ne prouve donc pas l'intégrité de celui-ci. La validation du PDF signé est un chantier prioritaire.

## Modules

| Fichier / dossier | Rôle |
| --- | --- |
| `app_unifiee.py` | Interface de bureau : signature, vérification, administration et workflows |
| `crypto_engine.py` | Empreintes signées en ECDSA P-256, protection des clés privées par mot de passe et Scrypt |
| `pdf_processor.py`, `stamp_engine.py` | Traitement PDF, QR codes et tampon visuel |
| `pades_engine.py` | Intégration pyHanko et certificats PKCS#12 auto-signés pour la démonstration |
| `auth_manager.py`, `eid_manager.py` | Comptes, mots de passe bcrypt et module TOTP |
| `kms_manager.py` | Stockage chiffré des identités avec Fernet et PBKDF2 |
| `audit_logger.py` | Journalisation avec chaînage HMAC |
| `ocr_engine.py`, `dlp_engine.py` | Extraction de texte et détection de données sensibles |
| `workflow_engine.py` | Circuits d'approbation des documents |
| `blockchain_engine.py` | Ancrage Sepolia, registre local et file d'attente |
| `api/` | API FastAPI et tâches Celery dans la version publiée du dépôt |
| `index.html` | Page de présentation statique ; ce n'est pas l'application de signature |

## Installation et lancement local

Depuis la racine du dépôt, dans PowerShell :

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python app_unifiee.py
```

L'interface nécessite un environnement graphique avec Tkinter. Le dossier `security_vault/` est créé au démarrage. Le compte de démonstration initial est défini dans `auth_manager.py` ; son mot de passe doit être remplacé pour toute utilisation au-delà d'un essai local.

Les intégrations optionnelles utilisent des dépendances absentes du fichier principal :

```powershell
python -m pip install pdfplumber pytesseract pyotp web3
```

L'OCR des documents scannés nécessite aussi le programme Tesseract et ses données de langues française, arabe et anglaise accessibles sur la machine. Les paramètres Scrypt actuels demandent environ 1 Go de mémoire lors de la dérivation d'une clé.

### API et worker

Ces commandes supposent que le dossier `api/` est présent. Si votre copie locale affiche sa suppression dans `git status`, le lancement de l'API est indisponible dans cette copie.

La configuration Docker Compose définit l'API, Redis et un worker Celery avec un volume partagé. Elle attend un fichier `.env` local, non versionné. Pour un essai isolé :

```powershell
if (-not (Test-Path .env)) { New-Item -ItemType File .env }
docker compose up --build
```

L'API expose alors sa documentation à `http://localhost:8000/docs` et son endpoint de santé à `http://localhost:8000/health`. Ce dernier confirme seulement que le processus API répond.

### Ancrage blockchain optionnel

Le module lit les variables d'environnement `SEPOLIA_RPC_URL`, `NOTARY_CONTRACT_ADDRESS` et `WALLET_PRIVATE_KEY`. Un contrat compatible avec l'ABI du module et un portefeuille de test sont nécessaires pour les transactions réelles.

- **Simulation** : Web3 absent ; les identifiants de transaction commencent par `0xSIM_`.
- **Dégradé** : réseau ou configuration indisponible ; enregistrement local et file d'attente.
- **Live** : transaction envoyée au contrat sur Sepolia.

Un enregistrement local ou simulé n'est pas une preuve d'ancrage sur la blockchain. L'application de bureau lit l'environnement du processus ; déposer un fichier `.env` ne suffit pas à y charger ces variables.

## Limites et prochaines étapes

- Valider la signature PAdES et les modifications du PDF avant d'afficher un verdict d'authenticité ; tester notamment la substitution de QR et la modification du document.
- Configurer et tester la chaîne de confiance, l'horodatage et les informations de révocation. Les certificats auto-signés et les options LTV/LTA du code ne suffisent pas à établir une validation à long terme.
- Corriger l'intégration de la réponse blockchain dans l'interface : elle attend actuellement `record`, alors que le moteur renvoie `local_record` et `chain_record`.
- Renforcer le contrôle d'accès des routes API, la gestion des secrets et l'initialisation des comptes avant toute exposition réseau.
- Compléter les dépendances optionnelles et automatiser les tests de signature, d'altération et de vérification.
- Réconcilier les anciens documents d'audit et `task.md` avec l'état actuel du code.

Les fichiers de clés, certificats privés, secrets et le dossier `security_vault/` sont exclus par `.gitignore`. Aucun document confidentiel ne doit être ajouté aux exemples publics.

## Présentation du projet

**Document Signature Guard — Signature numérique et anti-falsification documentaire.**
Prototype Python combinant SHA-256, ECDSA, QR codes et intégration PAdES pour explorer la protection et la traçabilité des PDF. Modules complémentaires : authentification, journal d'audit, OCR/DLP, workflows et ancrage blockchain optionnel.

[Code source](https://github.com/saidoukadr-dataAnalyst/document-signature-guard) · [Dépôt du portfolio](https://github.com/saidoukadr-dataAnalyst/portfolio)
