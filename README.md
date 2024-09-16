## Memo

### Difference between singleton manager vs dependency manager. Can you explain differences?

Singleton Manager example:
```swift
class SingletonManager {
       var environment: EnviromentManager {
                get {
                  // singleton implementation
             }
        }
}
```


While its true that a singleton manager may look similar to a dependency container, there are some major differences, that are vital to the scalability of a project, when modularised.
As shown in this example above, we could have a SingletonManager and add variables for each of our dependencies. Then later on we could just access them. 
But the problem with this approach is, now SingletonManager needs to know about all our dependencies. Their interfaces (protocols) and the actual implementations. This makes the manager a heavy package, if it was to be moved to its own package. Which prevents us from having a lightweight dependency manager. Since it has to know the details of all other packages, it has to import them all, and then when we need to import the manager our modules, they will also be indirectly importing all these packages, and will take longer to build each module package, as if we didn't modularise them.

But the DependencyContainer we build here has no idea about the dependencies, their interfaces or implementation details. It is a very lightweight container with no package dependencies. So all our module packages can easily import it and interact with it. The container only knows about foundational types of ObjectIdentifier, Any and AnyObject.

Additionally, with a singleton manager that exposes everything, any module accessing it will have access to all dependencies, even the ones it doesn't need. But with dependency container's we can have a better control. Because the module can only access to a dependency that it knows about, via its interface protocol.

My research:
| Aspect              | Singleton Manager                         | Coordinator                                              |
|---------------------|-------------------------------------------|----------------------------------------------------------|
| Coupling            | Tight coupling to global state            | Loose coupling, modularized by feature                    |
| State Management    | Global, shared state across the app        | Localized, scoped to specific flows/features              |
| Dependency Handling | Centralized, harder to replace for testing | Localized, easy to mock or replace                        |
| Navigation          | Not suitable for managing navigation flows | Handles view creation and navigation easily               |
| Testability         | Harder to test in isolation                | Easier to test navigation and logic                       |
| Flexibility         | Less flexible, requires changes in many places | High flexibility and scalability                       |

### Singleton Manager 

```swift
final class SingletonManager {
    static let shared = SingletonManager()
    
    // Global services
    let homeService: HomeService
    let analyticsTracker: AnalyticsEventTracker

    private init() {
        self.homeService = HomeService() // Singleton holds direct reference
        self.analyticsTracker = AnalyticsEventTracker.shared
    }

    // Accessing shared instance
    func getHomeService() -> HomeService {
        return homeService
    }

    func getAnalyticsTracker() -> AnalyticsEventTracker {
        return analyticsTracker
    }
}

class HomeViewController: UIViewController {
    let homeService = SingletonManager.shared.getHomeService()
    let analyticsTracker = SingletonManager.shared.getAnalyticsTracker()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Use services directly
        homeService.fetchHomeData()
        analyticsTracker.trackEvent("HomeViewOpened")
    }
}
```

## Coordinator Pattern
```swift
final class HomeCoordinator {
    private let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        // Setup Home View with a ViewModel and push it to the Navigation stack
        let viewModel = HomeViewModel(homeService: HomeService(), analyticsTracker: AnalyticsEventTracker.shared)
        let homeView = HomeView(viewModel: viewModel)
        let hostingVC = UIHostingController(rootView: homeView)
        navigationController.setViewControllers([hostingVC], animated: false)
    }

    func pushSongDetail(_ song: Song) {
        let coordinator = SongDetailsCoordinator(navigationController: navigationController)
        let songDetailVC = coordinator.makeViewController(with: song)
        navigationController.pushViewController(songDetailVC, animated: true)
    }
}

final class RootCoordinator {
    private let navigationController = UINavigationController()

    func start() -> UIViewController {
        let homeCoordinator = HomeCoordinator(navigationController: navigationController)
        homeCoordinator.start()
        
        let tabBarController = UITabBarController()
        tabBarController.viewControllers = [navigationController]
        return tabBarController
    }
}
```
