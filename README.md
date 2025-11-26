## 2. Projets de la solution
- Diayma (projet principal)

## 3. la version SDK .NET utilisée est :
- .NET 2.0

## 6. Les 2 bugs trouvés :
- Il n'y pas de mouvements de stock des produits même après la validation d'une commande
- Lorsqu'on choisit l'espagnol comme langue, c'est le français qui est appliqué.


## 8. t les namespaces, classes et méthodes visités avant l’affichage des produits sur l’écran d’accueil 

| Ordre | Namespace                                 | Classe                              | Méthode principale exécutée                              | Rôle / Commentaire |
|-------|-------------------------------------------|-------------------------------------|-----------------------------------------------------------|---------------------|
| 1     | `P2FixAnAppDotNetCode`                    | `Program`                           | `Main(string[] args)`                                     | Point d’entrée de l’application |
| 2     | `Microsoft.AspNetCore.Builder`            | `WebApplicationFactory` / Host      | Création du host et appel `Startup`                       | |
| 3     | `P2FixAnAppDotNetCode`                    | `Startup`                           | `ConfigureServices(IServiceCollection services)`         | **Point d’arrêt (e) ligne 20** → Enregistrement DI + localisation |
| 4     | `P2FixAnAppDotNetCode`                    | `Startup`                           | `Configure(IApplicationBuilder app, IWebHostEnvironment env)` | Configuration du pipeline (static files, session, MVC) |
| 5     | `Microsoft.AspNetCore.Routing`            | Routage MVC                         | Matching de la route par défaut `{controller=Product}/{action=Index}` | Sélectionne `ProductController.Index()` |
| 6     | `P2FixAnAppDotNetCode.Controllers`       | `ProductController`                 | `Index()`                                                 | **Point d’arrêt (b) ligne 15** → Première méthode métier atteinte |
| 7     | `P2FixAnAppDotNetCode.Models.Services`    | `ProductService`                    | `GetAllProducts()`                                        | Logique métier |
| 8     | `P2FixAnAppDotNetCode.Models.Repositories`| `ProductRepository`                 | `GetAllProducts()`                                        | Récupération des données (in-memory) |
| 9     | `P2FixAnAppDotNetCode.Controllers`       | `ProductController`                 | `return View(products)`                                   | Retour du ViewResult |
| 10    | Razor Engine                              |                                     | Rendu de `Views/Product/Index.cshtml`                     | |
| 11    | Razor Engine                              |                                     | Rendu du Layout partagé `Views/Shared/_Layout.cshtml`     | |
| 12    | `P2FixAnAppDotNetCode.Components`        | `CartSummaryViewComponent`          | `InvokeAsync()`                                           | **Point d’arrêt (a) ligne 12** → appelé via `@await Component.InvokeAsync("CartSummary")` dans le layout |
| 13    | `P2FixAnAppDotNetCode.Components`        | `LanguageSelectorViewComponent`     | `InvokeAsync()`                                           | Composant de sélection de langue |
| 14    | Razor Engine                              |                                     | Génération finale du HTML complet                         | Envoi au navigateur |

## Ordre de déclenchement des points d’arrêt demandés (lors du chargement de la page d’accueil)

1. **Startup.cs ligne 20** → `ConfigureServices` ou `Configure` (dès le démarrage)
2. **ProductController.cs ligne 15** → entrée dans `Index()`
3. **CartSummaryViewComponent.cs ligne 12** → lors du rendu du layout (après le contrôleur)

> Les points d’arrêt dans `OrderController` et `CartController` **ne se déclenchent pas** à l’affichage initial des produits (ils sont utilisés plus tard : ajout au panier, validation de commande, etc.).

## Résumé 

Avant que les produits apparaissent dans le navigateur, l’exécution passe obligatoirement par :

1. `Startup.ConfigureServices()` → injection des services  
2. `ProductController.Index()` → récupération des produits  
3. `CartSummaryViewComponent.InvokeAsync()` → affiché dans le layout partagé  

**Première méthode 100 % métier atteinte** : `ProductController.Index()`  
**Tous les produits sont affichés** grâce au `return View(products)` + la vue `Index.cshtml`

