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
- Génère **2 prises par segment**. Choisis dans cet ordre de priorité :
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

## 7. Traitement (adapté au contexte)

> Les valeurs ci-dessous sont des points de départ. Ajuste-les selon la scène, avec un but :
> la voix doit sonner vraie dans son contexte (un chuchotement reste fragile, un cri reste
> violent mais jamais agressif à l'oreille). L'important, c'est le contraste entre les moments,
> pas un son uniforme.
> Après le traitement, vérifie le niveau (LUFS, crêtes) et l'écart entre les passages doux
> et les passages forts avant de livrer.

### 7.1 Choisir le profil
- Lis le brief : narration / documentaire / tuto → **Narratif** (défaut).
  Fiction, horreur, thriller, ASMR, scène vécue → profils **immersifs**.
- Si la vidéo mélange les registres, choisis le profil **par segment**. Les audio tags
  ([whispers], [shouting], [panicked], [crying]…) et l'arc émotionnel indiquent lequel prendre.
- Entre deux profils, fais une transition en douceur (fondu de paramètres sur 100–300 ms).
  Jamais de saut brutal de compression ou de réverbération au milieu d'une phrase.
- Si le prompt donne des réglages explicites, ils passent avant le profil.

### 7.2 Profils

| | Narratif (défaut) | Intime / chuchoté | Tension / stress / panique | Cri / explosion |
|---|---|---|---|---|
| Passe-haut | 80 Hz | 100–120 Hz (le chuchotement n'a pas de grave utile) | 90 Hz | 100 Hz |
| Chaleur 150–200 Hz | +1,5 dB | +2 dB (effet de proximité) | 0 à +1 dB | 0 dB (sinon ça gonfle) |
| Présence 3–5 kHz | +2 dB | +0,5 à +1 dB seulement (le souffle siffle vite) | +2 à +3 dB (urgence, articulation) | −1 à −2 dB vers 2,5–4 kHz (dureté) |
| De-esser | doux | **fort** | moyen | moyen + limiteur |
| Compression | 2,5:1 à 3:1, lente | 2:1, très douce : garder le souffle | 3:1 à 4:1, attaque rapide (~5 ms) : respiration hachée audible | 4:1 + limiteur de crête : contenir sans écraser |
| Réverbération | petite pièce 5–8 %, ~0,4 s | quasi sèche (0–3 %) : la voix est « dans l'oreille » | sèche ou selon le lieu de la scène | selon le lieu ; une queue courte aide le cri à « exister » |
| Écart de niveau vs narratif | réf. 0 dB | −6 à −10 dB perçus | −2 à +2 dB | +4 à +6 dB perçus (jamais au-dessus du plafond de crête) |

### 7.3 Espace (immersif uniquement)
La réverbération suit le lieu de la scène, pas une règle fixe :
- Extérieur : aucune réverbération, éventuellement un léger filtre du bas (lointain).
- Petite pièce : 0,3–0,5 s.
- Couloir / cave / hangar : 0,8–1,5 s, plus de mix sur les cris.
- Voix « dans la tête » ou monologue intérieur : sèche, proche, légère chaleur en plus.

### 7.4 Musique et ambiance
- Narratif : ducking doux (~−7 dB, attaque ~30 ms, relâche ~250 ms), −2 à −3 dB de plus
  sur les passages intimes.
- Chuchoté : musique baissée de −10 à −12 dB, ou coupée. Le chuchotement doit rester intelligible.
- Panique : ducking plus rapide (attaque ~10 ms, relâche ~150 ms). La musique peut rester tendue,
  mais la voix reste devant.
- Cri : peu de ducking (−3 à −4 dB). Le cri passe naturellement par-dessus.
- Dans tous les cas, la voix reste intelligible au-dessus du fond.

### 7.5 Niveau final
- Normalise **la piste entière, jamais chaque segment séparément**. Sinon un chuchotement finit
  aussi fort qu'un cri et l'immersion disparaît.
- Narratif : ~−14 LUFS intégré, plage dynamique (LRA) ~5–8 LU.
- Immersif : −14 à −16 LUFS intégré, LRA de 10 à 15 LU accepté.
- Pics sous −1 dBTP partout (limiteur final transparent).
- Sauf consigne contraire.

## 8. Validation avant la suite
- Avant de monter la vidéo ou de livrer, fais écouter les segments retenus avec `voice_arc.md` et `voice_takes.md`, et demande si une partie sonne encore robotique. Régénère ce qui ne va pas avant de continuer.
