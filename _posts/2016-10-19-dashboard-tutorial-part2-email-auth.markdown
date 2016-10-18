---
layout: post
title:  "Angular 2 Dashboard Tutorial - Part 1: Email Authentication"
date:   2016-10-18 13:58:05 +0200
categories: jekyll update
---

# Create your Angular 2 Dashboard - Part 1: Email Authentication

This part will cover a simple email sign up and sign in.
If you haven't done the setup yet, please go to Part 0 to do so.
Feel free to skip this part to go directly to social (facebook) authentication using Selfbits.

## Complete Content

0. Part 0: Installation and Setup
1. Part 1: Simple email sign up and sign in
2. Part 2: Social (Facebook) sign up and sign in
3. Part 2.5: Creating a separate AuthModule and AuthGuard
4. Part 3: ...


## Part 1: Email Authentication

![Imgur](http://i.imgur.com/SYsfC35.png)


In the first part, we'll implement the following features using the vanilla ng2-admin sign in and sign up page.

* Email sign up
* Email sign in with sdk


### Step 1: Change default route to login page

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

![Imgur](http://i.imgur.com/SYsfC35.png)

**Note:** We'll create later we'll our own AuthModule with more advanced routing features in Part 2.5.

<br>

### Step 2: Connect ng2-admin to Selfbits

In order to connect to the selfbits app we've created earlier, we need to provide ng2-admin the
credentials of our selfbits app. We can provide them globally using the **providers** property inside our app.module
and use the @Inject decorator to retrieve them wherever we need.

**Note:** Step 2 illustrates a couple of cool angular 2 concepts. However for faster implementation you can [skip]() this step and go to Step 3 where we'll achieve the same thing with a SDK from Selfbits in much shorter time.

<br>

#### Step 2.1 Create Auth Service

First let's create an AuthService. Following the [StyleGuide](https://angular.io/docs/ts/latest/guide/style-guide.html), we create a **shared** folder inside src and then a **services** folder to house our auth.service.ts

```
ng2-admin
|...
|--src
|   |--app
|   |--shared
|   |   |--services
|   |   |   |--auth.service.ts
```
<br>

#### Step 2.2: Configure our app.module

Go to **ng2-admin/src/app/app.module.ts** and copy this code **below** the last import. If the terminal shows error don't worry! We'll implement AuthService shortly.

```js
import {AuthService} from "./shared/services/auth.service";

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

First we import the AuthService (since we already know it's location) and declare a typing for our config variable.

Next we enter the credentials into the config variable and add type support to it. The credentials can be found in your Selfbits app settings [here]() TOFILLOUT

<br>

```js
@NgModule({
  (...)
  providers:[
    ENV_PROVIDERS,
    APP_PROVIDERS,
    AuthService,
    {provide:'APP_CONFIG_TOKEN', useValue:APPCONFIG}
  ]
  (...)
})
```

Finally we need to provide both the AuthService and the config variable globally so that we can inject them later to any service or component we want.

<br>

#### Step 2.3: Create AuthService

Go to **ng2-admim/src/app/shared/services/auth.service.ts** and paste this code below.

```js
import {Injectable, Inject} from "@angular/core";
import {AppConfig} from "../../app.module";
import {Http, Headers, Response} from "@angular/http";
import {Observable} from "rxjs";


interface auth{
  email:string,
  password:string
}

@Injectable()

export class AuthService {

  constructor(@Inject ('APP_CONFIG_TOKEN') private config:AppConfig, private http:Http){
  }

  signup(formData:auth):Observable<Response>{
    return this.http.post(
      `${this.config.BASE_URL}/api/v1/auth/signup`,
      JSON.stringify(formData),
      {headers:new Headers({'sb-app-id':this.config.APP_ID,'sb-app-secret':this.config.APP_SECRET})})
  }
}
```

This AuthService contains only the **signup method** as later on we will use a cool sdk the guys from Selfbits provided.


```js
constructor(@Inject ('APP_CONFIG_TOKEN') private config:AppConfig, private http:Http){
  console.log(this.config)
}
```
Inside the constructor you can that we can now retrieve the provided config variable via it's string token and assign it to a local variable called config. Go ahead and console.log it!

```js
signup(formData:auth):Observable<Response>{
  return this.http.post(
    `${this.config.BASE_URL}/api/v1/auth/signup`,
    JSON.stringify(formData),
    {headers:new Headers({'sb-app-id':this.config.APP_ID,'sb-app-secret':this.config.APP_SECRET})})
}
```
As you can see the signup method is basically a http post wrapper containing the information provided by the APPCONFIG variable in app.module.

**Note:** the signup method returns an Observable of type Response, which is the default return type of angular's http requests. Also in order to be able to use Http, you need to have import the HttpModule, which ng2-admin did alredy in the app.module.

The [http post](https://angular.io/docs/ts/latest/api/http/index/Http-class.html) request takes 3 paramenters

```js
post(url: string, body: any, options?: RequestOptionsArgs) : Observable<Response>
```

<br>

#### url

The Url we get by concatenating **your app's subdomain + /api/v1 + rest end point**

Go to your [selfbits admin dashboard](https://admin.selfbits.io), enter your project and click on **Authorization** and then **Users**.

Scroll down and click on **Open/Hide** to see the Rest Endpoint Definitions

![Imgur](http://i.imgur.com/T9tZ5FN.png)

and click on **/auth/signup** in our case the url for sign up should be

```js
`${this.config.BASE_URL}/api/v1/auth/signup`
```

We are using string templates and retrieve the base url via @Inject from our APPCONFIG variable.

<br>

#### body

The body will be just the data we get past to the signup method.

**Note:** calling the JSON.stringify method isn't actually necessary, since angular's http will do it for us.

<br>

#### options?

We need to transmit our app credentials to selfbits and this is being done through headers.

The parameters stated here need to be added to the headers.

<br>

![Imgur](http://i.imgur.com/etC9H9l.png)

<br>

For auth/signup we need to add **sb-app-id** and **sb-app-secret** to the headers and assign their values accordingly.

```js
{headers:new Headers({'sb-app-id':this.config.APP_ID,'sb-app-secret':this.config.APP_SECRET})}
```

<br>


#### Step 2.4 Using AuthService

Go to **ng2-admin/src/app/pages/register/register.component.ts**

It uses angular's **data-driven** approach to create a form for the sign up process.

```js
export class Register{
  (...)
  constructor(fb:FormBuilder) {
    this.form = fb.group({
      'name': ['', Validators.compose([Validators.required, Validators.minLength(4)])],
      'email': ['', Validators.compose([Validators.required, EmailValidator.validate])],
      'passwords': fb.group({
        'password': ['', Validators.compose([Validators.required, Validators.minLength(4)])],
        'repeatPassword': ['', Validators.compose([Validators.required, Validators.minLength(4)])]
      }, {validator: EqualPasswordsValidator.validate('password', 'repeatPassword')})
    });
  }
}
```
<div class="alert alert-warning" role="alert">
**Note:** There's is another approach to create forms in angular 2 called **template-driven** approach, which is faster to implement but offers less control. For authorization I always recommend using the data-driven approach.
</div>

Inside register component's constructor you can see that the form has 2 controls (**'name'** and **'email'**) and 1 control group (**'passwords'**), which consists of 2 controls (**'password'** and **'repeatPassword'**). Each of them have some basic angular 2 validators, which we'll leave for now.

Import and inject our AuthService into the constructor and call the signup method from our service.

```js

import {AuthService} from "../../shared/services/auth.service";

(...)

constructor(fb:FormBuilder, private auth:AuthService) {}

(...)

public onSubmit(values:any):void {
    this.submitted = true;
    if (this.form.valid) {
      // your code goes here
      // console.log(values);
      this.auth.signup({email:values.email, password:values.passwords.password}).subscribe(res =>{
        alert(res);
      })
    }
  }
```
As signup returns an Observable, we need to subscribe to it. If the http request was successful, it should pop an alert with a response of status 200. Alternatively console.log the response instead of using alert.

**Note:** Change the type of values to **any** to avoid TypeScript Errors.

Let's test it! Wait for the app to compile, go to
```
http://localhost:3000#/register
```
punch in some credentials and click **Sign up**!

**Note:** The form validators we've left untouched are now kicking in. For example, each field needs to be longer than 4 characters or it'll be invalid.

![Imgur](http://i.imgur.com/sXZZchm.png)

In fact, let's head back to our selfbits app and check if the user is registered!

Go to your the [selfbits admin dashboard](https://admin.selfbits.io), enter your app and click Authentication > Users

![Imgur](http://i.imgur.com/seXRrCZ.png)

That's it! We now already have a working email sign up. Let's see if we can sign in with the user we've just created!

<br>

### Step 3: Email Login

Go back to **ng2-admim/src/app/shared/services/auth.service.ts**

Add the **login** method below the **signup** method

```js
login(formData:auth):Observable<Response>{
    return this.http.post(
      `${this.config.BASE_URL}/api/v1/auth/login`,
      JSON.stringify(formData),
      {headers:new Headers({'sb-app-id':this.config.APP_ID,'sb-app-secret':this.config.APP_SECRET})})
  }
```
As you can see all we actually need to change was the url to end with **login**!

Go to **ng2-admim/src/app/pages/login/login.component.ts**

This time we wan't to redirect the user if the sign in was successful. We'll be using angular's Router service.
First, inject AuthService and Router into the constructor.

```js
constructor(fb:FormBuilder, private auth:AuthService, private router:Router) {...}
```

Then we implement the onSubmit method

```js
public onSubmit(values:any):void {
    this.submitted = true;
    if (this.form.valid) {
      // your code goes here
      // console.log(values);
      this.auth.login({email:values.email, password:values.password}).subscribe(res => {
        console.log(res);
        if(res.status === 200){
          this.router.navigate(['pages']);
        }
      })
    }
  }
```

Let the app compile and go to
```
http://localhost:3000
```

Enter the credentials of the user we've just created. If everything goes right, we should get redirected to
```
http://localhost:3000/#/pages/dashboard
```

<br>

### Step 4: Error Handling

So what happens if the user types in the wrong email or password? We need to prompt the user in case there has been an error.

First, let's see what happens if the user does login with the wrong password.

Go to
```
http://localhost:3000
```

and type in the user email with a wrong password

![Imgur](http://i.imgur.com/1M4AItI.png)

The console will log out the the error, in particular the response body will contain the error message. In this case it's as expected a 401 Unauthorized error.

We could now add more **if** statement for various response status. The proper way however is to fully utilize our response observable by adding a 2nd callback to the subscribe method. Below is a simple example on how to add error message for the user.

Adjust the code in **ng2-admin/src/app/pages/login/login.component.ts**

```js

public error:boolean=false;
public errorMessage:string='';

(...)

public onSubmit(values):void {
    this.submitted = true;
    if (this.form.valid) {
      // your code goes here
      // console.log(values);
      this.auth.login({email:values.email, password:values.password}).subscribe(res =>{
        console.log(res);
        if(res.status === 200){
          this.router.navigate(['pages'])
        }
      }, err => {
        //do something with error
        this.error = true;
        this.errorMessage = err.json().message;
      })
    }
  }
```

And add below the two form control div (should be line 21) to **ng2-admin/src/app/pages/login/login.html**

```html
<!--Below the two form control div -->
<div class="alert alert-danger" role="alert" *ngIf="error">{{errorMessage}}</div>
```

![Imgur](http://i.imgur.com/QoZd91O.png)


### Step 4: selfbits-angular2-sdk

As stated earlier, we can reduce implementation time a lot by using a sweet sdk the guys form selfbits provided.

#### Step 4.1: Install SDK

Go to the project root folder and type into your terminal

```
npm install selfbits-angular2-sdk --save
```

#### Step 4.1: Import SDK
Go to **ng2-admin/src/app/app.module.ts**

```js
import {SelfbitsAngularModule} from "selfbits-angular2-sdk";
```

#### Step 4.2: Configure app
**Note** this needs to be placed **ABOVE** the @NgModule decorator

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

#### Step 4.3: Initialize the SDK

```js
@NgModule({
  imports: [
    (...)
    SelfbitsAngularModule.initializeApp(APPCONFIG)
  ]
})
```

#### Step 4.4 Use SelfbitsAngular Service

The SelfbitsAngular Service contains a auth property which has among many also the signup and login method implemented.

Go to **ng2-admin/src/app/pages/login/login.component.ts**

Import and inject the SelfbitsAngular Service

```js
import {SelfbitsAngular} from 'selfbits-angular2-sdk'

 (...)

 constructor(fb:FormBuilder, private auth:AuthService, private router:Router, private sb:SelfbitsAngular) {}

```

Now all you need is to call it!

```js
public onSubmit(values):void {
    this.submitted = true;
    if (this.form.valid) {
      // your code goes here
      // console.log(values);
      this.sb.auth.login({email:values.email, password:values.password}).subscribe(res =>{
        console.log(res);
        if(res.status === 200){
          this.router.navigate(['pages'])
        }
      }, err => {
        //do something with error
        this.error = true;
        this.errorMessage = err.json().message;
      })
    }
  }
```

Try to implemnt the auth.signup yourself! All in all if you go the the [sdk's repository](https://github.com/selfbits/selfbits-angular2-sdk/blob/master/src/services/auth.ts), you can see that the login method looks pretty much the same
