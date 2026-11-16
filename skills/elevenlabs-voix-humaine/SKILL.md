---
name: elevenlabs-voix-humaine
description: Use whenever generating a voice-over or speech with the ElevenLabs API (TTS, narration, TikTok/video voice), so the voice sounds human and emotional instead of flat or robotic.
---

# Voix ElevenLabs humaine et émotive

Objectif : une voix qui sonne comme un vrai humain qui parle à quelqu'un, jamais comme une lecture plate de synthèse vocale. Applique ces règles par défaut. Si le prompt de l'utilisateur donne d'autres valeurs (modèle, réglages, voix), les siennes font foi, mais signale-lui en une ligne si elles risquent de rendre la voix monotone.

## 1. Sécurité
- Lis la clé dans la variable d'environnement `ELEVENLABS_API_KEY`. Ne l'écris jamais dans un fichier, ne l'affiche jamais, ne la mets jamais dans un log.
- Si la voix demandée n'est pas dans le compte, demande avant de l'ajouter depuis la bibliothèque partagée.

## 2. Modèle et réglages
- Modèle par défaut : `eleven_v3`, avec `language_code` correspondant à la langue du texte (ex. "fr"). C'est le modèle le plus expressif et le seul qui comprend les tags audio d'émotion.
- `stability` : 0.5 (« Natural ») par défaut. Essaie aussi la valeur la plus basse (« Creative », 0.0) et garde-la quand elle sonne mieux sans artefacts. N'utilise jamais 1.0 (« Robust ») : voix plate, tags ignorés.
- `similarity_boost` : 0.75–0.8.
- Si l'API refuse un paramètre pour v3 (style, use_speaker_boost, speed…), retire-le au lieu de forcer.
- Fixe un `seed` identique pour tous les segments d'un même projet, pour garder le même timbre.
- Demande une sortie haute qualité (44,1 ou 48 kHz, ex. `mp3_44100_192` ou `pcm_44100` si le plan le permet).
- Premier projet avec une voix donnée : génère une phrase test en `eleven_v3` ET en `eleven_multilingual_v2` (stability 0.3, similarity_boost 0.8, style 0.6, use_speaker_boost true) et demande laquelle sonne le plus humain. Si v3 sonne mal avec cette voix (accent, artefacts), préviens avant de continuer.

## 3. Écrire pour l'oral (étape la plus importante)
Avant toute génération :
1. **Définis un arc émotionnel.** Découpe le texte en segments et attribue à chacun un état (ex. intime → pressant → lucide → vulnérable → enflammé → chuchoté). Une voix qui reste sur un seul ton sonne robotique, même bien réglée. Écris cet arc dans `voice_arc.md`.
2. **Ajoute des tags audio v3** entre crochets, en anglais (non prononcés) : `[calm, low voice]`, `[firm]`, `[intense, emphatic]`, `[softly, sincere]`, `[warm, reassuring]`, `[knowing]`, `[scoffs]`, `[sighs]`, `[laughs]`, `[whispers]`, `[suddenly serious]`, `[energetic]`, `[short pause]`. Maximum 2–3 tags par segment : au-delà, la voix devient théâtrale.
3. **Joue sur la ponctuation** : points de suspension pour les hésitations et le suspense, phrases courtes, points d'exclamation pour l'énergie, tirets pour les ruptures. Pas de balises SSML `<break>` (non prises en charge par v3).
4. **Ne change jamais les mots** si l'utilisateur a donné un texte exact : seuls les tags et la ponctuation peuvent être ajoutés. Si le texte est libre, tu peux le réécrire pour l'oral (phrases plus courtes, tournures parlées) et montrer la version finale.
5. Les sous-titres ou textes à l'écran n'affichent jamais les tags.

## 4. Génération et choix des prises
- **Une requête par segment** (phrase ou phrase musicale), jamais tout le texte d'un bloc : chaque segment garde son énergie et le calage est plus simple.
- Génère **3–4 prises par segment**. Choisis dans cet ordre de priorité :
  1. l'émotion prévue dans l'arc est bien là ;
  2. aucun artefact (mot coupé ou mangé, souffle métallique, accent étranger, tag prononcé à voix haute) ;
  3. la durée convient (si un minutage est imposé).
- Range toutes les prises dans `voice/takes/` et note les choix dans `voice_takes.md` (prise retenue + raison en une ligne).
- Si un segment sonne encore robotique : change les tags, baisse la stabilité, ou découpe-le en morceaux plus courts, puis régénère.

## 5. Minutage des mots (si nécessaire)
- Utilise l'endpoint `with-timestamps` pour le minutage par mot, en ignorant les tags.
- S'il ne renvoie rien d'exploitable avec v3 : API Forced Alignment d'ElevenLabs sur l'audio retenu + texte sans tags, sinon alignement Whisper par mot. Dis quelle méthode a été utilisée.

## 6. Montage de la voix
- Coupe uniquement dans les silences, avec des fondus de ~5 ms.
- **Garde les respirations** : elles rendent la voix humaine. Atténue-les de 3–6 dB seulement si elles sont trop fortes.
- N'étire ou ne compresse jamais un segment de plus de 5 % (algorithme qui préserve la hauteur). Si une prise ne tient pas dans le temps, prends-en une autre plutôt que de la déformer.

## 7. Traitement (subtil, rendu « micro studio »)
- Passe-haut 80 Hz, chaleur +1,5 dB vers 150–200 Hz, présence +2 dB vers 3–5 kHz, de-esser doux.
- Compression légère (2,5:1 à 3:1) qui garde les nuances : les passages doux restent doux.
- Très légère réverbération de petite pièce (5–8 % de mix, ~0,4 s) pour éviter l'effet « collé » synthétique. Pas d'écho.
- Avec musique : ducking doux (~-7 dB, attaque ~30 ms, relâche ~250 ms), et -2 à -3 dB de plus sur les passages intimes ou vulnérables. La voix reste toujours claire et au-dessus de la musique.
- Niveau final par défaut : environ -14 LUFS, pics sous -1 dBTP (sauf consigne contraire).

## 8. Validation avant la suite
- Avant de monter la vidéo ou de livrer, fais écouter les segments retenus avec `voice_arc.md` et `voice_takes.md`, et demande si une partie sonne encore robotique. Régénère ce qui ne va pas avant de continuer.
