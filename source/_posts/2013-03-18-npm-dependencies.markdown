---
layout: post
title: "NPM dependencies gotcha"
date: 2013-03-18 12:36
comments: true
categories: [nodejs, npm, dependencies]
---

For the last year I've been working with nodejs to create web applications and more often to create dev tools for everyday use.

NodeJS comes with a pretty simple but really powerful and elegant package manager called NPM. NPM is pretty straight forward to use but now we are facing some slight problems with managing dependencies.

So this is an attempt to explain how NPM dependencies work.

Let's start with an example:

# Context
You are developing a nodejs application which you wish to distribute as an NPM package. This application uses the coffee-script module which is listed in the "dependencies" section of the `package.json` file as well as another module called "secondNPM".
"secondNPM" uses coffee-script as well so it will be listed in its `package.json` file.
<!-- more -->
when you then run "npm install ." from the root of YourApp you would expect the following folder structure to be created:

    -YourApp
        -node_modules
            -coffee-script
            -secondNPM
                -node_modules
                    -coffee-script

Well, that's not always the case. There are different scenarios and rules which will apply depending on the dependencies versioning in your `package.json`.

# ">=" version selector
 - You have specified ">=1.3.0" as the version of coffee-script required by your application.
 - "secondNPM" has ">=1.3.0" as the version of coffee-script required.

in this case your application will install the latest available version of coffee-script in your local application, but then it won't install any for the "secondNPM" module. This is because the module installed for your application will satisfy the requirements of "secondNPM".

    -YourApp
        -node_modules
            -coffee-script (latest version available)
            -secondNPM
                -node_modules

# Be more specific with your versions
I think there is a good [blog post by Nodejitsu](http://blog.nodejitsu.com/package-dependencies-done-right) on the subject, but I think it's good to reiterate on it.

If you want to avoid surprises, like discovering in production that your application does not work because the latest version of a dependency breaks your code, always be specific with the version of your dependencies.

So if your application has been developed and tested against coffee-script 1.4.2, there is no harm in specify that exact version in the `package.json`. On the other hand, if you trust the developer of the npm your app depends on, you can specify your version as 1.4.x. This will ensure that only the latest patched version of the npm will be installed, avoiding minor and major versions to be used instead.

# ".x" version selector
 - You have specified "1.4.x" as the version of coffee-script required by your application.
 - "secondNPM" has "1.3.x" as the version of coffee-script required.

In this case you'll get the latest 1.4 patched version of coffee-script installed locally in your application, the latest 1.3 patched version of coffee-script installed under "secondNPM".

    -YourApp
        -node_modules
            -coffee-script (latest 1.4 patched version available)
            -secondNPM
                -node_modules
                    -coffee-script (latest 1.3 patched version available)

