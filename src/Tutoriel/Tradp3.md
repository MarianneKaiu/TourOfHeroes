Créer un composant de fonctionnalité
Actuellement, le composant HeroesComponent affiche à la fois la liste des héros et les détails du héros sélectionné.

Conserver toutes les fonctionnalités dans un seul composant à mesure que l'application grandit n'est pas maintenable. Ce tutoriel divise les grands composants en sous-composants plus petits, chacun se concentrant sur une tâche ou un flux de travail spécifique.

La première étape consiste à déplacer les détails du héros dans un composant séparé et réutilisable appelé HeroDetailComponent, pour obtenir :

Un HeroesComponent qui présente la liste des héros.
Un HeroDetailComponent qui présente les détails d'un héros sélectionné.
Pour l'application d'exemple décrite dans cette page, voir l'exemple en direct / télécharger l'exemple.

Créer le HeroDetailComponent
Utilisez la commande ng generate pour créer un nouveau composant appelé hero-detail.

content_copy
ng generate component hero-detail
La commande génère les éléments suivants :

Crée un répertoire src/app/hero-detail.
À l'intérieur de ce répertoire, quatre fichiers sont créés :

Un fichier CSS pour les styles du composant.
Un fichier HTML pour le template du composant.
Un fichier TypeScript avec une classe de composant nommée HeroDetailComponent.
Un fichier de test pour la classe HeroDetailComponent.
La commande ajoute également le HeroDetailComponent en tant que déclaration dans le décorateur @NgModule du fichier src/app/app.module.ts.

Écrire le template
Coupez le code HTML pour les détails du héros depuis la fin du template du HeroesComponent et collez-le sur le contenu de base du template du HeroDetailComponent.

Le code HTML collé fait référence à un selectedHero. Le nouveau HeroDetailComponent peut présenter n'importe quel héros, pas seulement un héros sélectionné. Remplacez selectedHero par hero partout dans le template.

Une fois terminé, le template du HeroDetailComponent devrait ressembler à ceci :

src/app/hero-detail/hero-detail.component.html
content_copy

<div *ngIf="hero">

  <h2>Détails de {{hero.name | uppercase}}</h2>
  <div><span>id : </span>{{hero.id}}</div>
  <div>
    <label for="hero-name">Nom du héros : </label>
    <input id="hero-name" [(ngModel)]="hero.name" placeholder="nom">
  </div>

</div>
Ajouter la propriété @Input() hero
Le template du HeroDetailComponent est lié à la propriété hero du composant, qui est de type Hero.

Ouvrez le fichier de classe HeroDetailComponent et importez le symbole Hero.

src/app/hero-detail/hero-detail.component.ts (import Hero)
content_copy
import { Hero } from '../hero';
La propriété hero doit être une propriété d'entrée (Input property), annotée avec le décorateur @Input(), car le composant externe HeroesComponent la lie de cette manière.

content_copy
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
Modifiez l'instruction d'importation @angular/core pour inclure le symbole Input.

src/app/hero-detail/hero-detail.component.ts (import Input)
content_copy
import { Component, Input } from '@angular/core';
Ajoutez une propriété hero, précédée du décorateur @Input().

src/app/hero-detail/hero-detail.component.ts
content_copy
@Input() hero?: Hero;
C'est le seul changement que vous devez apporter à la classe HeroDetailComponent. Il n'y a pas d'autres propriétés. Il n'y a pas de logique de présentation. Ce composant ne fait que recevoir un objet héros via sa propriété hero et l'affiche.

Afficher le HeroDetailComponent
Le composant HeroesComponent affichait auparavant les détails du héros par lui-même, avant que vous ne supprimiez cette partie du template. Cette section vous guide pour déléguer la logique au HeroDetailComponent.

Les deux composants ont une relation parent/enfant. Le parent, HeroesComponent, contrôle l'enfant, HeroDetailComponent, en lui envoyant un nouveau héros à afficher chaque fois que l'utilisateur sélectionne un héros dans la liste.

Vous n'avez pas besoin de modifier la classe HeroesComponent, modifiez plutôt son template.

Mettre à jour le template du HeroesComponent
Le sélecteur du HeroDetailComponent est 'app-hero-detail'. Ajoutez un élément <app-hero-detail> vers la fin du template du HeroesComponent, là où la vue des détails du héros était auparavant.

Liez HeroesComponent.selectedHero à la propriété hero de l'élément comme ceci.

heroes.component.html (liaison HeroDetail)
content_copy
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
[hero]="selectedHero" est une liaison de propriété Angular.

C'est une liaison de données à sens unique depuis la propriété selectedHero du HeroesComponent vers la propriété hero de l'élément cible, qui est liée à la propriété hero du HeroDetailComponent.

Maintenant, lorsque l'utilisateur clique sur un héros dans la liste, selectedHero change. Lorsque selectedHero change, la liaison de propriété met à jour hero et le HeroDetailComponent affiche le nouveau héros.

Le template révisé du HeroesComponent devrait ressembler à ceci :

heroes.component.html
content_copy

<h2>Mes héros</h2>

<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <button [class.selected]="hero === selectedHero" type="button" (click)="onSelect(hero)">
      <span class="badge">{{hero.id}}</span>
      <span class="name">{{hero.name}}</span>
    </button>
  </li>
</ul>

<app-hero-detail [hero]="selectedHero"></app-hero-detail>
Actualisez le navigateur et l'application fonctionnera à nouveau comme avant.

Qu'est-ce qui a changé ?
Comme auparavant, chaque fois qu'un utilisateur clique sur le nom d'un héros, les détails du héros apparaissent en dessous de la liste des héros. Maintenant, c'est le HeroDetailComponent qui présente ces détails au lieu du HeroesComponent.

En refactorant le composant d'origine HeroesComponent en deux composants, vous obtenez des avantages, tant maintenant que dans le futur :

Vous avez réduit les responsabilités du HeroesComponent.

Vous pouvez faire évoluer le HeroDetailComponent en un éditeur de héros complet sans toucher au composant parent HeroesComponent.

Vous pouvez faire évoluer le HeroesComponent sans toucher à la vue des détails du héros.

Vous pouvez réutiliser le HeroDetailComponent dans le template d'un futur composant.
