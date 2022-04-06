# React Native

## environment setup

[Setting up the development environment · React Native](https://reactnative.dev/docs/environment-setup)

##### Create a new application

```
npx react-native init AwesomeProject
```

```
npx react-native init AwesomeTSProject --template react-native-template-typescript
```



##### run application

[Step 1: Start Metro](https://reactnative.dev/docs/environment-setup#step-1-start-metro)

```shell
npx react-native start
```

[Step 2: Start your application](https://reactnative.dev/docs/environment-setup#step-2-start-your-application)

Let Metro Bundler run in its own terminal. Open a new terminal inside your React Native project folder. Run the following:

```shell
npx react-native run-ios
```



## basics

Basic components [Core Components and Native Components · React Native](https://reactnative.dev/docs/intro-react-native-components)

Event handling [Handling Touches · React Native](https://reactnative.dev/docs/handling-touches)



## navigation

[Getting started | React Navigation](https://reactnavigation.org/docs/getting-started)

> **Error**: Invariant Violation: requireNativeComponent: "RNSScreenStackHeaderConfig" was not found in the UIManager
>
> **Solution**: https://reactnavigation.org/docs/getting-started#installing-dependencies-into-a-bare-react-native-projecthttps://reactnavigation.org/docs/getting-started#installing-dependencies-into-a-bare-react-native-project):
>
> From React Native 0.60 and higher, [linking is automatic](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md). So you **don't need to run** `react-native link`.
>
> If you're on a Mac and developing for iOS, you need to install the pods (via [Cocoapods](https://cocoapods.org/)) to complete the linking.
>
> ```sh
> npx pod-install ios
> ```



`<NavigationContainer>`必须为被export的根元素



路由方法：

- `navigation.navigate('RouteName')` pushes a new route to the native stack navigator if it's not already in the stack, otherwise it jumps to that screen.
- We can call `navigation.push('RouteName')` as many times as we like and it will continue pushing routes.
- The header bar will automatically show a back button, but you can programmatically go back by calling `navigation.goBack()`. On Android, the hardware back button just works as expected.
- You can go back to an existing screen in the stack with `navigation.navigate('RouteName')`, and you can go back to the first screen in the stack with `navigation.popToTop()`.
- The `navigation` prop is available to all screen components (components defined as screens in route configuration and rendered by React Navigation as a route).



## Hermes： JS engine for mobile apps

a facebook intro: [Hermes: A new open source JavaScript engine optimized for mobile apps](https://engineering.fb.com/2019/07/12/android/hermes/)

official doc: [Building and Running | Hermes](https://hermesengine.dev/docs/building-and-running)

use for RN: [Using Hermes · React Native](https://reactnative.dev/docs/hermes)



