## Plugins and npm modules

As you build more complex apps, you'll likely run into functionality that is not implemented in the NativeScript modules. But no worries, as NativeScript lets you leverage [npm](https://www.npmjs.com/) (node package manager) to import npm modules into your apps. Alternately, you can install NativeScript plugins, which are simply npm modules that can access native code and use Android and iOS SDKs, if required. 

In this chapter, you'll install and use and external email validator module to verify the format of email addresses as they are entered on the registration screen. Then, you'll add a NativeScript plugin, the [NativeScript social share widget](https://www.npmjs.com/package/nativescript-social-share), to let users share their grocery lists using their device's native sharing widget.

### Using an npm module in your app

It would be nice to be able to make sure people are entering well-formatted email addresses into your app on the registration screen. You could write this functionality yourself, but validating email addresses is [surprisingly tricky](http://stackoverflow.com/questions/46155/validate-email-address-in-javascript), and it's a lot easier to use one of many npm modules that already provide this validation. For Groceries let's see how to add this [email-validator module](https://www.npmjs.com/package/email-validator) to test for valid addresses.

<h4 class="exercise-start">
    <b>Exercise</b>: Install the email validator module
</h4>

Return to your terminal and make sure that you are working in the root directory in your Groceries project folder, a.k.a. here:

<div class="no-copy-button"></div>

```
sample-Groceries <----------------
    ├── app
    │   └── ...
    ├── package.json
    └── platforms
        ├── android
        └── ios
```

From the root directory install the email-validator module:

```
npm install email-validator --save
```

<div class="exercise-end"></div>

The install process does a few things in the background. First, because you added the `--save` flag, npm records this dependency in your app's `package.json`. If you open your `package.json` you should see `"email-validator"` in your app's `"dependencies"` array.

``` JavaScript
"dependencies": {
  "email-validator": "^1.0.3"
}
```

The npm CLI also creates a `node_modules` folder in the root of your app. This folder contains the code for the email-validator module, which is a bit of validation logic in `node_modules/email_validator/index.js`. 

> **TIP**: By saving your app's npm dependencies in your `package.json` file, you can always regenerate your `node_modules` folder by running `npm install`. Because of this, it's a common practice to exclude the `node_modules` folder from source control. The Groceries app uses git for source control, and as such includes `node_modules/` in its `.gitignore`.

Now that you have the module installed let's look at how to use it.

<h4 class="exercise-start">
    <b>Exercise</b>: Use the email validator module
</h4>

Open `/app/shared/view-models/user-view-model.js` and add the following line at the top of the file:

``` JavaScript
var validator = require("email-validator/index");
```

> **NOTE**: You need to explicitly point to the validator module's `index.js` file. The NativeScript framework is configured to look at the `"main"` value in an npm module's `package.json` file. In the case of this module, the `"main"` value is simply `"index"` (rather than `"index.js"`) so you need to reference the index file directly. If this package's `"main"` were set to `"index.js"`, you could use the simpler `require("email-validator")`.

To make use of this validator, add a function to `user-view-model.js`, right above the line `return viewModel`:

``` JavaScript
viewModel.isValidEmail = function() {
    var email = this.get("email");
    return validator.validate(email);
};
```

Then, edit the registration function in `app/views/register/register.js` to trap any malformed email addresses:

``` JavaScript
exports.register = function() {
    if (user.isValidEmail()) {
        completeRegistration();
    } else {
        dialogsModule.alert({
            message: "Enter a valid email address.",
            okButtonText: "OK"
        });
    }
};
```

<div class="exercise-end"></div>

In this function, the user submits an email and password, and the value is sent to the view model for validation. If it passes, registration can proceed, otherwise you show an alert:

![](img/cli-getting-started/chapter5/ios/1.png)
![](img/cli-getting-started/chapter5/android/1.png)

In general npm modules greatly expand the number of things you're able to do in your NativeScript apps. Need date and time formatting? Use [moment](https://www.npmjs.com/package/moment). Need utility functions for objects and arrays? Use [lodash](https://www.npmjs.com/package/lodash) or [underscore](https://www.npmjs.com/package/underscore). This code reuse benefit gets even more powerful when you bring NativeScript plugins into the picture.

> **WARNING**: Not all npm modules work in NativeScript apps. Specifically, modules that depend on Node.js or browser APIs will not work, as those APIs do not exist in NativeScript. The NativeScript wiki contains a [list of some of the more popular npm modules that have been verified to work in NativeScript apps](https://github.com/NativeScript/NativeScript/wiki/supported-npm-modules).

### Using a NativeScript plugin in your app

NativeScript plugins are npm modules that have the added ability to run native code and use iOS and Android frameworks. Because NativeScript plugins are just npm modules, a lot of the techniques you learned in the previous section still apply. The one big difference is in the command you use to install plugins. Let's look at how it works by installing the NativeScript social share plugin.

<h4 class="exercise-start">
    <b>Exercise</b>: Install the social sharing plugin
</h4>

Return to your terminal, make sure you're still in the root of your app, and run the following command:

```
tns plugin add nativescript-social-share
```

<div class="exercise-end"></div>

The install process does the same thing that the `npm install` command does—including retrieving the module from npm, installing the module in `node_modules`, and saving the module as a dependency in your app's `package.json`—but the `tns plugin add` command additionally configures any native code that the plugin needs to use.

For example the [NativeScript push plugin](https://github.com/NativeScript/push-plugin) uses both iOS and Android SDKs, and the `tns plugin add` command takes care of installing those. The [NativeScript flashlight plugin](https://github.com/tjvantoll/nativescript-flashlight) needs permissions to use the camera on Android, and the `tns plugin add` command takes care of setting that up too.

Now that you've installed the social share plugin, let's look at how to use it.

<h4 class="exercise-start">
    <b>Exercise</b>: Use the social sharing plugin
</h4>

Open `app/views/list/list.js` and add the following line at the top of the file, which requires the social share module you just installed:

``` JavaScript
var socialShare = require("nativescript-social-share");
```

Next you have to build some UI that lets you share a grocery list. To do so, open `app/views/list/list.xml` and add the following code below the opening `<Page>` tag, and above the opening `<GridLayout>` tag.

``` XML
<Page.actionBar>
    <ActionBar title="Groceries">
        <ActionBar.actionItems>
            <ActionItem text="Share" tap="share" ios.position="right" />
        </ActionBar.actionItems>
    </ActionBar>
</Page.actionBar>
```

This code defines an [ActionBar](https://docs.nativescript.org/ApiReference/ui/action-bar/ActionBar), which is a UI component that includes various menu items, enclosed in the `<ActionBar.actionItems>` tag. The `title` of the ActionBar lets you show page-specific titles.

> **NOTE**: On iOS devices, `<ActionItem>`s are placed from left to right in sequence; you can override that (as the code above does) by providing an `ios.position` attribute.

With the module installed and required, and the UI in place, you can implement the `<ActionItem>`'s tap handler (`share()`) in the list page's code-behind file.

To do so, return to `list.js` and paste the following code at the bottom of the file:

``` JavaScript
exports.share = function() {
    var list = [];
    var finalList = "";
    for (var i = 0, size = groceryList.length; i < size ; i++) {
        list.push(groceryList.getItem(i).name);
    }
    var listString = list.join(", ").trim();
    socialShare.shareText(listString);
};
```

<div class="exercise-end"></div>

This code takes the groceries from the grocery list view model, converts the data into a comma-separated string, and passes that string to the social share module's `shareText()` method.

Now when you run the app, you'll see a new button at the top of the screen. When you tap it, the native iOS or Android sharing widget will show to let you post your groceries to your social networks, or send them via email, message, or any other method you prefer.

![Social sharing widget on iOS](img/cli-getting-started/chapter5/ios/2.gif)
![Social sharing widget on Android](img/cli-getting-started/chapter5/android/2.gif)

Pretty cool, huh? The ability to use npm modules greatly expands the number of things you're able to do in a NativeScript app. Need to compose emails in your app? Try out the [NativeScript email plugin](https://www.npmjs.com/package/nativescript-email). Need to use the clipboard in your app? Try out the [NativeScript clipboard plugin](https://www.npmjs.com/package/nativescript-clipboard).

If you're looking for NativeScript plugins start by [searching for NativeScript on npm](https://www.npmjs.com/search?q=nativescript). If you don't find the plugin you need, you can [request the plugin on our ideas portal](https://nativescript.ideas.aha.io/), or you can take a stab at [creating the plugin yourself](https://docs.nativescript.org/plugins).

Between NativeScript modules, npm modules, and NativeScript plugins, the NativeScript framework provides a lot of functionality you can use to build your next app. However, we've yet to talk about NativeScript's most powerful feature: the ability to directly access iOS and Android APIs in JavaScript. Let's look at how it works.