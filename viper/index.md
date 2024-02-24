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
* Router has a create() function

These are the key aspects for a VIPER module. There are more going on in these protocols that we will look into later, but for now this is all we need to know about the types to understand the basics of VIPER.
