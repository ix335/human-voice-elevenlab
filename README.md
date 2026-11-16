# human-voice-elevenlab

Un skill pour **Claude Code** qui rend les voix générées avec l'API ElevenLabs plus **humaines et émotives**, au lieu de plates et monotones.

Une fois installé, Claude Code l'utilise automatiquement dès qu'il génère une voix avec ElevenLabs.

## Ce que fait le skill

- Utilise le modèle **`eleven_v3`** (le plus expressif), avec une stabilité basse (« Natural » ou « Creative »).
- Construit un **arc émotionnel** : chaque phrase a son propre ton (intime, pressant, vulnérable, enflammé…).
- Ajoute des **tags audio** (`[whispers]`, `[softly, sincere]`, `[intense, emphatic]`…) et une ponctuation expressive, sans changer les mots du texte.
- Génère **une requête par phrase** et **plusieurs prises**, puis garde la plus naturelle.
- **Garde les respirations** au montage et applique un traitement léger (EQ, compression douce, un peu de réverbération de pièce).
- Te fait **écouter les voix** avant de continuer le projet.

Si ton prompt impose d'autres réglages, le skill les respecte, mais te prévient s'ils risquent de rendre la voix monotone.

## Installation

### Option 1 : avec git

**macOS / Linux**
```bash
git clone https://github.com/ix335/human-voice-elevenlab.git
mkdir -p ~/.claude/skills
cp -r human-voice-elevenlab/skills/elevenlabs-voix-humaine ~/.claude/skills/
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/ix335/human-voice-elevenlab.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse human-voice-elevenlab\skills\elevenlabs-voix-humaine "$env:USERPROFILE\.claude\skills\"
```

### Option 2 : à la main

Copie le dossier `skills/elevenlabs-voix-humaine/` dans `~/.claude/skills/` (sous Windows : `C:\Users\<toi>\.claude\skills\`).

Pour que le skill ne serve que dans un seul projet, copie-le plutôt dans `.claude/skills/` à la racine de ce projet.

Redémarre ensuite Claude Code pour qu'il le prenne en compte.

## Prérequis

- Un compte ElevenLabs et une clé API dans la variable d'environnement `ELEVENLABS_API_KEY`. Le skill ne l'écrit jamais dans un fichier et ne l'affiche jamais.
- `ffmpeg` installé, pour le montage et le traitement audio.

## Licence

MIT, voir [LICENSE](LICENSE).
