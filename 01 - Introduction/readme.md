Qu’est-ce que CSPro?
=================
* Système intégré – saisie, apurement, tabulation, diffusion
* Domaine Publique (gratuit)
* Windows, Android
* Produit et suivi par le Bureau de Recensement des États-Unis et ICFI
* Financé par USAID

Modules
=======
| Tache                          | Module       |
|--------------------------------|--------------|
| Structure du Fichier de Données| Dictionnaire |
| Saisie/Validation              | CSEntry      |
| Apurement                      | CSBatch      |
| Tabulation                     | CSFreq/CSTab |
| Diffusion                      | TRS/MapView  |

Pour cet atelier on se concentre sur Dictionnaire et CSEntry

Création de l'Application de Saisie
===================================

* Lorsqu'on lance CSPro on demande le type d'application à créer.
	* Choisissez "data entry application" pour les enquêtes sur papier et "CAPI data entry application" pour CAPI.
	* CAPI data entry differ de data entry:
		* System controlled
		* CAPI Texte
		* Extended controls
		* Pas de "operator id"
		
* On commence avec le dictionnaire qui a l'hiérarchie suivante:

		Niveau (Level)
			Enregistrement (Record)
				Variable (Item)
					Sous-variable(Sub-item)

* Définir les identifiants: des variables qui sont unique pour chaque questionnaire. Typiquement des codes géographiques ou des codes de l'échantillonnage. Dans notre exemple utilisons:
	* Province
	* District
	* Unité primaire de sondage
	* Numéro de ménage
* Propriétés des variables:
	* Label: libellé visible aux intervieweurs. 
	* Name: Nom pour se référer a la variables dans la programmation. 
	* Start (position): numéro de la colonne dans le ficher de données ou se trouve le première caractère de la variable.
	* Len (longueur): nombre de chiffres maximum pour les numériques ou nombre de caractères maximum pour les alpha
	* Data type: alpha pour les noms et autres textes, numérique pour les chiffres et les réponse codées
	* Item type: variable normale ou sous variable (subitem).
	* Occ: nombre d'occurrences pour des variables qui sont répétées plusieurs fois dans l'enregistrement 
	* Dec: pour des nombres décimales le nombre de chiffres après la virgule.
	* Dec char: inclure ou ne pas inclure la virgule dans le fichier de données.
	* Zero fill: pour les numériques ajouter des zéros au lieu des blancs a gauche lorsque le nombre chiffres est moins de la longueur.
* Définir les enregistrements
	* Habitation
	* Population
* Propriétés des enregistrements
	* Label/Name: comme pour les variables
	* Type Value: pour faire la différence entre les différents types d'enregistrements dans le ficher de données.
	* Required: si l'enregistrement est requis, si oui, un questionnaire n'est pas valide si l'enregistrement n'est pas remplis. Dans notre exemple habitation est requis mais population n'est pas requis car il est possible d'avoir une maison sans habitant.
	* Max: pour les enregistrements a plusieurs occurrences le nombre maximum d'occurrences. Généralement c'est 1 pour les enregistrements qui ne sont pas répétés est un nombre très grands pour les enregistrements répétés. Dans notre exemple c'est 1 pour habitation est 999 pour population.
* Définir les variables dans les enregistrements
	* Habitation
		* No de pièces (numérique longueur 2)
		* Nature des murs (numérique longueur 1)
	* Population
		* Nom et prénom (alpha, longueur 50)
		* Sexe (numérique, longueur 1) 
		* Age (numérique, longueur 3)
* Définir les réponses possibles des variables (value sets)
	L'ensemble de valeurs défini les réponses valides de la variable et les libellés des codes pour des variables codées. Sans ensemble de valeurs on peut entrer n'importe quelle valeur (sauf blanc).
	* Nombre de pièces: 1 à 20
	* Nature des murs: 1. Dur 2. Tôle 3. Bois 4. Pisé 5. Feuilles
	* Nom et prénom: pas d'ensemble de valeurs
	* Sexe: 1. Masculin 2. Feminin
	* Age: 0 à 120
	* Province: copier du fichier Excel
	* District, UPS et numéro: revisitons le après
	* Numéro de ménage: 1 à 500 (nombre maximum de ménage dans l'UPS)
	
* Créer les formulaires
	* Glisser-déposer les enregistrements ou les variables individuelles sur les formulaires.
	* Nous pouvons y ajouter d'autres textes et des encadrements. Notons que ça ne sera pas visible sur Android.
	* Pour l'enregistrement population nous pouvons mettre sur sa propre formulaire ou le combiner avec habitation. Si il est seul on a le choix entre formulaire qui se répète ou roster (grille). 
* Tester l'application sur Windows
	* Cliquer sur le feu vert pour le lancer. On demande le nom du fichier de données. D'habitude on utilise l'extension ".dat".
	* Pour le moment on n'arrive pas a terminer le questionnaire car on ne peut pas sortir du roster. Ajoutons une variable pour contrôler le nombre de lignes dans le roster.
		* Ajouter la variable "nombre de personnes dans le ménage" à l'enregistrement habitation.
		* Glisser-déposer la variable sur la formulaire
		* Modifier l'ordre des champs dans l'arborescence de la formulaire pour mettre nombre de personnes avant le roster.
* Tester l'application sur Android
	* Publier le fichier .pen (File menu-->Publish Entry Application)
	* Connecter la tablette à l'ordinateur et copier les fichiers .pen et .pff sur la tablette dans la répertoire CSEntry.
	* Lancer CSEntry sur la tablette
	* Notons les différences entre Windows et Android
	* Copier le fichier de la tablette sur l'ordinateur

Fichier de Données
================
* Format texte (UTF8)
* Chaque ligne est un enregistrement
* Un enregistrement est compose de plusieurs champs de taille fixe
* La structure des enregistrements et ses champs sont définies par le dictionnaire
* Ouvrir le fichier de données qui nous venons de créer dans TextViewer. 
	* Identifier les enregistrements population et habitation a partir des record type.
	* Trouver les valeurs des variables. Marier les postions et longueur dans le dictionnaire avec les colonnes dans le fichier.

Exercices
=========
* Ajouter les variables suivantes au dictionnaire et à la formulaire:
	* Nature du toit (1. Béton armé, 2. Béton traditionnel, 3. Tôle, 4. Paille, 5. Autre) 
	* Nature du sol (1.	Ciment 2. Terre/Gravillon/Sable 3. Carreaux 4. Autre)
	* Lien de parenté avec le chef de ménage (1. Chef, 2. Épouse, 3. Fils/Fille, 4. Mere/Pere, 5. Autre relation, 6. Sans lien de parenté)
	* État matrimonial (1. Célibataire 2. Marié(e) 3. Divorcé(e) 4. Veuf/Veuve)
	* Nombre d'enfants nés vivants (0 à 9)
* Tester vos additions sur l'ordinateur et ensuite sur Android. Saisir trois ménages.
