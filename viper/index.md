VIPER is an implementation of clean architecture for iOS. This architectural pattern differs from the more commonly known patterns like MVC, MVVM and MVP in that a module is split into five different parts - each with its own specific responsibility. These are: View, Interactor, Presenter, Entity and Router.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/Viper-diagram.png?raw=true)
We are now going to learn how to use the VIPER architecture for iOS by going through a fully working example.

## Implementation
The entire code we will be going through is available at GitHub, but you can read along here without checking it out. You can clone it with
```shell
git clone https://github.com/knutvalen/VIPER-architecture.git
```
The app is simple. It shows information about the light side and the dark side of the Star Wars universe. It has a navigation bar that you can navigate to the other side using buttons to go back and forth between the view controllers.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/giphy.gif?raw=true)
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/folderStructure.png?raw=true)
We have two VIPER modules in our app: the light-side screen and the dark-side screen. In our LightSideScreen module we have a file called LightSideScreenTypes.swift and a folder for each of the VIPER parts of our module - View, Interactor, Presenter, Entity and Router. Inside our LightSideScreenTypes.swift file we have defined a protocol for each VIPER part:
```swift
import UIKit

protocol LightSideScreenViewType {
    var presenter: LightSideScreenPresenterType? { get set }
    func set(title: String?)
    func set(jediCode: LightSideScreenEntityType.JediCode?)
    func set(loading isLoading: Bool)
    func refreshJediList()
}

protocol LightSideScreenInteractorType {
    var entity: LightSideScreenEntityType? { get set }
    var webService: WebServiceType? { get set }
    func getCode(completionHandler: @escaping JediCodeResponse, useCache: Bool)
    func getJediList(completionHandler: @escaping JediListResponse, useCache: Bool)
}

protocol LightSideScreenPresenterType {
    var view: LightSideScreenViewType? { get set }
    var interactor: LightSideScreenInteractorType? { get set }
    var router: LightSideScreenRouterType? { get set }
    var jediList: [LightSideScreenEntityType.Jedi]? { get }
    func viewDidAppear()
    func viewDidDisappear()
    func onDarkSideSelected()
}

protocol LightSideScreenEntityType {
    typealias JediCode = CodeModel
    typealias Jedi = JediModel
    typealias Error = ErrorResponse
    var jediCode: JediCode? { get set }
    var jediList: [Jedi]? { get set }
}

protocol LightSideScreenRouterType {
    static func create() -> UIViewController
    func routeToDarkSideScreen(from view: LightSideScreenViewType?)
}

typealias JediCodeResponse = (_ jediCode: LightSideScreenEntityType.JediCode?, _ error: LightSideScreenEntityType.Error?) -> Void
typealias JediListResponse = (_ jediList: [LightSideScreenEntityType.Jedi]?, _ error: LightSideScreenEntityType.Error?) -> Void
```
I like to call these protocols “types” because they define what a type of each VIPER part should implement. Here you can see that:

* View reference to the Presenter
* Interactor reference to the Entity and the web service
* Presenter reference to View, the Interactor and the Router
* Entity reference to the models or data structures
* Router has a `create()` function

These are the key aspects for a VIPER module. There are more going on in these protocols that we will look into later, but for now this is all we need to know about the types to understand the basics of VIPER.

## Router
The Router is responsible for navigating to other modules and acts as the single point of routing between its VIPER module and other VIPER modules. Between VIPER modules only the Routers are directly connected to each other. Now, let’s look at the Router implementation in LightSideScreenRouter.swift.
```swift
import UIKit

class LightSideScreenRouter: LightSideScreenRouterType {
    ...
}
```
All VIPER parts have its own type, and here you can see that the LightSideScreenRouter inherits from the LightSideScreenRouterType. In the create() function of the LightSideScreenRouter the VIPER module is wired up and returns its View.
```swift
class LightSideScreenRouter: LightSideScreenRouterType {
    static func create() -> UIViewController {
        let storyboard = UIStoryboard(name: "LightSideScreenView", bundle: .main)
        
        if let view = storyboard.instantiateViewController(withIdentifier: "LightSideScreenView")
            as? LightSideScreenView
        {
            let interactor = LightSideScreenInteractor()
            let presenter = LightSideScreenPresenter()
            let entity = LightSideScreenEntity()
            let router = LightSideScreenRouter()
            
            view.presenter = presenter
            interactor.entity = entity
            interactor.webService = FakeWebService()
            presenter.view = view
            presenter.interactor = interactor
            presenter.router = router
            
            return view
        }
        
        return UIViewController()
    }
    
    func routeToDarkSideScreen(from view: LightSideScreenViewType?) {
        if let view = view as? UIViewController,
            let navigationController = view.navigationController
        {
            navigationController.pushViewController(
                DarkSideScreenRouter.create(),
                animated: true
            )
        }
    }
    
}
```
We can now use this module as our initial screen by configuring it in the SceneDelegate. Opening the SceneDelegate at the project root level we can find the scene(_:willConnectTo:options:) function.
```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?

    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        guard let windowScene = scene as? UIWindowScene else { return }
        
        let navigationController = UINavigationController(
            rootViewController: LightSideScreenRouter.create()
        )
        
        let window = UIWindow(windowScene: windowScene)
        window.rootViewController = navigationController
        window.makeKeyAndVisible()
        self.window = window
    }

}
```
This is where we use our Router’s create() function as the root view controller for the app’s navigation controller.

## View
The next VIPER part we will look into is the View. The View is responsible for showing whatever the Presenter tells it to and it is responsible for passing events like button clicks to the Presenter. The View implementation is under the folder Screens/LightSideScreen/View.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/viewStructureImage.png?raw=true)
Here you’ll find the files LightSideScreenView.swift and LightSideScreenView.storyboard that together implements the View. The LightSideScreenView class implements UIViewController, UICollectionViewDataSource and UICollectionViewDelegateFlowLayout for showing its list of Jedi, and the LightSideScreenViewType for implementing its VIPER type.
```swift
import UIKit

class LightSideScreenView: UIViewController {
    ...
}

extension LightSideScreenView: LightSideScreenViewType {
    ...
}

extension LightSideScreenView: UICollectionViewDataSource {
    ...   
}

extension LightSideScreenView: UICollectionViewDelegateFlowLayout {
    ...
}
```
When the Router returns its view controller in its create() function, the view controller’s viewDidLoad() function will be invoked. Next, the viewDidAppear(_:) function will be invoked and this is where we invoke the Presenter’s viewDidAppear() function.
```swift
override func viewDidLoad() {
    ...
}

override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    presenter?.viewDidAppear()
}
```
This is an important step in the VIPER module lifecycle because this is where we “start” the Presenter.

## Presenter
So, we have loaded our View and started our Presenter, but the View has no data to show. Now it’s time for the Presenter to do its thing. The Presenter is responsible for acquiring the data needed for the View to display and it is responsible for using the Router.
```swift
func viewDidAppear() {
    view?.set(loading: true)
    interactorIsLoading = ["onJediCode": true, "onJediList": true]
    interactor?.getCode(completionHandler: onJediCode, useCache: true)
    interactor?.getJediList(completionHandler: onJediList, useCache: true)
}
```
In the Presenter function viewDidAppear() we first start a loading animation on the View, then we ask the Interactor to get the data we need to display. Specifically we ask for the Jedi code and the list of Jedi masters, both with its own completion handlers, and we ask for cached data if it’s available. We’ll look into what’s going on in the Interactor soon, but for now all the Presenter is concerned about is waiting for the data to arrive in its completion handlers.
```swift
private func onJediCode(
    _ jediCode: LightSideScreenEntityType.JediCode?,
    _ error: LightSideScreenEntityType.Error?)
{
    onLoaded(function: "onJediCode")
    
    if let jediCode = jediCode {
        view?.set(jediCode: jediCode)
    }
}

private func onJediList(
    _ jediList: [LightSideScreenEntityType.Jedi]?,
    _ error: LightSideScreenEntityType.Error?)
{
    onLoaded(function: "onJediList")
    
    if let jediList = jediList {
        lightSideScreenJediList = jediList
        view?.refreshJediList()
    }
}
```
The completion handlers onJediCode(_:_:) and onJediList(_:_:), are functions that takes two parameters - an optional model (the data we want) and an optional error. The model will be not nil if the data was successfully loaded. So we use this optional model by safely unwrapping and adding it to our View with
```swift
view?.set(jediCode: jediCode)
```
and by invoking a refresh of the list of Jedi masters with
```swift
view?.refreshJediList()
```
When all the data is loaded we finish the loading animation on the View so that the data can be displayed:
```swift
private func onLoaded(function: String) {
    ...
    view?.set(loading: false)
    ...
}
```
The Presenter is also reacting to user events. When the user clicks the “Dark side” button in the navigation bar, it will invoke the View’s onDarkSideSelected(_:) function that first invokes the Presenter’s onDarkSideSelected() function that in turn will invoke the Router’s routeToDarkSideScreen(from:) function.
```swift
func onDarkSideSelected() {
    router?.routeToDarkSideScreen(from: view)
}
```
This will push the dark-side VIPER module’s view controller onto the navigation stack.

## Entity & Interactor
We’ve looked into what the Router, View and Presenter is doing, so now it’s time to see what is going on in the last two VIPER parts - Entity and Interactor. The Interactor is responsible for interacting with API’s - typically through network HTTP requests - to load the data needed for the Presenter to handle.
```swift
class LightSideScreenInteractor {
    ...
}

extension LightSideScreenInteractor: LightSideScreenInteractorType {
    ...
    
    func getJediList(completionHandler: @escaping JediListResponse, useCache: Bool) {
        if let cache = entity?.jediList,
            useCache == true
        {
            completionHandler(cache, nil)
            return
        }
        
        webService?.request(path: "light-side-service/jedi-list", method: "GET") {
            (result: Result<[LightSideScreenEntityType.Jedi]?, LightSideScreenEntityType.Error>) in
            switch result {
            case let .success(jediList):
                self.entity?.jediList = jediList
                completionHandler(jediList, nil)
                
            case let .failure(error):
                completionHandler(nil, error)
            }
        }
    }
    
    ...
}
```
When the Interactor gets data, it decodes it to an entity type and caches it in the Entity:
```swift
case let .success(jediList):
    self.entity?.jediList = jediList
    completionHandler(jediList, nil)
```
An entity type is a typealias for a data structure/model referenced in the Entity:
```swift
protocol LightSideScreenEntityType {
    typealias JediCode = CodeModel
    typealias Jedi = JediModel
    typealias Error = ErrorResponse
    var jediCode: JediCode? { get set }
    var jediList: [Jedi]? { get set }
}
```
The Entity is responsible for keeping references to models and storing the cache of the models used by the VIPER module. This enables us to use the models indirectly in all parts of the VIPER module by referencing to the Entity’s entity types. The next time the Presenter asks the Interactor for data, it can first look into the Entity cache and see if there is available data present and return this instantly instead of doing a lengthy network request:
```swift
if let cache = entity?.jediList,
    useCache == true
{
    completionHandler(cache, nil)
    return
}
```

## Wrapping up
And that’s about all there is to using VIPER in an iOS app 🎉

Navigating to the dark-side screen using the navigation bar will start the other VIPER module in the app - which is very similar to the light-side screen when we look at the structural flow of the code. The View is a little different and there are other entity types here, but everything else should be familiar.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/testsStructureImage.png?raw=true)
The VIPER architectural pattern can make unit testing your code a delightful experience. You can easily write unit tests for any part of a VIPER module. By mocking and stubbing the other parts you can inject them as dependencies in the VIPER part (or system) under test. I added a complete set of unit tests to our app, specifically testing the two VIPER modules. There’s also a basic UI test included. I managed to get as much as 98% test coverage with these tests.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/testCoverageImage.png?raw=true)
You can run these tests in Xcode by hitting `command + U`.
