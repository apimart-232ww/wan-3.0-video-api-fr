# API Wan 3.0 : identifiants de modèle, tarif à la seconde, exemples

> **0.0329 $ par seconde (480P)**, facturé à l'usage. Recharge à partir de 1 $ et une seule URL compatible OpenAI : `https://api.apimart.ai/v1`.

<p align="center"><img src="assets/01-preview-thumb.jpg" width="820" alt="Wan 3.0 sample frame"></p>
**[Page modèle Wan 3.0](https://go.apimart.ai/k-073c51)** · **[Tarifs en direct](https://go.apimart.ai/k-e4ca28)** · **[Obtenir une clé API](https://go.apimart.ai/k-fe9d5b)**

La dernière génération d'Alibaba : moins de 3,3 cents la seconde en 480p, même API pour texte et image.

## Pourquoi appeler Wan 3.0 via APIMart

- **Une clé pour tout le catalogue.** La même URL de base et le même en-tête d'authentification donnent accès à Wan 3.0 et à plus de 300 modèles image, vidéo et langage : seul le champ `model` change.
- **1 $ minimum, à l'usage.** Ni abonnement ni forfait prépayé, et aucun crédit gratuit à épuiser d'abord : le tarif du tableau est le tarif réel.
- **Le montant est renvoyé dans la réponse.** Chaque appel retourne `cost` / `credits_cost`, donc la dépense se lit appel par appel.
- **Pensé pour l'asynchrone.** Soumission, récupération du `task_id`, puis interrogation de `GET /v1/tasks/{id}` : les lots et les reprises restent de la logique de file classique.

## Identifiant de modèle et endpoint

| Champ | Valeur |
| --- | --- |
| `model` | `wan3.0-video` |
| endpoint | `POST https://api.apimart.ai/v1/videos/generations` |
| task | GET /v1/tasks/{id} |


## Exemples réellement générés (résultats d'appels réels)

| Sortie | file | Coût | prompt |
| --- | --- | --- | --- |
| <img src="assets/01-preview-thumb.jpg" width="260"> | [01-preview.mp4](assets/01-preview.mp4) | $0.1644 | `villa moderne sur une falaise, heure dorée, piscine reflétant le ciel, travelling avant le` |

## Tarifs relevés

<!-- pricing:model:start -->
| Sortie | Tarif |
| --- | --- |
| `480P` | $0.0329 |
| `720P` | $0.0658 |
<!-- pricing:model:end -->

## Paramètres de requête

| Champ | Valeur |
| --- | --- |
| `model` | `wan3.0-video` |
| `resolution` | `480p / 720p / 1080p` |
| `duration` | `5 / 10 秒` |
| `n` | `1` |

## Démarrer en 60 secondes

```bash
export APIMART_API_KEY="<token>"
curl --request POST \
  --url https://api.apimart.ai/v1/videos/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" \
  --header 'Content-Type: application/json' \
  --data '{"model":"wan3.0-video", "prompt":"a cozy reading nook by a rainy window, warm lamp light", "n":1}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

r = requests.post(f"{BASE}/videos/generations", headers=HEADERS, timeout=60, json={
    "model": "wan3.0-video",
    "prompt": "a cozy reading nook by a rainy window, warm lamp light",
    "n": 1,
})
r.raise_for_status()
task_id = (r.json().get("data") or {}).get("id")
while True:
    t = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json().get("data", {})
    if t.get("status") in ("completed", "failed"):
        print(t.get("status"), t.get("cost"))
        break
    time.sleep(5)
```

## Coût en volume

| Volume | Coût |
| --- | --- |
| 60 secondes | $1.9728 |
| 600 | $19.728 |

Calcul linéaire au tarif indiqué, sans remise de volume. Vérifiez les tarifs en direct avant de budgéter.（snapshot 2026-09-21）

## Dépannage du premier appel

| Symptôme | Cause | Correctif |
| --- | --- | --- |
| `401` / invalid api key | clé absente, tronquée, ou retour à la ligne collé dans l'en-tête | Recopiez-la depuis la console ; l'en-tête est `Authorization: Bearer $APIMART_API_KEY` |
| solde insuffisant / erreur credit | le compte n'a pas de solde | Rechargez à partir de 1 $ dans la console — il n'y a pas de quota gratuit |
| `429` | trop de requêtes simultanées sur une clé | Attendez puis réessayez avec le même `Idempotency-Key` |
| `400` / modèle introuvable | identifiant ou paramètre incorrect | Reprenez la valeur exacte de `model` dans le tableau ci-dessus ; les champs diffèrent selon le palier |
| tâche en `failed` | prompt filtré ou URL d'image de référence expirée | Relancez avec un **nouveau** `Idempotency-Key` et réhébergez l'image de référence |

## Questions fréquentes

**Quelle est l'unité de facturation ?**

L'image, la seconde de vidéo ou le million de tokens selon le modèle. Le montant figure dans la réponse de la tâche, donc il se vérifie ligne par ligne.

**Peut-on consulter le détail ?**

La page de facturation de la console liste chaque appel et la consommation associée.

**Depuis quel langage appeler l'API ?**

N'importe quel client HTTP. Comme l'API est compatible OpenAI, Python fonctionne en changeant simplement la base_url du SDK openai.

**Les URL de résultat expirent-elles ?**

Oui. Téléchargez le fichier dans votre propre stockage dès que la tâche est terminée.

## Divulgation

Ce dépôt décrit l'usage d'APIMart, un service de relais tiers, sans lien avec les fournisseurs de modèles. Les tarifs et paramètres correspondent au relevé indiqué ; la facturation réelle fait foi sur la console.

## Structure du dépôt

```
README.md            本文件
data/model.json      模型 ID、价格快照、参数
examples/curl.sh     curl 示例
examples/python.py   Python（提交 + 轮询）
LICENSE, .gitignore
```

## License

MIT
