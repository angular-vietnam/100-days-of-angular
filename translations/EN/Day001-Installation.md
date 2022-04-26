# Day 1: PREPARE THE WORKING ENVIRONMENT

## INTRODUCTION

In the recent years, Angular has remarkably improved and re-innovated its features and performance. These positive changes have helped this powerful framework to attract more organisations, including big names such as Apple, PayPal, Telegram, Forbes, Nike, etc. We, **Angular Vietnam**, recognised that it is the opportunistic time to embark on this project, **100 Days of Angular**. 

This series of tutorials aim at providing you a continuously intensive learning experience which not only inspire but also challenge you in your learning journey with Angular. In order to follow along, you're expected to be equipped with basic JavaScript and TypeScript knowlegde. If you already know JS, [this tutorial](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) from TS official documentation would help you to quickly get started with TS.

## PREREQUISITES

The essentials tools and utilities for your learning journey with Angular includes (but not limits to):

- **IDE/Text Editor**: Choose what you're comfortable with. At **Angular Vietnam**, our team recommends either Visual Studio Code (VS Code) or JetBrains WebStorm as they are equiped with features leveraging your developer experience such as smart code auto-completetion, advanced debugging tools, etc.  
- For readers choosing VS Code: These extensions suggested below will be tremendously helpful: 
  - [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template)
  - [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)
  - [ESLint/TSLint](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-tslint-plugin)
  - [Nx Console (optional)](https://marketplace.visualstudio.com/items?itemName=nrwl.angular-console)

## SETTING UP THE WORKSPACE

### NODEJS

First thing first, you need to download **NodeJS** on [`https://nodejs.org/en/download/`](https://nodejs.org/en/download/) and install it on your machine. It doesn't matter whether you choose long-term support (LTS) version or the Current one. At the time of this writing, the latest LTS version is Node 12, with which Angular 9 is compatible.

If you're familiar with terminal environment, you should use [**NVM**](https://github.com/nvm-sh/nvm) to install and manage multiple versions of NodeJS in cases you need to work with multiple projects requiring different versions of NodeJS.

To check if NodeJS is successfully installed on your machine, execute these command lines in your OS' terminal application (by default, `Terminal` if you're using MacOS/Linux, or `Command Prompt`/`Powershell` if you're using Windows):

```bash
node -v
npm -v
```

If you can see version number displayed, it means that `NodeJS` and `NPM CLI` are ready to be used.

### ANGULAR CLI

For development of an Angular project, **Angular CLI** is an essential utility to manage the project from terminal environment. One way to install Angular CLI is to use NPM CLI by running this command line in your OS' terminal application:

```bash
npm install -g @angular/cli@latest
```

To check if Angular CLI is properly installed, use this command:
```bash
ng version
```

At the time of this writing, Angular CLI is at version 9.

A few side notes:

- If you're using Windows, there's a chance that you have to install either `Python` or `windows-build-tools` to get SCSS running properly for the upcoming projects.
- If you encountered the error `'ng' is not recognized as an internal or external command.` after executing `ng version`, you need to add `npm global` into your OS' `PATH` global variable.
- If you executed these commands on `Powershell`, you may encounter this error:

`File C:\Users\< username >\AppData\Roaming\npm\ng.ps1 cannot be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.`

In this case, you have to `enable policy` to run commands. To enable it, you open `Powershell` as Administrator and execute this command `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine` or follow the link in error message for more details.

## APPLICATION INITIALISATION

After you've done with these above steps, run this command to initialise an Angular application:

```bash
ng new <your-app-name>
```

For example, `ng new angular100-d-o-c`

You have to provide your answers for questions regarding concerns about your application's routing, style config as listed below:

- `Would you like to add Angular routing?`
- `Which stylesheet format would you like to use?`
  
At this point, you may want to accept the default options (`Y`for question about routing and `SCSS` for question aboutstyling).

After this step, you can use your prefer text editor/IDE toopen and edit the generated Angular application's source code.
  
To run the app, execute this command in your project'sdirectory:

```bash
ng serve
```

By default, the app runs at port 4200. You can change which port to run the app by using command:

```bash
ng serve --port=other-port-number
```
For instance: `ng serve --port=9000`

The working application is served at the address `http://localhost:4200/`, which can be viewed in your browser.

We're done for the first day. See you in the day 2's tutorial.

## Youtube Video

[Vietnamese]
https://youtu.be/NS6P1fpU77o

## HASHTAGS

`#100DaysOfCodeAngular` `#100DaysOfCode` `#AngularVietNam100DoC_Day1`

## REFERENCES

- [https://angular.io/guide/setup-local](https://angular.io/guide/setup-local)
- [https://angular.io/tutorial/toh-pt0](https://angular.io/tutorial/toh-pt0)

## AUTHORS

- [Tiep Phan](https://github.com/tieppt)
- Translator: [Minh Tu Hoang](https://github.com/m1nhtu99-hoan9)