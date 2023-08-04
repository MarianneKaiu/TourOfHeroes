Afficher une liste de sélection

Ce tutoriel vous montre comment :

1. Étendre l'application Tour of Heroes pour afficher une liste de héros.
2. Permettre aux utilisateurs de sélectionner un héros et afficher les détails du héros.

Créer des héros fictifs

La première étape consiste à créer des héros à afficher.

Créez un fichier nommé mock-heroes.ts dans le répertoire src/app/. Définissez une constante HEROES sous forme d'un tableau de dix héros et exportez-la. Le fichier devrait ressembler à ceci.

src/app/mock-heroes.ts

```typescript
import { Hero } from "./hero";

export const HEROES: Hero[] = [
  { id: 12, name: "Dr. Nice" },
  { id: 13, name: "Bombasto" },
  { id: 14, name: "Celeritas" },
  { id: 15, name: "Magneta" },
  { id: 16, name: "RubberMan" },
  { id: 17, name: "Dynama" },
  { id: 18, name: "Dr. IQ" },
  { id: 19, name: "Magma" },
  { id: 20, name: "Tornado" },
];
```

Affichage des héros

Ouvrez le fichier de classe HeroesComponent et importez les HEROES fictifs.

src/app/heroes/heroes.component.ts (import HEROES)

```typescript
import { HEROES } from "../mock-heroes";
```

Dans la classe HeroesComponent, définissez une propriété de composant appelée heroes pour exposer le tableau HEROES pour la liaison.

src/app/heroes/heroes.component.ts

```typescript
export class HeroesComponent {
  heroes = HEROES;
}
```

Liste des héros avec \*ngFor

Ouvrez le fichier de template HeroesComponent et effectuez les changements suivants :

1. Ajoutez un élément <h2> en haut.
2. En dessous du <h2>, ajoutez un élément <ul>.
3. Dans l'élément <ul>, insérez un élément <li>.
4. Placez un élément <button> à l'intérieur du <li> qui affiche les propriétés d'un héros à l'intérieur d'éléments <span>.
5. Ajoutez des classes CSS pour styliser le composant.

Le template devrait ressembler à ceci :

heroes.component.html (template des héros)

```html
<h2>Mes héros</h2>
<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <button type="button">
      <span class="badge">{{hero.id}}</span>
      <span class="name">{{hero.name}}</span>
    </button>
  </li>
</ul>
```

Cela affiche une erreur car la propriété hero n'existe pas encore. Pour avoir accès à chaque héros individuel et les lister tous, ajoutez un \*ngFor au <li> pour itérer à travers la liste des héros :

```html
<li *ngFor="let hero of heroes"></li>
```

Le \*ngFor est la directive répétitrice d'Angular. Elle répète l'élément hôte pour chaque élément d'une liste.

La syntaxe dans cet exemple est la suivante :

```html
SYNTAXE DETAILS
<li>L'élément hôte. heroes Contient la liste des héros fictifs de la classe HeroesComponent, c'est-à-dire la liste des héros fictifs. hero Contient l'objet héros actuel pour chaque itération à travers la liste.</li>
```

N'oubliez pas de mettre l'astérisque \* devant ngFor. C'est une partie essentielle de la syntaxe.

Après le rafraîchissement du navigateur, la liste des héros apparaît.

ÉLÉMENTS INTERACTIFS

À l'intérieur de l'élément <li>, ajoutez un élément <button> pour envelopper les détails du héros, puis rendez le héros cliquable. Pour améliorer l'accessibilité, utilisez des éléments HTML intrinsèquement interactifs au lieu d'ajouter un écouteur d'événements à un élément non interactif. Dans ce cas, on utilise l'élément interactif <button> au lieu d'ajouter un événement à l'élément <li>.

Pour plus de détails sur l'accessibilité, voir l'accessibilité dans Angular.

Styliser les héros

La liste des héros devrait être attrayante et réagir visuellement lorsque les utilisateurs survolent et sélectionnent un héros dans la liste.

Dans le premier tutoriel, vous avez défini les styles de base pour l'ensemble de l'application dans styles.css. Cette feuille de style ne comprenait pas les styles pour cette liste de héros.

Vous pourriez ajouter plus de styles à styles.css et continuer à agrandir cette feuille de style au fur et à mesure que vous ajoutez des composants.

Vous pouvez préférer plutôt définir des styles privés pour un composant spécifique. Cela garde tout ce dont un composant a besoin, comme le code, le HTML et le CSS, ensemble au même endroit.

Cette approche facilite la réutilisation du composant ailleurs et permet de fournir l'apparence prévue du composant même si les styles globaux sont différents.

Vous pouvez définir des styles privés soit en ligne dans le tableau @Component.styles, soit dans des fichiers de feuilles de style identifiés dans le tableau @Component.styleUrls.

Lorsque ng generate a créé le composant HeroesComponent, il a créé une feuille de style heroes.component.css vide pour le composant HeroesComponent et l'a liée dans @Component.styleUrls comme ceci.

src/app/heroes/heroes.component.ts (@Component)

```typescript
@Component({
  selector: 'app-heroes',
  templateUrl: './heroes.component.html',
  styleUrls: ['./heroes.component.css']
})
```

Ouvrez le fichier heroes.component.css et collez-y les styles CSS privés pour le composant HeroesComponent à partir de l'examen final du code.

Les styles et les feuilles de style identifiés dans les métadonnées @Component sont propres à ce composant spécifique. Les styles de heroes.component.css s'appliquent uniquement au compos
