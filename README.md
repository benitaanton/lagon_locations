# Lagon Locations

**Des confirmations de réservation en PDF à un tableau de bord dont chaque chiffre est vérifié.**

Lagon Locations est une agence de gestion locative fictive, installée à l'Île Maurice, qui gère neuf logements pour trois propriétaires. Comme beaucoup d'agences, elle reçoit ses réservations sous forme de PDF, dans un format différent selon le canal, et suit ses encaissements dans un journal bancaire à part. Personne ne sait dire, à un instant donné, ce qui a été réellement payé et ce qui reste dû.

Ce projet construit l'outil qui répond à cette question. Un modèle d'IA lit les documents, un programme contrôle ce qu'il en a tiré, un rapprochement confronte les montants aux encaissements, et un tableau de bord présente le résultat. Le modèle n'est utilisé que pour lire : il ne calcule rien et n'invente rien.

## Voir le résultat

Le tableau de bord s'ouvre directement dans le navigateur : **https://benitaanton.github.io/lagon_locations/**

La note de projet (`Lagon_Locations_notice.pdf`) raconte la démarche en détail, du jeu de données aux contrôles.

## Ce que l'outil a trouvé

Sur quinze réservations et dix-sept mouvements bancaires, huit anomalies côté réservations et quatre côté paiements, dont :

- une réservation annonçant cinq nuits alors que les dates en donnent quatre ;
- un tarif facturé 129 € pour un logement dont le référentiel dit 124 € ;
- un document sans sous-total ni total, reconstitués à partir du tarif et des nuits, et signalés comme tels ;
- un double encaissement de 480 €, c'est-à-dire deux paiements pour un seul séjour ;
- un virement de 350 € qui ne correspond à aucune réservation connue.

Aucune de ces anomalies n'est corrigée automatiquement. L'outil les signale, les classe par gravité et laisse la décision à un humain.

## Comment ça marche

1. **Extraction.** Le texte de chaque PDF est envoyé à un modèle d'IA qui renvoie les champs sous forme structurée. La consigne lui interdit d'inventer une valeur manquante et de faire le moindre calcul. Un champ absent ressort comme absent.
2. **Contrôles.** Sept contrôles confrontent les valeurs lues au référentiel des logements et à elles-mêmes : nombre de nuits contre dates, sous-total contre tarif multiplié par les nuits, total contre sous-total plus frais de ménage, tarif et frais contre la grille, réservations annulées, doublons.
3. **Rapprochement.** Chaque réservation est rapprochée des mouvements du compte, par référence d'abord, par nom ensuite. Les écarts sont séparés en impayés et trop-perçus, parce qu'un solde net les aurait masqués l'un par l'autre.
4. **Tableau de bord.** Une page HTML autonome, sans serveur ni dépendance, qui présente la trésorerie, les réservations, les annonces et les points à traiter.

## Le principe de vérification

Chaque montant affiché porte son origine : lu dans le document, ou reconstitué par calcul. Les chiffres reconstitués sont signalés par un marqueur dans le tableau de bord. Ce qui ne peut pas être vérifié n'est pas deviné, il est remonté dans les fichiers d'anomalies.

Le jeu de données a été fabriqué volontairement avec des pièges, dans trois mises en page et deux langues, avec des documents incomplets, des incohérences internes et un doublon. Un jeu de données propre n'aurait rien prouvé.

## Rejouer le traitement

L'extraction est la seule étape qui appelle un modèle d'IA et consomme des crédits. La clé d'API est lue dans les secrets Colab ou dans la variable d'environnement `ANTHROPIC_API_KEY`, et n'apparaît nulle part dans le code.

Sans clé, le traitement se relance à partir de `resultats/reservations_extraites.xlsx` : les contrôles, le rapprochement et le tableau de bord reproduisent exactement les mêmes chiffres.

## Contenu du dépôt

```
index.html                       le tableau de bord
Lagon_Locations_VIE.ipynb        le traitement complet, étape par étape
Lagon_Locations_notice.pdf       la note de projet
donnees/
  pdf_reservations/              les 15 confirmations de réservation
  referentiel_logements.xlsx     les logements, propriétaires et tarifs
  paiements.xlsx                 le journal du compte
  annonces.xlsx                  les annonces en ligne et leurs résultats
resultats/
  reservations_extraites.xlsx    la sortie du modèle
  reservations_controlees.xlsx   après les sept contrôles
  anomalies.xlsx                 les anomalies de réservation
  rapprochement.xlsx             réservations et encaissements
  anomalies_paiements.xlsx       les anomalies de paiement
```

Les données sont entièrement fictives. Toute ressemblance avec une agence existante serait fortuite.

---

Projet réalisé par Benita Anton Arulnathan dans le cadre d'une candidature VIE.
