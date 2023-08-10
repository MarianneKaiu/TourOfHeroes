Les composants ne devraient pas récupérer ou enregistrer directement des données, et ils ne devraient certainement pas présenter délibérément de fausses données. Ils devraient se concentrer sur la présentation des données et déléguer l'accès aux données à un service.

Ce tutoriel crée un HeroService que toutes les classes de l'application peuvent utiliser pour obtenir des héros. Au lieu de créer ce service avec le mot-clé "new", utilisez l'injection de dépendance qu'Angular prend en charge pour l'injecter dans le constructeur de HeroesComponent.

Les services sont un excellent moyen de partager des informations entre des classes qui ne se connaissent pas. Créez ensuite un MessageService et injectez-le à ces deux endroits.

Injectez-le dans HeroService, qui utilise le service pour envoyer un message.
Injectez-le dans MessagesComponent, qui affiche ce message et affiche également l'identifiant lorsque l'utilisateur clique sur un héros.

Créez le HeroService
Exécutez ng generate pour créer un service appelé "hero".

ng generate service hero
La commande génère une classe HeroService de base dans src/app/hero.service.ts comme suit :

src/app/hero.service.ts (nouveau service)
import { Injectable } from '@angular/core';

@Injectable({
providedIn: 'root',
})
export class HeroService {

constructor() { }

}
Services injectables (@Injectable())
Remarquez que le nouveau service importe le symbole Injectable d'Angular et annote la classe avec le décorateur @Injectable(). Cela marque la classe comme participant au système d'injection de dépendances. La classe HeroService va fournir un service injectable et peut également avoir ses propres dépendances injectées. Elle n'a pas encore de dépendances.

Le décorateur @Injectable() accepte un objet de métadonnées pour le service, de la même manière que le décorateur @Component() le faisait pour les classes de composants.

Obtenir les données des héros
Le HeroService pourrait obtenir des données de héros de n'importe où, comme un service Web, un stockage local ou une source de données fictives.

Le fait de supprimer l'accès aux données des composants signifie que vous pouvez changer d'avis sur l'implémentation à tout moment, sans toucher aux composants. Ils ne savent pas comment le service fonctionne.

L'implémentation de ce tutoriel continue de fournir des héros fictifs (mock).

Importez les classes Hero et HEROES.

src/app/hero.service.ts
import { Hero } from './hero';
import { HEROES } from './mock-heroes';
Ajoutez une méthode getHeroes pour renvoyer les héros fictifs.

src/app/hero.service.ts
getHeroes(): Hero[] {
return HEROES;
}
Fournir le HeroService
Vous devez rendre le HeroService disponible dans le système d'injection de dépendances avant qu'Angular puisse l'injecter dans HeroesComponent en enregistrant un fournisseur. Un fournisseur est quelque chose qui peut créer ou fournir un service. Dans ce cas, il instancie la classe HeroService pour fournir le service.

Pour vous assurer que le HeroService peut fournir ce service, enregistrez-le avec l'injecteur. L'injecteur est l'objet qui choisit et injecte le fournisseur là où l'application en a besoin.

Par défaut, ng generate service enregistre un fournisseur avec l'injecteur racine pour votre service en incluant des métadonnées de fournisseur, c'est-à-dire providedIn: 'root' dans le décorateur @Injectable().

@Injectable({
providedIn: 'root',
})
Lorsque vous fournissez le service au niveau racine, Angular crée une seule instance partagée de HeroService et l'injecte dans n'importe quelle classe qui en fait la demande. L'enregistrement du fournisseur dans les métadonnées @Injectable permet également à Angular d'optimiser une application en supprimant le service s'il n'est pas utilisé.

Pour en savoir plus sur les fournisseurs, consultez la section Fournisseurs. Pour en savoir plus sur les injecteurs, consultez le guide sur l'injection de dépendances.

Le HeroService est maintenant prêt à être utilisé dans HeroesComponent.

Ceci est un exemple de code intérimaire qui vous permet de fournir et d'utiliser le HeroService. À ce stade, le code diffère du HeroService dans la version finale du code.

Mettre à jour HeroesComponent
Ouvrez le fichier de classe HeroesComponent.

Supprimez l'importation de HEROES, car vous n'en avez plus besoin. Importez plutôt HeroService.

src/app/heroes/heroes.component.ts (import HeroService)
import { HeroService } from '../hero.service';
Remplacez la définition de la propriété heroes par une déclaration.

src/app/heroes/heroes.component.ts
heroes: Hero[] = [];
Injectez le HeroService
Ajoutez un paramètre heroService privé de type HeroService au constructeur.

src/app/heroes/heroes.component.ts
constructor(private heroService: HeroService) {}
Le paramètre définit simultanément une propriété heroService privée et l'identifie comme un site d'injection de HeroService.

Lorsque Angular crée un HeroesComponent, le système d'injection de dépendances attribue au paramètre heroService l'instance singleton de HeroService.

Ajoutez getHeroes()
Créez une méthode pour récupérer les héros à partir du service.

src/app/heroes/heroes.component.ts
getHeroes(): void {
this.heroes = this.heroService.getHeroes();
}
Appelez-la dans ngOnInit()
Bien que vous puissiez appeler getHeroes() dans le constructeur, ce n'est pas la meilleure pratique.

Réservez le constructeur pour une initialisation minimale telle que le câblage des paramètres du constructeur aux propriétés. Le constructeur ne devrait rien faire. Il ne devrait certainement pas appeler une fonction qui effectue des requêtes HTTP vers un serveur distant, comme le ferait un véritable service de données.

À la place, appelez getHeroes() à l'intérieur du crochet de cycle de vie ngOnInit et laissez Angular appeler ngOnInit() à un moment approprié après la création d'une instance de HeroesComponent.

src/app/heroes/heroes.component.ts
ngOnInit(): void {
this.getHeroes();
}
Voir le résultat
Après le rafraîchissement du navigateur, l'application devrait fonctionner comme précédemment, affichant une liste de héros et une vue détaillée du héros lorsque vous cliquez sur son nom.

Données observables
La méthode HeroService.getHeroes() a une signature synchrone, ce qui implique que le HeroService peut récupérer les héros de manière synchrone. Le composant HeroesComponent consomme le résultat de getHeroes() comme s'il était possible de récupérer les héros de manière synchrone.

src/app/heroes/heroes.component.ts
content_copy
this.heroes = this.heroService.getHeroes();
Cette approche ne fonctionnera pas dans une véritable application qui utilise des appels asynchrones. Cela fonctionne actuellement car votre service renvoie synchronement des héros fictifs.

Si getHeroes() ne peut pas renvoyer immédiatement les données des héros, il ne doit pas être synchrone, car cela bloquerait le navigateur en attendant les données.

HeroService.getHeroes() doit avoir une signature asynchrone d'une certaine sorte.

Dans ce tutoriel, HeroService.getHeroes() renvoie un Observable afin de pouvoir utiliser la méthode Angular HttpClient.get pour récupérer les héros et que HttpClient.get() renvoie un Observable.

Observable HeroService
Observable est l'une des classes clés de la bibliothèque RxJS.

Dans le tutoriel sur HTTP, vous pouvez voir comment les méthodes d'HttpClient d'Angular renvoient des objets Observable RxJS. Ce tutoriel simule la récupération de données depuis le serveur avec la fonction of() de RxJS.

Ouvrez le fichier HeroService et importez les symboles Observable et of de RxJS.

src/app/hero.service.ts (imports d'Observable)
content_copy
import { Observable, of } from 'rxjs';
Remplacez la méthode getHeroes() par ce qui suit :

src/app/hero.service.ts
content_copy
getHeroes(): Observable<Hero[]> {
const heroes = of(HEROES);
return heroes;
}
of(HEROES) renvoie un Observable<Hero[]> qui émet une seule valeur, le tableau de héros fictifs.

Le tutoriel sur HTTP vous montre comment appeler HttpClient.get<Hero[]>(), qui renvoie également un Observable<Hero[]> émettant une seule valeur, un tableau de héros depuis le corps de la réponse HTTP.

Abonnement dans HeroesComponent
La méthode HeroService.getHeroes qui renvoyait un Hero[] auparavant renvoie désormais un Observable<Hero[]>.

Vous devez ajuster votre application pour prendre en compte ce changement dans HeroesComponent.

Trouvez la méthode getHeroes et remplacez-la par le code suivant. Le nouveau code est affiché côte à côte avec la version actuelle pour comparaison.

heroes.component.ts (Observable)
heroes.component.ts (Original)
content_copy
getHeroes(): void {
this.heroService.getHeroes()
.subscribe(heroes => this.heroes = heroes);
}
Observable.subscribe() est la différence essentielle.

La version précédente attribue un tableau de héros à la propriété heroes du composant. L'attribution se fait de manière synchrone, comme si le serveur pouvait renvoyer instantanément les héros ou que le navigateur pouvait bloquer l'interface utilisateur en attendant la réponse du serveur.

Cela ne fonctionnera pas lorsque le HeroService effectue réellement des requêtes vers un serveur distant.

La nouvelle version attend que l'Observable émette le tableau de héros, ce qui peut se produire maintenant ou dans plusieurs minutes. La méthode subscribe() passe le tableau émis au rappel, qui définit la propriété heroes du composant.

Cette approche asynchrone fonctionne lorsque le HeroService demande les héros au serveur.

Afficher les messages
Cette section vous guide à travers les étapes suivantes :

Ajout d'un composant MessagesComponent qui affiche les messages de l'application en bas de l'écran
Création d'un MessageService injectable à l'échelle de l'application pour envoyer les messages à afficher
Injection du MessageService dans HeroService
Affichage d'un message lorsque HeroService récupère les héros avec succès
Créer MessagesComponent
Utilisez ng generate pour créer le composant MessagesComponent.

content_copy
ng generate component messages
ng generate crée les fichiers du composant dans le répertoire src/app/messages et déclare le MessagesComponent dans AppModule.

Modifiez le template de AppComponent pour afficher le MessagesComponent.

src/app/app.component.html
content_copy

<h1>{{title}}</h1>
<app-heroes></app-heroes>
<app-messages></app-messages>
Vous devriez voir le paragraphe par défaut de MessagesComponent en bas de la page.

Créer le MessageService
Utilisez ng generate pour créer le MessageService dans src/app.

content_copy
ng generate service message
Ouvrez MessageService et remplacez son contenu par ce qui suit.

src/app/message.service.ts
content_copy
import { Injectable } from '@angular/core';

@Injectable({
providedIn: 'root',
})
export class MessageService {
messages: string[] = [];

add(message: string) {
this.messages.push(message);
}

clear() {
this.messages = [];
}
}
Le service expose son cache de messages et deux méthodes :

Une pour ajouter() un message dans le cache.
Une autre pour clear() le cache.

Injectez-le dans HeroService
Dans HeroService, importez MessageService.

src/app/hero.service.ts (import MessageService)
content_copy
import { MessageService } from './message.service';
Modifiez le constructeur en ajoutant un paramètre qui déclare une propriété privée messageService. Angular injecte le singleton MessageService dans cette propriété lorsqu'il crée le HeroService.

src/app/hero.service.ts
content_copy
constructor(private messageService: MessageService) { }
Ceci est un exemple typique de scénario de service dans un service, dans lequel vous injectez le MessageService dans le HeroService qui est ensuite injecté dans HeroesComponent.

Envoyer un message depuis HeroService
Modifiez la méthode getHeroes() pour envoyer un message lorsque les héros sont récupérés.

src/app/hero.service.ts
content_copy
getHeroes(): Observable<Hero[]> {
const heroes = of(HEROES);
this.messageService.add('HeroService: fetched heroes');
return heroes;
}
Afficher le message depuis HeroService
Le MessagesComponent devrait afficher tous les messages, y compris le message envoyé par HeroService lorsqu'il récupère les héros.

Ouvrez MessagesComponent et importez MessageService.

src/app/messages/messages.component.ts (import MessageService)
content_copy
import { MessageService } from '../message.service';
Modifiez le constructeur en ajoutant un paramètre qui déclare une propriété publique messageService. Angular injecte le singleton MessageService dans cette propriété lorsqu'il crée le MessagesComponent.

src/app/messages/messages.component.ts
content_copy
constructor(public messageService: MessageService) {}
La propriété messageService doit être publique car vous allez la lier dans le template.

Angular ne lie que les propriétés publiques des composants.

Liez le MessageService
Remplacez le template de MessagesComponent créé par ng generate par le code suivant.

src/app/messages/messages.component.html
content_copy

<div *ngIf="messageService.messages.length">

  <h2>Messages</h2>
  <button type="button" class="clear"
          (click)="messageService.clear()">Clear messages</button>
  <div *ngFor='let message of messageService.messages'> {{message}} </div>

</div>
Ce template est directement lié à la propriété messageService du composant.

DÉTAILS
*ngIf Affiche la zone des messages uniquement s'il y a des messages à afficher.
*ngFor Affiche la liste des messages dans des éléments <div> répétés.
Liaison d'événements Angular Lie l'événement click du bouton à la méthode MessageService.clear().
Les messages seront mieux présentés après avoir ajouté les styles CSS privés dans messages.component.css, comme indiqué dans l'un des onglets "final code review" ci-dessous.

Ajouter MessageService à HeroesComponent
L'exemple suivant montre comment afficher l'historique à chaque fois que l'utilisateur clique sur un héros. Cela vous aidera pour la prochaine section sur le routage.

src/app/heroes/heroes.component.ts
content_copy
import { Component, OnInit } from '@angular/core';

import { Hero } from '../hero';
import { HeroService } from '../hero.service';
import { MessageService } from '../message.service';

@Component({
selector: 'app-heroes',
templateUrl: './heroes.component.html',
styleUrls: ['./heroes.component.css']
})
export class HeroesComponent implements OnInit {

selectedHero?: Hero;

heroes: Hero[] = [];

constructor(private heroService: HeroService, private messageService: MessageService) { }

ngOnInit(): void {
this.getHeroes();
}

onSelect(hero: Hero): void {
this.selectedHero = hero;
this.messageService.add(`HeroesComponent: Selected hero id=${hero.id}`);
}

getHeroes(): void {
this.heroService.getHeroes()
.subscribe(heroes => this.heroes = heroes);
}
}
Actualisez le navigateur pour voir la liste des héros, et défilez vers le bas pour voir les messages provenant du HeroService. Chaque fois que vous cliquez sur un héros, un nouveau message apparaît pour enregistrer la sélection. Utilisez le bouton "Clear messages" pour effacer l'historique des messages.
