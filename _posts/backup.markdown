

The guys from Selfbits created a simple SDK to make this whole process easier. I'm going to use their sdk, but I will also provide an example later on how to connect [without the SDK]().

First head to your project root folder and type in your terminal

```
npm install selfbits-angular2-sdk -S
```

Next we need to load the selbits angular module and initialize it with a SELFBITSCONFIG constant.

Go to

```
ng2-admin/src/app/app.module.ts
```

##### Step 2.1: Imports
```js
import {SelfbitsAngularModule} from "selfbits-angular2-sdk";
```

##### Step 2.2: Configure app
**Note** this needs to be placed ABOVE the @NgModule decorator

```js
(...)
export const SELFBITSCONFIG = {
    BASE_URL: 'your app subdomain',
    APP_ID: 'your app id',
    APP_SECRET: 'your app secret',
};
(...)
@NgModule({...})
```

##### Step 2.3: Import and initialize the module

```js
@NgModule({
  imports: [
    (...)
    SelfbitsAngularModule.initializeApp(SELFBITSCONFIG)
  ]
})

```

One more step to see some results!

### Step 3 Email Sign up

Go to

```
ng2-admin/src/app/pages/register/register.component.ts
```

It uses angular's **data-driven** approach to create a form for the sign up process.

**Note** There's is another approach to create forms in angular 2 called **template-driven** approach, which is faster to implement but offers less control. For authorization I always recommend using the data-driven approach.


Inside register component's constructor you can see that the form has 2 controls and 1 control group ('passwords'), which consists of 2 controls ('password' and 'repeatPassword'). Each of them have some basic builtin validators, which we'll leave for now. Inside the constructor we only need to inject the sdk's service called SelfbitsAngular.


```js

import {SelfbitsAngular} from 'selfbits-angular2-sdk'

(...)

export class Register{
(...)
  constructor(fb:FormBuilder, private sb:SelfbitsAngular) {

    this.form = fb.group({
      'name': ['', Validators.compose([Validators.required, Validators.minLength(4)])],
      'email': ['', Validators.compose([Validators.required, EmailValidator.validate])],
      'passwords': fb.group({
        'password': ['', Validators.compose([Validators.required, Validators.minLength(4)])],
        'repeatPassword': ['', Validators.compose([Validators.required, Validators.minLength(4)])]
      }, {validator: EqualPasswordsValidator.validate('password', 'repeatPassword')})
    });
    (...)
  }
  (...)
}
```

Now we'll use the SelfbitsAngular service to access it's **auth.signup** method.
The signup method return an Observable of type Response, so we need to subscribe to it.

```js
public onSubmit(values:any):void {
  this.submitted = true;
  if (this.form.valid) {
    // your code goes here
    this.sb.auth.signup({email:values.email, password:values.passwords.password}).subscribe(res => alert(res))
  }
}
```

<div class="alert alert-info" role="alert">
**Note**: Change the value type of onSubmit to any to avoid TypeScript errors
</div>

Let the app compile and reload, click on **"new to ng2-admin? Sign up!"** or go directly to
```
http://localhost:3000/#/register
```

Fill in the form and click on Signup!

If everything went well, there should be an alert popup with a status:200 looking like this

![Imgur](http://i.imgur.com/sXZZchm.png)

**Note** if you have popup blocker installed, you can also use console.log(res) instead of alert(res)

In fact, let's head back to our selfbits app and check if the user is registered!

Go to your the [selfbits admin dashboard](https://admin.selfbits.io), enter your app and click Authentication > Users

![Imgur](http://i.imgur.com/seXRrCZ.png)

That's it! We now already have a working email sign up. Let's quick do the login can check if we can do that with the user we've just created!

### Step 4 Email Login

Go to

```
ng2-admin/src/app/pages/login/login.component.ts
```

We repeat the same process by importing and injecting the SelfbitsAngular service as above but call the auth.login method instead!

```js

import {SelfbitsAngular} from 'selfbits-angular2-sdk'

(...)


constructor(fb:FormBuilder, private sb:SelfbitsAngular){...}

(...)

public onSubmit(values:any):void {
  this.submitted = true;
  if (this.form.valid) {
    // your code goes here
    this.sb.auth.login({email: value.email, password: values.password}).subscribe(res => alert(res))
  }
}

```

**Note** Remember to change the type of values. Also here we only need values.password since this is how the login form control is defined.

Fill in the form with the credentials we've just used to create a new user and hit login!

![Imgur](http://i.imgur.com/zcvvpRz.png)

### Step 5 Redirect upon successful sign up / sign in

Now we obviously want to redirect the user upon authentication so let's use angular's router to do that!

Inside
```
ng2-admin/src/app/pages/login/login.component.ts
```

Import and inject the Router service from @angular/router
```js
import {Router} from "@angular/router";

(...)

constructor(fb:FormBuilder, private sb:SelfbitsAngular, private router:Router) {...}

(...)

public onSubmit(values:any):void {
  this.submitted = true;
  if (this.form.valid) {
    // your code goes here
    this.sb.auth.login({email: value.email, password: values.password}).subscribe(res => {
      console.log(res);
        if(res.status === 200){
          this.router.navigate(['pages'])
        }
    })
  }
}
```
Repeat the exact same thing for

```
ng2-admin/src/app/pages/register/register.component.ts
```

**Note** the route 'pages' will redirect to dashboard, as it is coded inside pages.routing.ts

### Step 6 Error Handling

So what happens if the user types in the wrong email or password? We need to prompt the user in case there has been an error.

First, let's see what happens if the user does login with the wrong password.

Go to
```
ng2-admin/src/app/pages/login/login.component.ts
```

and type in the user email with a wrong password

![Imgur](http://i.imgur.com/1M4AItI.png)

The console will log out the the error, in particular the response body will contain the error message. In this case it's as expected a 401 Unauthorized error.

We could now add more **if** statement for various response status. The proper way however is to fully utilize our response observable by adding a 2nd callback to the subscribe method. Below is a simple example on how to add error message for the user.

```
ng2-admin/src/app/pages/login/login.component.ts
```
```js

public error:boolean=false;
public errorMessage:string='';

(...)

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
```
ng2-admin/src/app/pages/login/login.html
```
```html
<!--Below the two form control div -->
<div class="alert alert-danger" role="alert" *ngIf="error">{{errorMessage}}</div>
```

![Imgur](http://i.imgur.com/QoZd91O.png)
