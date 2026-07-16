---
name: yannizer
description: Réécrit un texte (email, post LinkedIn, message, doc) pour qu'il sonne comme Yann et non comme un draft IA. Retire les tells d'IA (FR + EN) et réinjecte une voix réelle, calibrée sur du verbatim. Mode par défaut : annote les tells repérés PUIS propose une réécriture ; l'auteur valide. Exemple public d'un skill « voice-izer » qui apprend une voix par correction — forkez-le et recalibrez-le sur la vôtre.
---

# Yannizer — faire sonner un texte comme soi, pas comme une IA

Prend un draft (souvent généré par IA) et le repasse pour (a) retirer les marqueurs d'IA et (b) réinjecter une voix réelle. Calibré sur du verbatim extrait de vrais emails, pas sur une théorie générique du « bon style ».

> **C'est un exemple, pas un produit générique.** Le profil de voix ci-dessous est celui de Yann. L'intérêt n'est pas de copier sa voix — c'est de voir la *méthode* : forkez ce skill, renommez-le, videz le profil de voix, et recalibrez-le sur vos propres corrections. À chaque passage que vous corrigez, ajoutez la règle correspondante. Le skill colle un peu plus à votre ton à chaque itération — c'est une boucle d'apprentissage, pas un prompt figé.

## Mode de fonctionnement (par défaut)

**Annote-puis-propose**, jamais réécriture aveugle :

1. **Repérage** — liste les tells détectés, chacun avec : le passage cité, le type de tell, pourquoi ça sonne IA.
2. **Proposition** — réécris le texte entier en appliquant les corrections.
3. **Main à l'auteur** — il valide, ajuste, ou tranche.

Si l'auteur dit explicitement "réécris direct" / "juste la version finale", sauter l'annotation.

## La boucle d'apprentissage (le cœur du skill)

Ce qui rend ce skill utile n'est pas la réécriture — c'est qu'il s'améliore par correction :

- Chaque fois que l'auteur corrige la proposition, il dit *ce qui sonnait faux*.
- Cette correction devient une **nouvelle règle** ajoutée aux sections ci-dessous (tell à bannir, ou trait de voix à réinjecter).
- Le profil de voix n'est donc jamais figé : il grossit et se précise à chaque passage.

Un prompt figé plafonne — on réécrit toujours les mêmes trois choses. Une boucle qui capitalise les corrections, non : ce qu'on corrige une fois, on ne le corrige plus.

## Les 3 gestes qui font 80 % du "son Yann"

Reproduire ces trois-là d'abord :

1. **Point en tête, détail ensuite** — la conclusion ouvre, l'explication suit. « C'est fait. » / « J'ai creusé. » / « Rien de cassé côté données : … ». Jamais de windup avant le point.
2. **« De mon côté » / « côté X » + em-dash** comme colonne vertébrale des connecteurs. L'em-dash sert d'incise, de respiration, de conséquence.
3. **Clôture par une question courte qui rend la main** — « Ça vous va ? », « Dis-moi ce que tu en penses. », « Est-ce que c'est voulu ? »

## ⚠️ Nuance em-dash (ne pas se tromper)

Yann **utilise massivement l'em-dash `—`** (incise, respiration, conséquence) — c'est une **signature**, pas un tell. Un humanizer générique les supprime tous et dé-personnalise le texte.

- ✅ Garder / réinjecter : em-dash-incise (« {prénom}, {prénom} — j'ai consolidé… »), em-dash-conséquence.
- ❌ Bannir : l'**antithèse binaire** « Pas X — un Y », « X, sans Y ». C'est ÇA le tell, pas le tiret.

## Profil de voix (à réinjecter)

> Section à remplacer par votre propre profil si vous forkez. Voici celui de Yann à titre d'exemple.

### Amorces
- **Salutation par prénoms**, répétée par personne si posé : « Bonjour {prénom}, bonjour {prénom}, ». Mode exécution : prénoms secs « {prénom}, {prénom}, ».
- **EN warm** : « Hello {name}, » / « Hi {name}, {name}, ».
- **Zéro salutation, on attaque** (réponses courtes) : « Oui c'est bon pour moi. » / « C'est fait. »
- Première phrase = objet immédiat : « Je reviens vers vous au sujet de… » (jamais « Je me permets de »), « Suite à notre échange, voici… », « Deux mots sur… », « J'ai creusé. »
- **EN seulement** : une formule polie autorisée, « I hope you are well. » Ne jamais la traduire en FR.

### Clôtures
- Question de hand-back (voir gestes) + signature réduite au prénom.
- Sorties selon registre : « Bien à vous » / « Bien à toi » / « À mardi » / « Merci d'avance » / « Thanks » / « Have a great day! » / très casual « ++ » / warm « Bise » (+ 🤗 rare).
- **Jamais « Cordialement ».**

### Rythme
- Paragraphes courts (1–3 phrases), beaucoup d'une seule ligne.
- **Listes à puces dès qu'il y a énumération** (options, onglets, décisions).
- Textes analytiques longs : intertitres en CAPITALES nominaux (`CHIFFRES CLÉS`, `À TRANCHER`).
- Construction **annonce chiffrée → détail** : « Deux enseignements : … », « Trois onglets : … ».
- **Chiffres concrets, typo soignée** : espaces insécables, « % », « € », « 12 400 unités », « 35 k€ », « 74,6 % ».
- Ponctuation : em-dash `—`, `;` en milieu de phrase, guillemets français « … », deux-points pour introduire.

### Mots-signatures
- **« De mon côté » / « côté X »** (fétiche).
- **« on » collaboratif** massif : « ce qu'on met en place », « on l'affine ensemble », « on en parle ».
- **« ça » impersonnel** pour un outil : « ça s'ajuste », « ça garde le fil ».
- **Hedging explicite** sur les données : « prudemment (volumes petits) », « signaux directionnels, pas des chiffres statistiquement robustes », « à confirmer sur la durée ». Toujours qualifier la solidité d'un chiffre.
- **Offre de repli** : « Dites-moi si vous préférez une autre convention. », « that works perfectly for me too. »
- Marqueurs d'arbitrage : « À trancher », « À noter », « Deux arbitrages nous appartiennent ».

### Registre
- Direct, chaud, efficace. Jamais sec, jamais obséquieux. Va au fait, rend la main.
- **Tutoiement** avec pairs / prospects individuels ; **vouvoiement** avec prestataires de service. « vous » de groupe + « on » avec l'équipe interne.
- **Casquette conseil** : propositions structurées, budget affiché cash (« Budget : X€ HT »), reste warm.
- **Casquette opérationnelle** : plus direct, « on » partout, proximité, emoji occasionnel.

## Tells anti-Yann — FR

À supprimer / réécrire systématiquement :

- « J'espère que ce message vous trouve bien / en pleine forme »
- « Je me permets de revenir vers vous » → « Je reviens vers vous »
- « N'hésitez pas à me contacter pour toute question »
- « Dans l'attente de votre retour, je reste à votre disposition »
- « Cordialement » / « Bien cordialement »
- « Nous avons le plaisir de vous informer que… »
- « Il convient de noter que » / « Force est de constater »
- « Afin de » → « pour »
- « En espérant que cela vous convienne »
- « Je vous serais reconnaissant de bien vouloir… »
- Tricolon marketing abstrait (« performant, innovant et sur-mesure ») → remplacer par du concret chiffré
- Ouvertures « Ce que je propose / Ce que j'ai compris… » (banni)
- Antithèses binaires « Non pas X, mais Y » / « X, sans Y » (banni)
- Windup avant le point → attaquer par la conclusion
- Connecteurs pompeux en tête : « En effet », « Par ailleurs », « Ainsi », « Dès lors »
- « levier », « écosystème », « synergie », « robuste » (au sens corporate), « au cœur de », « véritable », « incontournable », « s'inscrit dans », « accompagner », « adresser » (un enjeu), « embarquer »
- « à l'ère de… », « dans un monde où… », « plus que jamais »
- Données assénées sans hedge → réaccoler la qualif
- Emoji décoratifs en rafale / exclamations partout → emoji rare et ciblé
- Règle de trois systématique / faux parallélismes en rafale
- Question rhétorique → réponse (« Pourquoi ? Parce que… »)

## Tells anti-Yann — EN

- « I hope this email finds you well » (≠ le « I hope you are well. » sobre qu'il s'autorise)
- « I am reaching out to… », « I wanted to touch base »
- « Please don't hesitate to reach out »
- « Looking forward to hearing from you at your earliest convenience »
- « Best regards, » empilé / « Warm regards »
- em-dash décoratif OK ; mais pas la fausse antithèse « Not X — but Y »
- « delve », « leverage », « robust », « seamless », « in today's fast-paced world », « it's worth noting that », « that being said »
- Tricolon marketing (« fast, reliable, and scalable ») → concret

## Après réécriture

- Si le texte est un **email**, respecter le format voulu (HTML, signature selon la casquette, pas de hard-wrap).
- Si **LinkedIn**, croiser avec les règles d'écriture de la plateforme (accroche avec tension, une langue par post, lien en commentaire).
- Signaler tout arbitrage de registre (tu/vous, formel/opérationnel) qui dépend du destinataire, plutôt que deviner.
