# Guide d'installation et d'exécution sur Windows 11

Ce document explique comment déployer Agent-S2 avec Qwen3‑32B et UI‑TARS‑1.5‑7B sur Windows 11. Les deux modèles sont servis via **vLLM** et accessibles en local.

## Préparer l'environnement

1. Installez **Python 3.10** ou supérieur depuis [python.org](https://www.python.org/). Lors de l'installation, cochez *Add Python to PATH*.
2. Installez **Git** pour cloner le dépôt.
3. *(Optionnel)* Créez un environnement virtuel :
   ```powershell
   python -m venv venv
   venv\Scripts\activate
   ```

## Cloner le dépôt

```powershell
git clone https://github.com/simular-ai/Agent-S.git
cd Agent-S
```

## Installer les dépendances

1. Installez la bibliothèque `gui-agents` et les dépendances :
   ```powershell
   pip install -r requirements.txt
   ```
2. Installez **Tesseract OCR** (requis par `pytesseract`). Téléchargez l'installateur depuis [tesseract-ocr.github.io](https://tesseract-ocr.github.io/) ou exécutez `choco install tesseract` si vous utilisez Chocolatey. Vérifiez que `tesseract.exe` est dans votre `PATH`.

## Configurer les modèles distants

Le fichier `examples/qwen_ui_tars_snippet.py` définit les paramètres des deux modèles vLLM :

```python
engine_params = {
    "engine_type": "vllm",
    "model": "Qwen3-32B",
    "base_url": "http://localhost:8002/v1",  # endpoint vLLM
}
engine_params_for_grounding = {
    "engine_type": "vllm",
    "model": "UI-TARS-1.5-7B",
    "base_url": "http://localhost:8001/v1",  # endpoint UI-TARS
    "grounding_width": 1000,
    "grounding_height": 1000,
}
```

Adaptez les URLs à votre déploiement si nécessaire.

## Lancer les services de modèles

Assurez‑vous que :

- Le serveur vLLM hébergeant **Qwen3‑32B** écoute sur `http://localhost:8002/v1`.
- Le serveur vLLM pour **UI‑TARS‑1.5‑7B** tourne sur `http://localhost:8001/v1`.

Les commandes de démarrage sont fournies ci‑dessous à titre d'exemple :

```bash
# UI‑TARS‑1.5‑7B sur un GPU
python -m vllm.entrypoints.openai.api_server \
  --host 0.0.0.0 --port 8001 \
  --model /data/huggingface/UI-TARS-1.5-7B \
  --served-model-name ui-tars-1.5-7B

# Qwen3‑32B sur quatre GPU
python -m vllm.entrypoints.openai.api_server \
  --host 0.0.0.0 --port 8002 \
  --model /data/huggingface/Qwen3-32B \
  --served-model-name qwen3-32b --tensor-parallel-size 4
```

## Exécuter le script

Depuis le dossier racine du dépôt :

```powershell
python examples\qwen_ui_tars_snippet.py
```

Le script initialise Agent-S2 puis lance la requête suivante :

```python
query = "Open a text editor and type 'Hello from Agent-S!'"
```

L'agent :

1. Interroge Qwen3‑32B pour déterminer les actions nécessaires.
2. Utilise UI‑TARS pour convertir les références visuelles en coordonnées.
3. Exécute les actions sur votre système via `pyautogui`.

## Conseils supplémentaires

- Sous Windows, il peut être nécessaire d'exécuter le terminal ou la session Python en tant qu'administrateur pour permettre à `pyautogui` et `pywinauto` d'interagir avec l'interface graphique.
- Les dépendances spécifiques à Windows (`pywin32`, `pywinauto`) sont installées automatiquement via `requirements.txt`.

Une fois ces étapes effectuées, le script devrait se connecter aux deux modèles distants et piloter votre machine Windows 11 pour exécuter la tâche de démonstration.
