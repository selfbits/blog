---
layout: post
title:  "Angular 2 Dashboard Tutorial - Part 3: Social (Facebook) Authentication"
date:   2016-10-18 13:58:05 +0200
categories: jekyll update
---

# Create your Angular 2 Dashboard - Part 1: Email Authentication

This part will cover a simple email sign up and sign in.
If you haven't done the setup yet, please go to Part 1 to do so.
Feel free to skip this part to go directly to social (facebook) authentication using Selfbits.

## Complete Content

- [Part 1: Installation and Setup](dashboard-tutorial-part1-setup)
- [Part 2: Simple email sign up and sign in](dashboard-tutorial-part2-email-auth)
- [Part 3: Social (Facebook) sign up and sign in](dashboard-tutorial-part3-social-auth)


## Part 3: Facebook Authentication

To optimize user experience, you **GOT** to provide social auth capabilities to your users. Instead of undergoing the burdensome OAuth process, maybe providers have developed libraries to make things easier for us.
So does Selfbits and we'll explore how by adding a facebook signup to our login page.

We'll be using Selfbit's SDK from the start this time. However for those who are interested I'll explain later what exactly the SDK is actually doing later!

### Content

- [Step 0: Setup selfbits-angular2-sdk](#step-0-setup-selfbits-angular2-sdk)
- [Step 1: Change default route to login page](#step-1-change-default-route-to-login-page)
- [Step 2: Setting up Facebook Auth on Selfbits](#step-2-setting-up-facebook-auth-on-selfbits)
- [Step 3: Setting up on Facebook](#step-3-setting-up-on-facebook)
- [Step 4: Configure Facebook Auth on Selfbits](#step-4-configure-facebook-auth-on-selfbits)
- [Step 5: Implement Facebook sign in](#step-5-implement-facebook-sign-in)
- [Summary](#summary)


### Step 0: Setup Selfbits-Angular2-SDK

We'll need to install, import, and initialize the library. If you've did the Bonus section from [part 2](dashboard-tutorial-part2-email-auth#bonus-selfbits-angular2-sdk), skip this Step.

If you've completed rigorously the complete part 2, you can go directly to [Step2: Setting up Facebook Auth on Selfbits](#step-2-setting-up-facebook-auth-on-selfbits)

#### Step 0.1: Install SDK

Go to the project root folder and type into your terminal

```
npm install selfbits-angular2-sdk --save
```

#### Step 0.2: Import SDK
Go to **ng2-admin/src/app/app.module.ts**

```js
import {SelfbitsAngularModule} from "selfbits-angular2-sdk";
```

#### Step 0.3: Configure app

You should have done that already in [part 1 setup](dashboard-tutorial-part1-setup#configure-appmodule)

```js
(...)
export const APPCONFIG = {
    BASE_URL: 'your app subdomain',
    APP_ID: 'your app id',
    APP_SECRET: 'your app secret',
};
(...)
@NgModule({...})
```

#### Step 0.4: Initialize the SDK

```js
@NgModule({
  imports: [
    (...)
    SelfbitsAngularModule.initializeApp(APPCONFIG)
  ]
})
```

That's it, we're ready to go!

### Step 1: Change default route to login page

**Note:** This is the exact same [step 1 from part 1](dashboard-tutorial-part1-setup#step-1-change-default-route-to-login-page), if you've done that and ng2-admin routes you to the login page, skip this step!


From your project root folder start your ng2-admin app by typing into your terminal

```
npm start
```
open up a browser and go to

```
http://localhost:3000/
```

**Note:** by default the port is set to 3000, change it here to some other port if needed.

**ng2-admin/config/webpack.dev.js**

```js
(...)
const PORT = process.env.PORT || 3000;
(...)
```

If you look at your address line of your browser, it will change directly to

```
http://localhost:3000/#/pages/dashboard
```

That's because it's the default route set by ng2-admin.
Let's change it in order to redirect to the login page located here.


Change **ng2-admin/src/app/app.routing.ts** to

```js
export const routes: Routes = [
  { path: '', redirectTo: 'login', pathMatch: 'full' }
];
```

wait for the change to set in and type into the address line of your browser

```
http://localhost:3000
```

Now it should redirect the url to the login page

```
http://localhost:3000/#/login
```

![login-page](/images/email-auth-tutorial/login-page.png)

<br>

### Step 2: Setting up Facebook Auth on Selfbits

Go to

1. [Selfbits Dashboard](https://admin.selfbits.io){:target="\_blank"}
2. Enter your project
3. Click on Authentication on the left-side menu
4. Click on Social

![social-auth-1](/images/social-auth-tutorial/1.png)

<br>

Next click on Add inside Facebook to add it to your Selfbits app.

![social-auth-2](/images/social-auth-tutorial/2.png)

It should display a modal asking for **Client Id** and **Client Secret**. Sound familiar?
Close the modal for now, go to settings and note down your Selfbits app suddomain (or copy it from your APPCONFIG BASE_URL).

Next we'll need to setup facebook.

<br>

### Step 3: Setting up on Facebook

#### Step 3.1: Sign in to Facebook Developer Center

![social-auth-3](/images/social-auth-tutorial/3.png)

Go to

1. [Facebook Developer Center](https://developers.facebook.com/)
2. Sign in with your Facebook account
3. Click on **My Apps** in the navigation bar
4. Click on the button **Create a New App**
5. Select "Website" as platform.

<br>

#### Step 3.2:  Create new Facebook app
![social-auth-4](/images/social-auth-tutorial/4.png)

Next click on **Skip and Create App ID** on the upper right side. Give your app a name, email address, select a category and hit **Create App ID**

<br>

#### Step 3.3: Configure Facebook app

![social-auth-5](/images/social-auth-tutorial/5.png)

Go to **Settings** on the left-side menu. There you'll see your facebook app's **App ID** and **App Secret**.

All you need to do is to paste the Selfbits app subdomain you've noted down earlier inside **App Domains**.

Next, click on **+ Add Platform**, select **Website** and paste the subdomain also into **Site URL**.

Hit **Save Changes**!

Finally note down your Facebook app's **App ID** and **App Secret**.


<br>

### Step 4: Configure Facebook Auth on Selfbits

Back to Selfbits, open up Authentication, Social and click on the settings gear of facebook to open the modal again.

Paste in the credentials of the facebook app we've created above.

App ID goes into Client Id.
App Secret goes into Client Secret.

Click **Save**

That's it! We're ready to implement the facebook sign in to our ng2-admin.

### Step 5: Implement Facebook sign in

**Note:** For this step, we'll need the Selfbits-Angular2-SDK. If you haven't installed it go to [Step 1](/)

Go to **ng2-admin/src/app/pages/login/login.component.ts** and inject the SelfbitsAngular as well as the Router Service into the constructor.

```js
constructor(fb:FormBuilder, private router:Router, private sb:SelfbitsAngular) {...}
```

Next we'll implement a quick method for our facebook.

```js
public social(provider:string){
    this.sb.auth.social(provider).subscribe(res => {
      if(res.status === 200){
        this.router.navigate(['pages']);
      }
    })
  }
```

All we need to do is to call the auth.social method from the SDK and add a click event to the login.html file (line ~34).

```html
<li (click)="social('facebook')"><i class="socicon socicon-facebook" title="Share on Facebook"></i></li>
```

Wait for the app to compile and try clicking on the facebook button on the login page!

![social-auth-5](/images/social-auth-tutorial/6.png)


Congrats! You've just added a facebook sign in to your app! Try to repeat the process with another social provider like **google+**! The process is similar to facebook, check the [official docs from Selfbits](http://docs.selfbits.io/auth/social/google/)


### Summary

In this article we've connected our dashboard to a user management system from Selfbits and enabled end users to sign up and login with their facebook accounts. There are a couple more methods the SDK offers, which you can look up [here](https://github.com/selfbits/selfbits-angular2-sdk).
