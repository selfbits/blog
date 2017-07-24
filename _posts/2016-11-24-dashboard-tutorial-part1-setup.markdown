---
layout: post
title:  "Angular 2 Dashboard Tutorial - Part 1: Introduction and Setup"
author: Han Che, Klaus Welle
date:   2016-11-24 08:00:01 +0200
categories: blog tutorial angular2
tags: Angular2 TypeScript Tutorial Authentication Social
---

<div class="alert alert-danger" role="alert">
UPDATE (24. Juli 2017):
This article was published when Angular 2 was still unstable. Since then, the Angular team published two major (breaking) releases. Struggling with limited time and resources, we are not able to provide support for this version. We also deprecated our Platform V1 to be able to focus on the release of a stable Version 2 where we focus on a better user and tenant management. If you are interested in receiving news of our platform V2, feel free to subscribe to our newsletter at [Selfbits.io](https://www.selfbits.io?utm_source=blog&utm_medium=ang2tut_part1).
</div>

# Create your Angular 2 Dashboard

Admin Templates are awesome ...  and massive! You can create all sorts of web apps with it,
yet they often require a lot of __*read in*__ time. With the official release of Angular 2 in late August,
creating a dashboard app with it seems quite tempting.

This is a humble attempt to guide you throw the steps of creating such a web app,
using the current best & free Angular 2 admin template out there and to connect it to a awesome free backend service.

<div class="alert alert-info" role="alert">
It is assumed that you have at least some basic Angular 2 knowledge including <strong>http calls, services and components</strong>. The main purpose is to help you ease into a mighty Angular 2 admin template and to get your project started!
</div>

![ng2-admin](/images/setup-tutorial/ng2-admin.png)

## Tutorial

- [Part 1: Installation and Setup](dashboard-tutorial-part1-setup)
- [Part 2: Simple email sign up and sign in](dashboard-tutorial-part2-email-auth)
- [Part 3: Social (Facebook) sign up and sign in](dashboard-tutorial-part3-social-auth)


## Part 1: Installation & Setup

### Content

- [1. Install Node and NPM](#install-node-and-npm)
- [2. Install Global Node Modules](#install-global-node-modules)
- [3. Setup admin template](#setup-admin-template)
- [4. Setup Backend App](#setup-backend-app)
- [5. Configure AppModule](#configure-appmodule)

### 1. Install Node and NPM

Go to [nodejs.org](https://nodejs.org) and install the latest version on your pc/mac.

If you are mac user, please use [Homebrew](http://brew.sh) to install node.
It's easier to update node and brew sets up your rights to use 'npm -g' without sudo.
Inside your terminal type **brew install node**

Check if node and npm are installed by typing **node -v** and **npm -v** inside your terminal.

```
node -v
v6.7.0

npm -v
3.10.8
```

### 2. Install Global Node Modules


You can check your global installed node modules with
```
npm ls -g -depth=0
```

We need to install a couple of global dependencies. If you have them already skip this part.

* typescript
* typings
* git

We'll be using <img src="https://upload.wikimedia.org/wikipedia/commons/d/db/Npm-logo.svg" height="22" align="top"> to install these.

```
sudo npm install -g git typescript typings
```

### 3. Setup admin template

We'll be using [akveo's](https://github.com/akveo/ng2-admin) ng2-admin as a starter project. It's an awesome **FREE** Angular 2 admin theme template.

Create a project folder and enter it
```
mkdir projects && cd projects/
```

Go to [ng2-admin's Github site](https://github.com/akveo/ng2-admin) and download the file as zip to your project folder and unpack it. (Or use git to clone it).
Rename the folder if you like, mine will be **ng2-admin**, which I'll refer to as **project (root) folder** later on.

Using your terminal to go into your project folder ( cd ng2-admin/ ) and install the local dependencies defined in package.json by running

```
npm install
```

Once finished (its a bit heavy and should take around 1-2min), we are ready to go but first let's give the template a try

```
npm start
```
Once finished compiling go to your browser's address line and punch in

```
http://localhost:3000
```

It should look like the first screen shot above. Here's a [live version](http://akveo.com/ng2-admin)

Feel free to play around it a little bit! Here are the [documentations](https://akveo.github.io/ng2-admin/articles/001-getting-started/)



### 4. Setup Backend App

In order to connect our dashboard with a real backend system, we'll use [Selfbits](https://www.selfbits.io?utm_source=blog&utm_medium=ang2tut_part1). It's a great alternative to other backend service like firebase because unlike firebase, we can actually connect **our own database** to it.

It's totally free and we can set it up within 5min.

#### 4.1 Signup for a free account
Go to [Selfbits](https://www.selfbits.io?utm_source=blog&utm_medium=ang2tut_part1) and signup for a new account.
I've signed up with my Github account. Once logged in, you should see a familiar dashboard.

![selfbits-dashboard](/images/setup-tutorial/selfbits-admin-dashboard.png)

#### 4.2 Create a new app
Click on **Create new project** and give it any name. I've called mine **ng-admin-tutorial**

![selfbits-app-setting](/images/setup-tutorial/selfbits-create-project.png)

#### 4.3 Get your app credentials
Once the app is created click on the settings button (the small gear icon on the left side menu) to access your app credentials.

![selfbits-app-setting](/images/setup-tutorial/selfbits-app-setting.png)

You will need the **sub domain, app ip and app secret** to configure your http calls inside your root ngModule, which will be our last step in the setup.

### 5. Configure AppModule

This step is necessary in order to connect ng2-admin to the Selfbits app we've created in the previous step.

Go to **ng2-admin/src/app/app.module.ts** and add these codes **above** the @NgModule decorator.

```js
export interface AppConfig {
  BASE_URL:string,
  APP_ID:string,
  APP_SECRET:string
}

export const APPCONFIG:AppConfig = {
    BASE_URL: 'your app subdomain',
    APP_ID: 'your app id',
    APP_SECRET: 'your app secret',
};

```
Fill in the credentials found under Settings of your app from Selfbits and copy it accordingly to the APPCONFIG variable, be sure not to copy any white spaces!

That's it, we are ready to start. Feel free to browse the Selfibts dashboard and [continue](dashboard-tutorial-part2-email-auth) .
