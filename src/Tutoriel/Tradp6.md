Obtenir des données à partir d'un serveur
Ce tutoriel ajoute les fonctionnalités suivantes de persistance des données à l'aide de HttpClient d'Angular :

Le service HeroService obtient les données des héros à l'aide de requêtes HTTP.
Les utilisateurs peuvent ajouter, modifier et supprimer des héros, et enregistrer ces modifications via HTTP.
Les utilisateurs peuvent rechercher des héros par nom.
Pour l'application exemple décrite sur cette page, consultez l'exemple en direct / téléchargez l'exemple.
Activer les services HTTP
HttpClient est le mécanisme d'Angular pour communiquer avec un serveur distant via HTTP.

Rendez HttpClient disponible partout dans l'application en deux étapes. Tout d'abord, ajoutez-le au AppModule racine en l'important :

src/app/app.module.ts (import de HttpClientModule)
content_copy
import { HttpClientModule } from '@angular/common/http';
Ensuite, toujours dans AppModule, ajoutez HttpClientModule au tableau des imports :

src/app/app.module.ts (extrait du tableau des imports)
content_copy
@NgModule({
imports: [
HttpClientModule,
],
})
Simuler un serveur de données
Cet exemple de tutoriel simule la communication avec un serveur de données distant en utilisant le module In-memory Web API.

Après avoir installé le module, l'application effectue des requêtes et reçoit des réponses du HttpClient. L'application ne sait pas que l'In-memory Web API intercepte ces requêtes, les applique à un magasin de données en mémoire et renvoie des réponses simulées.

En utilisant l'In-memory Web API, vous n'avez pas besoin de configurer un serveur pour apprendre à utiliser HttpClient.

IMPORTANT :
Le module In-memory Web API n'a rien à voir avec HTTP dans Angular.

Si vous lisez ce tutoriel pour apprendre à utiliser HttpClient, vous pouvez passer cette étape. Si vous suivez ce tutoriel en codant, restez ici et ajoutez maintenant l'In-memory Web API.

Installez le package In-memory Web API à partir de npm avec la commande suivante :

content_copy
npm install angular-in-memory-web-api --save
Générez la classe src/app/in-memory-data.service.ts avec la commande suivante :

content_copy
ng generate service InMemoryData
Remplacez le contenu par défaut de in-memory-data.service.ts par ce qui suit :

src/app/in-memory-data.service.ts
content_copy
import { Injectable } from '@angular/core';
import { InMemoryDbService } from 'angular-in-memory-web-api';
import { Hero } from './hero';

@Injectable({
providedIn: 'root',
})
export class InMemoryDataService implements InMemoryDbService {
createDb() {
const heroes = [
{ id: 12, name: 'Dr. Nice' },
{ id: 13, name: 'Bombasto' },
{ id: 14, name: 'Celeritas' },
{ id: 15, name: 'Magneta' },
{ id: 16, name: 'RubberMan' },
{ id: 17, name: 'Dynama' },
{ id: 18, name: 'Dr. IQ' },
{ id: 19, name: 'Magma' },
{ id: 20, name: 'Tornado' }
];
return {heroes};
}

// Remplace la méthode genId pour garantir qu'un héros a toujours un id.
// Si le tableau des héros est vide,
// la méthode renvoie le nombre initial (11).
// Si le tableau des héros n'est pas vide, la méthode renvoie le plus grand
// id de héros + 1.
genId(heroes: Hero[]): number {
return heroes.length > 0 ? Math.max(...heroes.map(hero => hero.id)) + 1 : 11;
}
}
Dans AppModule, importez le module HttpClientInMemoryWebApiModule et la classe InMemoryDataService que vous avez créée précédemment.

src/app/app.module.ts (imports de l'In-memory Web API)
content_copy
import { HttpClientInMemoryWebApiModule } from 'angular-in-memory-web-api';
import { InMemoryDataService } from './in-memory-data.service';
Après HttpClientModule, ajoutez HttpClientInMemoryWebApiModule au tableau des imports de AppModule et configurez-le avec InMemoryDataService.

src/app/app.module.ts (extrait du tableau des imports)
content_copy
HttpClientModule,

// Le module HttpClientInMemoryWebApiModule intercepte les requêtes HTTP
// et renvoie des réponses simulées du serveur.
// Supprimez-le lorsque un vrai serveur est prêt à recevoir les requêtes.
HttpClientInMemoryWebApiModule.forRoot(
InMemoryDataService, { dataEncapsulation: false }
)
La méthode de configuration forRoot() prend une classe InMemoryDataService qui alimente la base de données en mémoire.

Le fichier in-memory-data.service.ts prend la place de mock-heroes.ts. Ne supprimez pas mock-heroes.ts pour l'instant. Vous en aurez encore besoin pour quelques étapes supplémentaires de ce tutoriel.

Une fois que le serveur est prêt, détachez l'In-memory Web API pour que les requêtes de l'application puissent passer au serveur.

Héros et HTTP
Dans HeroService, importez HttpClient et HttpHeaders :

src/app/hero.service.ts (import des symboles HTTP)
content_copy
import { HttpClient, HttpHeaders } from '@angular/common/http';
Toujours dans HeroService, injectez HttpClient dans le constructeur, dans une propriété privée appelée "http".

src/app/hero.service.ts
content_copy
constructor(
private http: HttpClient,
private messageService: MessageService) { }
Remarquez que vous continuez à injecter MessageService, mais comme votre application l'appelle très fréquemment, enveloppez-le dans une méthode privée log() :

src/app/hero.service.ts
content*copy
/\*\* Enregistre un message de HeroService avec le MessageService */
private log(message: string) {
this.messageService.add(HeroService : ${message});
}
Définissez heroesUrl sous la forme :base/:collectionName avec l'adresse de la ressource des héros sur le serveur. Ici, la base est la ressource vers laquelle les requêtes sont effectuées, et collectionName est l'objet de données des héros dans in-memory-data-service.ts.

src/app/hero.service.ts
content_copy
private heroesUrl = 'api/heroes'; // URL vers l'API web
Obtenir des héros avec HttpClient
La méthode actuelle HeroService.getHeroes() utilise la fonction RxJS of() pour renvoyer un tableau de héros simulés sous la forme d'une Observable<Hero[]>.

src/app/hero.service.ts (getHeroes avec RxJs 'of()')
content_copy
getHeroes(): Observable<Hero[]> {
const heroes = of(HEROES);
return heroes;
}
Transformez cette méthode pour utiliser HttpClient comme suit :

src/app/hero.service.ts
content*copy
/\*\* Obtient les héros du serveur */
getHeroes(): Observable<Hero[]> {
return this.http.get<Hero[]>(this.heroesUrl)
}
Actualisez le navigateur. Les données des héros devraient être chargées avec succès depuis le serveur simulé.

Vous avez remplacé of() par http.get() et l'application continue de fonctionner sans autre modification, car les deux fonctions renvoient une Observable<Hero[]>.

Les méthodes HttpClient renvoient une seule valeur
Toutes les méthodes HttpClient renvoient une Observable RxJS d'une certaine valeur.

HTTP est un protocole de demande/réponse. Vous effectuez une demande, il renvoie une seule réponse.

En général, une observable peut renvoyer plusieurs valeurs au fil du temps. Une observable issue de HttpClient émet toujours une seule valeur, puis se termine, sans émettre à nouveau.

Cet appel particulier à HttpClient.get() renvoie une Observable<Hero[]>, qui est une observable de tableaux de héros. En pratique, elle ne renvoie qu'un seul tableau de héros.

HttpClient.get() renvoie les données de réponse
Par défaut, HttpClient.get() renvoie le corps de la réponse sous forme d'objet JSON non typé. En ajoutant le spécificateur de type facultatif, <Hero[]> , vous ajoutez des capacités de TypeScript, ce qui réduit les erreurs lors de la compilation.

L'API de données du serveur détermine la forme des données JSON. L'API de données Tour of Heroes renvoie les données des héros sous forme de tableau.

D'autres API peuvent enfouir les données que vous voulez dans un objet. Vous pourriez devoir extraire ces données en traitant le résultat Observable avec l'opérateur RxJS map().

Bien qu'elle ne soit pas discutée ici, il y a un exemple de map() dans la méthode getHeroNo404() incluse dans le code source de l'exemple.

Gestion des erreurs
Les choses peuvent mal tourner, en particulier lorsque vous obtenez des données à partir d'un serveur distant. La méthode HeroService.getHeroes() doit intercepter les erreurs et faire quelque chose d'adapté.

Pour intercepter les erreurs, vous "pipez" le résultat observable de http.get() à travers l'opérateur RxJS catchError().

Importez le symbole catchError depuis rxjs/operators, ainsi que d'autres opérateurs à utiliser plus tard.

src/app/hero.service.ts
content_copy
import { catchError, map, tap } from 'rxjs/operators';
Prolongez ensuite le résultat observable avec la méthode pipe() et ajoutez-lui l'opérateur catchError().

src/app/hero.service.ts
content_copy
getHeroes(): Observable<Hero[]> {
return this.http.get<Hero[]>(this.heroesUrl)
.pipe(
catchError(this.handleError<Hero[]>('getHeroes', []))
);
}
L'opérateur catchError() intercepte une observable qui a échoué. L'opérateur passe ensuite l'erreur à la fonction de gestion des erreurs.

La méthode handleError() suivante signale l'erreur, puis renvoie une valeur inoffensive pour que l'application continue de fonctionner.

Gestion des erreurs
La méthode handleError() suivante peut être utilisée par de nombreuses méthodes de HeroService, elle est donc généralisée pour répondre à leurs besoins différents.

Au lieu de gérer l'erreur directement, elle renvoie une fonction de gestionnaire d'erreur à catchError. Cette fonction est configurée avec le nom de l'opération qui a échoué et une valeur de retour sûre.

src/app/hero.service.ts
content_copy
/\*\*

Gère l'opération HTTP qui a échoué.

Laisse l'application continuer.

@param operation - nom de l'opération qui a échoué

@param result - valeur facultative à renvoyer en tant que résultat observable
\*/
private handleError<T>(operation = 'opération', result?: T) {
return (error: any): Observable<T> => {

javascript
Copy code
// TODO : envoyer l'erreur à une infrastructure de journalisation à distance
console.error(error); // journalisation dans la console

// TODO : meilleure transformation de l'erreur pour une consommation par l'utilisateur
this.log(`${operation} a échoué : ${error.message}`);

// Permet à l'application de continuer de fonctionner en renvoyant un résultat vide.
return of(result as T);
};
}
Après avoir signalé l'erreur dans la console, le gestionnaire construit un message convivial et renvoie une valeur sûre pour que l'application puisse continuer de fonctionner.

Écouter l'Observable
La méthode getHeros() écoute le flux de valeurs observables et envoie un message, en utilisant la méthode log(), à la zone de message en bas de la page.

L'opérateur RxJS tap() permet cette fonctionnalité en observant les valeurs observables, en faisant quelque chose avec ces valeurs et en les transmettant. Le rappel tap() n'accède pas directement aux valeurs elles-mêmes.

Voici la version finale de getHeroes() avec le tap() qui enregistre l'opération.

src/app/hero.service.ts
content*copy
/\* Obtient les héros du serveur */
getHeroes(): Observable<Hero[]> {
return this.http.get<Hero[]>(this.heroesUrl)
.pipe(
tap(\* => this.log('héros récupérés')),
catchError(this.handleError<Hero[]>('getHeroes', []))
);
}
Obtenir un héros par son identifiant
La plupart des API Web prennent en charge une demande de type "obtenir par identifiant" sous la forme :baseURL/:id.

Ici, la base URL est l'URL des héros définie dans la section Héros et HTTP sous forme d'api/heroes, et l'identifiant (id) est le numéro du héros que vous souhaitez récupérer. Par exemple, api/heroes/11.

Mettez à jour la méthode getHero() de HeroService avec ce qui suit pour effectuer cette demande :

src/app/hero.service.ts
content*copy
/\* Obtient un héros par son identifiant. Renvoie 404 si l'identifiant n'est pas trouvé */
getHero(id: number): Observable<Hero> {
const url = ${this.heroesUrl}/${id};
return this.http.get<Hero>(url).pipe(
tap(\* => this.log(`héros récupéré, id=${id}`)),
catchError(this.handleError<Hero>(`getHero id=${id}`))
);
}
getHero() présente trois différences significatives par rapport à getHeroes() :

getHero() construit une URL de demande avec l'identifiant du héros souhaité.
Le serveur doit répondre avec un seul héros plutôt qu'un tableau de héros.
getHero() renvoie une Observable<Hero>, qui est une observable d'objets Hero plutôt qu'une observable de tableaux de héros.

Mettre à jour les héros
Modifiez le nom d'un héros dans la vue détaillée du héros. Au fur et à mesure que vous tapez, le nom du héros met à jour l'en-tête en haut de la page. Cependant, lorsque vous cliquez sur "Revenir en arrière", vos modifications sont perdues.

Si vous souhaitez que les modifications persistent, vous devez les enregistrer sur le serveur.

À la fin du modèle de détail du héros, ajoutez un bouton "Enregistrer" avec une liaison d'événement de clic qui invoque une nouvelle méthode de composant appelée "save()".

src/app/hero-detail/hero-detail.component.html (enregistrer)
<button type="button" (click)="save()">enregistrer</button>

Dans la classe HeroDetail du composant, ajoutez la méthode "save()" suivante, qui enregistre les modifications du nom du héros à l'aide de la méthode "updateHero()" du service de héros, puis revient à la vue précédente.

src/app/hero-detail/hero-detail.component.ts (enregistrer)
save(): void {
if (this.hero) {
this.heroService.updateHero(this.hero)
.subscribe(() => this.goBack());
}
}

Ajouter HeroService.updateHero()
La structure de la méthode "updateHero()" est similaire à celle de "getHeroes()", mais elle utilise "http.put()" pour enregistrer le héros modifié sur le serveur. Ajoutez ce qui suit au service de héros.

src/app/hero.service.ts (mise à jour)
/\*_ PUT : mettre à jour le héros sur le serveur _/
updateHero(hero: Hero): Observable<any> {
return this.http.put(this.heroesUrl, hero, this.httpOptions).pipe(
tap(\_ => this.log(`héros mis à jour, id=${hero.id}`)),
catchError(this.handleError<any>('updateHero'))
);
}

La méthode "HttpClient.put()" prend trois paramètres :

L'URL
Les données à mettre à jour, qui sont le héros modifié dans ce cas
Options
L'URL reste inchangée. L'API Web des héros sait quel héros mettre à jour en examinant l'ID du héros.

L'API Web des héros attend un en-tête spécial dans les requêtes d'enregistrement HTTP. Cet en-tête se trouve dans la constante "httpOptions" définie dans le service de héros. Ajoutez ce qui suit à la classe HeroService.

src/app/hero.service.ts
httpOptions = {
headers: new HttpHeaders({ 'Content-Type': 'application/json' })
};

Actualisez le navigateur, changez le nom d'un héros et enregistrez votre modification. La méthode "save()" dans HeroDetailComponent navigue vers la vue précédente. Le héros apparaît désormais dans la liste avec le nom modifié.

Ajouter un nouveau héros
Pour ajouter un héros, cette application n'a besoin que du nom du héros. Vous pouvez utiliser un élément <input> associé à un bouton "Ajouter".

Insérez ce qui suit dans le modèle de composant Heroes, après l'en-tête :

src/app/heroes/heroes.component.html (ajouter)

<div>
  <label for="nouveau-hero">Nom du héros : </label>
  <input id="nouveau-hero" #nomHero />

  <!-- (click) transmet la valeur de l'entrée à add() puis efface l'entrée -->

<button type="button" class="bouton-ajouter" (click)="add(nomHero.value); nomHero.value=''">
Ajouter un héros
</button>

</div>

En réponse à un événement de clic, appelez le gestionnaire de clic du composant, "add()", puis effacez le champ d'entrée pour qu'il soit prêt pour un autre nom. Ajoutez ce qui suit à la classe HeroesComponent :

src/app/heroes/heroes.component.ts (ajouter)
add(nom: string): void {
nom = nom.trim();
if (!nom) { return; }
this.heroService.addHero({ name } as Hero)
.subscribe(hero => {
this.heroes.push(hero);
});
}

Lorsque le nom donné n'est pas vide, le gestionnaire crée un objet basé sur le nom du héros. Le gestionnaire passe le nom de l'objet à la méthode "addHero()" du service.

Lorsque "addHero()" crée un nouvel objet, le rappel "subscribe()" reçoit le nouveau héros et l'ajoute à la liste des héros pour l'affichage.

Ajoutez la méthode "addHero()" suivante à la classe HeroService.

src/app/hero.service.ts (addHero)
/\*_ POST : ajouter un nouveau héros sur le serveur _/
addHero(hero: Hero): Observable<Hero> {
return this.http.post<Hero>(this.heroesUrl, hero, this.httpOptions).pipe(
tap((nouveauHero: Hero) => this.log(`héros ajouté, id=${nouveauHero.id}`)),
catchError(this.handleError<Hero>('addHero'))
);
}

"addHero()" diffère de "updateHero()" de deux manières :

Elle appelle "HttpClient.post()" au lieu de "put()"
Elle attend que le serveur crée un ID pour le nouveau héros, qu'elle renvoie dans l'Observable<Hero> à l'appelant

Actualisez le navigateur et ajoutez quelques héros.

Supprimer un héros
Chaque héros dans la liste des héros doit avoir un bouton de suppression.

Ajoutez l'élément de bouton suivant au modèle de composant Heroes, après le nom du héros dans l'élément <li> répété.

src/app/heroes/heroes.component.html
<button type="button" class="supprimer" title="supprimer le héros"
(click)="delete(hero)">x</button>

Le HTML pour la liste des héros devrait ressembler à ceci :

src/app/heroes/heroes.component.html (liste des héros)

<ul class="heroes">
  <li *ngFor="let hero of heroes">
    <a routerLink="/detail/{{hero.id}}">
      <span class="badge">{{hero.id}}</span> {{hero.name}}
    </a>
    <button type="button" class="supprimer" title="supprimer le héros"
      (click)="delete(hero)">x</button>
  </li>
</ul>

Pour positionner le bouton de suppression tout à droite de l'entrée du héros, ajoutez un peu de CSS provenant du code de révision final au fichier "heroes.component.css".

Ajoutez le gestionnaire "delete()" à la classe de composant.

src/app/heroes/heroes.component.ts (supprimer)
delete(hero: Hero): void

Recherche par nom
Dans cet exercice final, vous apprendrez à chaîner des opérateurs d'Observable ensemble pour réduire le nombre de requêtes HTTP similaires et économiser la bande passante réseau de manière efficace.

Ajouter une fonctionnalité de recherche de héros au tableau de bord
Lorsque l'utilisateur tape un nom dans une boîte de recherche, votre application effectue des requêtes HTTP répétées pour obtenir les héros filtrés par ce nom. Votre objectif est de n'émettre que le nombre de requêtes nécessaires.

HeroService.searchHeroes()
Commencez par ajouter une méthode "searchHeroes()" au service de héros.

src/app/hero.service.ts
/_ OBTENIR les héros dont le nom contient le terme de recherche _/
searchHeroes(term: string): Observable<Hero[]> {
if (!term.trim()) {
// si aucun terme de recherche, renvoyer un tableau de héros vide.
return of([]);
}
return this.http.get<Hero[]>(${this.heroesUrl}/?name=${term}).pipe(
tap(x => x.length ?
this.log(héros trouvés correspondant à "${term}") :
this.log(aucun héros correspondant à "${term}")),
catchError(this.handleError<Hero[]>('searchHeroes', []))
);
}
La méthode renvoie immédiatement un tableau vide s'il n'y a pas de terme de recherche. Le reste ressemble beaucoup à "getHeroes()", la seule différence significative étant l'URL, qui inclut une chaîne de requête avec le terme de recherche.

Ajouter la recherche au tableau de bord
Ouvrez le modèle de DashboardComponent et ajoutez l'élément de recherche de héros, <app-hero-search>, en bas du balisage.

src/app/dashboard/dashboard.component.html

<h2>Héros populaires</h2>
<div class="menu-heros">
  <a *ngFor="let hero of heroes"
      routerLink="/detail/{{hero.id}}">
      {{hero.name}}
  </a>
</div>
<app-hero-search></app-hero-search>
Ce modèle ressemble beaucoup au répéteur *ngFor dans le modèle HeroesComponent.

Pour que cela fonctionne, la prochaine étape consiste à ajouter un composant avec un sélecteur correspondant à <app-hero-search>.

Créer le composant HeroSearch
Exécutez "ng generate" pour créer un composant HeroSearch.

ng generate component hero-search
"ng generate" crée les trois fichiers du composant HeroSearch et ajoute le composant aux déclarations du AppModule.

Remplacez le modèle du composant HeroSearch par un <input> et une liste des résultats de recherche correspondants, comme suit.

src/app/hero-search/hero-search.component.html

<div id="composant-recherche">
  <label for="zone-recherche">Recherche de héros</label>
  <input #zoneRecherche id="zone-recherche" (input)="recherche(zoneRecherche.value)" />
  <ul class="resultat-recherche">
    <li *ngFor="let hero of heroes$ | async" >
      <a routerLink="/detail/{{hero.id}}">
        {{hero.name}}
      </a>
    </li>
  </ul>
</div>
Ajoutez des styles CSS privés au fichier hero-search.component.css comme indiqué dans la révision finale du code ci-dessous.
À mesure que l'utilisateur tape dans la zone de recherche, une liaison d'événement d'entrée appelle la méthode de recherche du composant avec la nouvelle valeur de la zone de recherche.

AsyncPipe
Le *ngFor répète les objets hero. Remarquez que le *ngFor itère sur une liste appelée heroes$, et non heroes. Le symbole $ est une convention qui indique que heroes$ est un Observable, et non un tableau.

src/app/hero-search/hero-search.component.html

<li *ngFor="let hero of heroes$ | async" >
Étant donné que *ngFor ne peut rien faire avec un Observable, utilisez le pipe | suivi de "async". Cela identifie le "AsyncPipe" d'Angular et s'abonne automatiquement à un Observable, de sorte que vous n'avez pas à le faire dans la classe du composant.
Modifiez la classe HeroSearchComponent
Remplacez la classe et les métadonnées du composant HeroSearch comme suit.

src/app/hero-search/hero-search.component.ts
import { Component, OnInit } from '@angular/core';

import { Observable, Subject } from 'rxjs';

import {
debounceTime, distinctUntilChanged, switchMap
} from 'rxjs/operators';

import { Hero } from '../hero';
import { HeroService } from '../hero.service';

@Component({
selector: 'app-hero-search',
templateUrl: './hero-search.component.html',
styleUrls: [ './hero-search.component.css' ]
})
export class HeroSearchComponent implements OnInit {
heroes$!: Observable<Hero[]>;
private searchTerms = new Subject<string>();

constructor(private heroService: HeroService) {}

// Ajouter un terme de recherche dans le flux observable.
recherche(term: string): void {
this.searchTerms.next(term);
}

ngOnInit(): void {
this.heroes$ = this.searchTerms.pipe(
// Attendre 300 ms après chaque frappe avant de considérer le terme
debounceTime(300),

scss
Copy code
// Ignorer le nouveau terme s'il est identique au terme précédent
distinctUntilChanged(),

// Basculer vers un nouvel observable de recherche à chaque changement de terme
switchMap((term: string) => this.heroService.searchHeroes(term)),
);
}
}
Remarquez la déclaration de heroes$ en tant qu'Observable :

src/app/hero-search/hero-search.component.ts
heroes$!: Observable<Hero[]>;
Définissez cela dans ngOnInit(). Avant cela, concentrez-vous sur la définition de searchTerms.

Le sujet RxJS searchTerms
La propriété searchTerms est un sujet RxJS.

src/app/hero-search/hero-search.component.ts
private searchTerms = new Subject<string>();

// Ajouter un terme de recherche dans le flux observable.
recherche(term: string): void {
this.searchTerms.next(term);
}
Un "sujet" est à la fois une source de valeurs observables et un Observable lui-même. Vous pouvez vous abonner à un "sujet" comme vous le feriez avec n'importe quel Observable.

Vous pouvez également envoyer des valeurs dans cet Observable en appelant sa méthode next(value), comme le fait la méthode "recherche()".

La liaison d'événement sur l'événement d'entrée de la zone de texte appelle la méthode "recherche()".

src/app/hero-search/hero-search.component.html
<input #zoneRecherche id="zone-recherche" (input)="recherche(zoneRecherche.value)" />
Chaque fois que l'utilisateur tape dans la zone de texte, la liaison appelle "recherche()" avec la valeur de la zone de texte en tant que terme de recherche. "searchTerms" devient un Observable émettant un flux constant de termes de recherche.

Chaîner les opérateurs RxJS
Passer un nouveau terme de recherche directement à "searchHeroes()" après chaque frappe d'utilisateur crée des requêtes HTTP excessives, ce qui sollicite les ressources du serveur et épuise les forfaits de données.

À la place, la méthode "ngOnInit()" fait passer l'observable "searchTerms" à travers une séquence d'opérateurs RxJS qui réduisent le nombre d'appels à "searchHeroes()". En fin de compte, cela renvoie un observable de résultats de recherche de héros opportuns, où chacun est un tableau de héros.

Voici un aperçu plus détaillé du code.

src/app/hero-search/hero-search.component.ts
this.heroes$ = this.searchTerms.pipe(
// Attendre 300 ms après chaque frappe avant de considérer le terme
debounceTime(300),

// Ignorer le nouveau terme s'il est identique au terme précédent
distinctUntilChanged(),

// Basculer vers un nouvel observable de recherche à chaque changement de terme
switchMap((term: string) => this.heroService.searchHeroes(term)),
);
Chaque opérateur fonctionne comme suit :

debounceTime(300) attend que l'écoulement de nouveaux événements de chaîne se mette en pause pendant 300 millisecondes avant de transmettre la dernière chaîne. Les requêtes ne sont pas susceptibles de se produire plus fréquemment que toutes les 300 ms.

distinctUntilChanged() garantit qu'une requête est envoyée uniquement si le texte de filtrage a changé.

switchMap() appelle le service de recherche pour chaque terme de recherche qui passe par debounce() et distinctUntilChanged(). Il annule et rejette les observables de recherche précédents, ne renvoyant que le dernier observable de service de recherche.

Avec l'opérateur "switchMap", chaque événement clé éligible peut déclencher un appel à la méthode "HttpClient.get()". Même avec une pause de 300 ms entre les requêtes, vous pourriez avoir de nombreuses requêtes HTTP en cours, et elles peuvent ne pas revenir dans l'ordre envoyé.

"switchMap()" préserve l'ordre d'origine des requêtes tout en ne renvoyant que l'observable de la méthode HTTP la plus récente. Les résultats des appels précédents sont annulés et rejetés.

Annuler un observable précédent de "searchHeroes()" n'annule pas réellement une requête HTTP en attente. Les résultats indésirables sont rejetés avant d'atteindre votre code d'application.

N'oubliez pas que la classe de composant ne s'abonne pas à l'observable "heroes$". C'est le rôle de "AsyncPipe" dans le modèle.

Essayez-le
Exécutez à nouveau l'application. Dans le tableau de bord, saisissez du texte dans la zone de recherche. Saisissez des caractères correspondant à des noms de héros existants et recherchez quelque chose comme ceci.

Champ de recherche de héros avec les lettres 'm' et 'a' ainsi que quatre résultats de recherche correspondant à la requête affichée dans une liste sous l'entrée de recherche.
