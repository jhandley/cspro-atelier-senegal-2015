Contrôles de cohérence avec logique
===================================
Nous avons vu comment comment limiter les réponses avec les ensembles de valeurs mais des fois il faut des contrôles plus complexes. Pour cela il faut passer par la logique.

Dans ce leçon nous apprendrons comment:
* ajouter la logique dans l'application de saisie
* utiliser la commande *errmsg* pour informer l'interviewer des erreurs de saisie  
* Utiliser *if* *then* *else* pour les commandes conditionnelles dans les contrôles de cohérence à plusieurs variables
* Utiliser les boucles pour les contrôles de cohérence sur les groupes (rosters) 
* Comprendre l'ordre des postprocs et preprocs
* Utiliser la commande *skip* pour ajouter des sauts
* Comprendre les valeurs spéciales *notappl* et *missing* pour les champs sautés et inconnus
* Remplir les variables automatiquement avec la logique
	

La fonction errmsg
-----------------------------------
Dans notre exemple il est possible de mettre un nombre de pièces pour dormir qui dépasse le nombre de pièces total. Ajoutons un message d'erreur dans ce cas.
* Cliquer sur nombre de pièces pour dormir dans l'arborescence de la formulaire et puis sur *Logic* dans la barre d'outils 
* Ajouter le code suivant au procédure du champs NOMBRE_DE_PIECES_DORMIR:

```
PROC NOMBRE_DE_PIECES_DORMIR

if NOMBRE_DE_PIECES_DORMIR > NOMBRE_DE_PIECES then
	errmsg("Le nombre de pièces pour dormir dépasse le nombre de pièces total");
	reenter;
endif
```

* Il est possible d'insérer des valeurs dans le message en ajoutant %d (%s pour les alpha)

```
PROC NOMBRE_DE_PIECES_DORMIR

if NOMBRE_DE_PIECES_DORMIR > NOMBRE_DE_PIECES then
	errmsg("Le nombre de pièces pour dormir, %d, dépasse le nombre de pièces total de %d", 
		   NOMBRE_DE_PIECES_DORMIR, NOMBRE_DE_PIECES);
	reenter;
endif
```

* En ajoutant *select* on donne l'option de revenir sur un autre champ:

```
PROC NOMBRE_DE_PIECES_DORMIR

if NOMBRE_DE_PIECES_DORMIR > NOMBRE_DE_PIECES then
	errmsg("Le nombre de pièces pour dormir, %d, dépasse le nombre de pièces total de %d", 
		   NOMBRE_DE_PIECES_DORMIR, NOMBRE_DE_PIECES)
		   select("Corriger nombre de pièces pour dormir", NOMBRE_DE_PIECES_DORMIR, 
		   		  "Corriger nombre de pièces total", NOMBRE_DE_PIECES);
endif
```
Notez qu'avec *select* nous n'avons plus besoin du *reenter*.

* Prenons un autre exemple: afficher un message si le chef du ménage est âgé de moins de 12 ans.
```
if LIEN_DE_PARENTE = 1 and AGE < 12 then
	errmsg("Le chef du ménage a %d ans mais l'âge minimum est 12", AGE)
		select("Corriger age", AGE, "Corriger lien de parenté", LIEN_DE_PARENTE);
endif
```

* Dans la commande *if* *then* on peut combiner plusieurs conditions avec *and* et *or*

Postproc et preproc
-------------------
* Dans nos exemples la logique est exécuté après la saisie de la variable. On peut aussi exécuter de la logique avant, dans le *preproc*
* Remplissons automatiquement la date de l'interview dans le preproc

```
PROC JOUR_INTERVIEW
preproc
DATE_INTERVIEW = sysdate("DDMMYYYY");
```

* Ajoutons *noinput* pour éviter la saisie de la date

```
PROC JOUR_INTERVIEW
preproc
DATE_INTERVIEW = sysdate("DDMMYYYY");
noinput;

PROC MOIS_INTERVIEW
preproc
noinput;

PROC ANEE_INTERVIEW
preproc
noinput;
```
* Ou bien on peut changer ces champs en *protected* (protégées). Dans ce cas on n'a plus besoin de *noinput*.

Sauts
------
* Sauter le montant de loyer si le statut d'occupation n'est pas égal à "locataire" ou "location terre"

```
PROC STATUT_OCCUPATION

if not STATUT_OCCUPATION in 2:3 then
	skip to NOMBRE_DE_PERSONNES;
endif
```

* Noter qu'après avoir sauté montant de loyer, on ne peut pas y revenir en faisant marche arrière  
* Si l'individu a moins de 12 ans sauter état matrimonial et nombres d'enfants nés vivants

```
if AGE < 12 then
	skip to next;
endif
```

* Noter qu'on utilise `skip to next` au lieu de `skip to NOM_ET_PRENOM`
* Lorsque le champ est sauté les procs du champs ne sont pas exécutés

Les commandes *accept*, *endgroup* et *endlevel*
------------------------------------------------

* A lieu de demander le nombre de personnes dans le ménage nous pouvons terminer le roster lorsque le nom de la personne est blanc. N'oublier pas d'enlever le "roster control field".

```
PROC NOM_ET_PRENOM

if NOM_ET_PRENOM = "" then
	endgroup;
endif;
```

* Nous pouvons également utiliser *endlevel* comme le groupe (le roster) est à la fin du questionnaire.
* Utiliser la fonction *accept* pour vérifier qu'on veut vraiment terminer et ce n'est pas un erreur de saisie.

```
PROC NOM_ET_PRENOM

if NOM_ET_PRENOM = "" then
	if accept("Terminez saisie du ménage?","Oui", "Non") = 1 then
		endgroup;
	else
		reenter;
	endif;
endif;
```

Blancs et Inconnus
------------------
* La valeur d'une variable sautée est *notappl* (blanc). Ajouter le code suivant au preproc du nom et prénom. Tester en saisissant des valeurs différentes pour statut d'occupation.  

```
if MONTANT_DU_LOYER = notappl then
	errmsg("Montant du loyer est blanc");
else
	errmsg("Montant du loyer est %d", MONTANT_DU_LOYER);
endif;
```

* Même si on saisie le champ et puis on le saut après, il deviendra *notappl*
* Même si *notappl* se trouve dans l'ensemble de valeurs, il n'est pas possible de mettre blanc sauf si on utilise la commande *set* *behavior*:

```
PROC MONTANT_DU_LOYER

preproc
set behavior() canenter(notappl) on (noconfirm);
postproc
set behavior() canenter(notappl) off;
```

* Si on utilise le mode "operator controlled" cela n'est pas nécessaire.

* Si on n'éteint pas *canenter(notappl)* il sera possible de saisir des blancs pour tous les champs qui suit. Alternativement on peut donner le nom de la variable pour ne pas faire le changement global.

```
PROC MONTANT_DU_LOYER

preproc
set behavior(MONTANT_DU_LOYER) canenter(notappl) on (noconfirm);
```

* En plus de *notappl* il y a une autre valeur spéciale *missing* pour les valeurs inconnu. Dans l'ensemble de valeurs on peut ajouter *missing* a une des valeurs. Souvent ces valeurs sont codées avec 9, 99, 999... Dans la logique on peut comparer ces valeurs avec *missing* au lieu de la code numérique.
 
```
if MONTANT_DU_LOYER = missing then
	errmsg("Montant du loyer est inconnu");
else
	errmsg("Montant du loyer est %d", MONTANT_DU_LOYER);
endif;
```

Ordre des procs
---------------
* Dans notre exemple il est possible de saisir un ménage sans chef ou avec deux chefs. Ajouter un contrôle pour l'empêcher. On l'ajoute où?
* Ce n'est pas seulement les champs qui ont des procs. Le niveau, la formulaire et les rosters ont des procs aussi.
* Ajouter des errmsg dans le preproc et postproc du niveau, formulaire et le roster pour illustrer l'ordre des procs. Pouvez-vous prévoir l'ordre des messages?

```
preproc
errmsg("%s preproc", getsymbol());

postproc
errmsg("%s postproc", getsymbol());
```

* Revenons sur le contrôle du nombre de chefs. On peut le mettre dans le postproc de la groupe POPULATION000 (le roster).

```
PROC POPULATION000

numeric nombrePersonnes = totocc(POPULATION000);

// Assurer qu'il y a un chef de ménage et pas plus
// pour les maisons habitées
if nombrePersonnes > 0 then

	// Compter le nombre de chefs (indivus avec LIEN_DE_PARENTE = 1)
	numeric i;
	numeric nombreChefs = 0;
	do i = 1 while i <= nombrePersonnes
		if LIEN_DE_PARENTE(i) = 1 then
			nombreChefs = nombreChefs + 1;
		endif;
	enddo;

	if nombreChefs = 0 then
		errmsg("Le chef de ménage n'a pas été saisie. Il faut un chef pour chaque ménage.");
		reenter LIEN_DE_PARENTE(1);
	elseif nombreChefs > 1 then
		errmsg("Le ménage a %d chefs. Il faut un seul chef pour chaque ménage.", nombreChefs);
		reenter LIEN_DE_PARENTE(1);
	endif;
endif;
```

* Avec la fonction *count* nous pouvons éviter le boucle:

```
numeric nombreChefs = count(POPULATION000 where LIEN_DE_PARENTE = 1);
```

* Ajouter une contrôle pour assurer que le chef du ménage est plus âgée que ses enfants. An ajoute la logique suivante au postproc du roster:

```
	// Trouver l'index du chef 
	numeric indexChef;
	numeric i;
	do i = 1 while i <= nombrePersonnes 
		if LIEN_DE_PARENTE(i) = 1 then
			indexChef = i;
			break;
		endif;
	enddo;
	
	// Comparer l'age de chaque enfant avec l'âge du chef
	do i = 1 while i <= nombrePersonnes 
		if LIEN_DE_PARENTE(i) = 3 and AGE(indexChef) < AGE(i) then
			errmsg("L'âge de l'enfant %s est %d mais l'âge du chef de ménage %s est %d. Le chef doit être plus âgé qui son enfant.",
					strip(NOM_ET_PRENOM(i)), AGE(i), strip(NOM_ET_PRENOM(indexChef)), AGE(indexChef))
					select("Corriger l'âge de " + strip(NOM_ET_PRENOM(i)), AGE(i),
						   "Corriger l'âge de " + strip(NOM_ET_PRENOM(indexChef)), AGE(indexChef),
						   "Corriger le lien de parenté de " + strip(NOM_ET_PRENOM(i)), LIEN_DE_PARENTE(i),
						   "Corriger le lien de parenté de " + strip(NOM_ET_PRENOM(indexChef)), LIEN_DE_PARENTE(indexChef));
		endif;
	enddo;
```

* Avec la fonction *seek* on peut enlever le premier boucle:

```
numeric indexChef = seek(LIEN_DE_PARENTE = 1);
```

Exercices
---------

1. Ajouter une variable pour l'heure de l'interview et la remplir automatiquement (utiliser la fonction systime())
2. Ajouter deux variables à l'enregistrement habitation après montant du loyer:
	* mode d'approvisionnement en eau (Eau courante à domicile, Eau courante chez le voisin, Citerne privée, Fontaine publique, Citerne publique, Forage/Puits, Rivière/Source)
	* distance d'approvisionnement en eau (0 à 998 mètres ou 999 si inconnu)
	* sauter distance d'approvisionnement si mode d'approvisionnement est Eau courante à domicile ou Citerne privée.
3. Sauter le nombre d'enfants nés vivants pour les hommes.
4. Ajouter un contrôle de l'âge et le lien de parenté pour assurer que l'âge de l'époux/épouse du chef est au moins 12 ans.
5. Ajouter de la logique pour assurer que l'état matrimoniale de l'époux/épouse du chef n'est pas célibataire ou veuf/veuve.
6. Afficher un message d'erreur si le chef du ménage a au moins un époux/une épouse dans le ménage et son état matrimoniale n'est pas mariée. 
7. Afficher un message d'erreur si le chef du ménage et un son époux/épouse sont de même sexe. Tester le contrôle avec les ménages polygames aussi.
