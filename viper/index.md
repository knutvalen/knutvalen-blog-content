VIPER is an implementation of clean architecture for iOS. This architectural pattern differs from the more commonly known patterns like MVC, MVVM and MVP in that a module is split into five different parts - each with its own specific responsibility. These are: View, Interactor, Presenter, Entity and Router.

We are now going to learn how to use the VIPER architecture for iOS by going through a fully working example.

## Implementation
The entire code we will be going through is available at GitHub, but you can read along here without checking it out. You can clone it with
```shell
git clone https://github.com/knutvalen/VIPER-architecture.git
```
The app is simple. It shows information about the light side and the dark side of the Star Wars universe. It has a navigation bar that you can navigate to the other side using buttons to go back and forth between the view controllers.
![alt text](https://github.com/knutvalen/knutvalen-blog-content/blob/master/viper/giphy.gif?raw=true)
