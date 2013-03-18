---
layout: post
title: "NPM dependencies"
date: 2013-03-18 12:36
comments: true
categories: [nodejs, npm, dependencies]
---

For the last year I've been working with nodejs to create web applications and more often to create dev tools for everyday use.

NodeJS comes with a pretty simple but really powerful and elegant package manager called NPM. NPM is pretty straight forward to use but now we are facing some slight problems with managing dependencies.

So this is an attempt to explain how NPM dependencies work.

Let's start with an example:

# Context
You are developing a nodejs application which you wish to distribute as an NPM package. This application uses the coffee-script module which is listed in the "dependencies" section of the package.json file as well as another module called "secondNPM".
"secondNPM" uses coffee-script as well so it will be listed in its package.json file.
<!-- more -->
when you then run "npm install ." from the root of YourApp you would expect the following folder structure to be created:

    -YourApp
        -node_modules
            -coffee-script
            -secondNPM
                -node_modules
                    -coffee-script

Well, that's not always the case. There are different scenarios and rules which will apply depending on your global npms and dependencies versioning in your package.json.

# Global modules
With NPM you can decide to install a module globally. That means that the module will be available from all your apps without the need to install it locally in your applications.

Back to the example:
# Scenario 1
 - You have coffee-script v1.6.0 globally installed.
 - You have specified ">=1.3.0" as the version of coffee-script required by your application.
 - "secondNPM" has ">=1.3.0" as the version of coffee-script required.

in this case when you run "npm install ." from the root folder of your application, npm will NOT install any local version of coffee-script because the global version satisfies the requirements of both child npms.

    -YourApp
        -node_modules
            -secondNPM
                -node_modules

# Scenario 2 <- VERIFY THIS ONE!
 - You DON'T have a global installation of coffee-script
 - You have specified ">=1.3.0" as the version of coffee-script required by your application.
 - "secondNPM" has ">=1.3.0" as the version of coffee-script required.

in this case your application will install the latest available version of coffee-script in your local application, but then it won't install any for the "secondNPM" module. This is because the module installed for your application will satisfy the requirements of "secondNPM".

    -YourApp
        -node_modules
            -coffee-script (latest version available)
            -secondNPM
                -node_modules

# Be more specific with your versions
I think there is a good blog post by Nodejitsu on the subject, but I think it's good to reiterate on it.

If you want to avoid surprises, like discovering in production that your application does not work because the latest version of a dependency breaks your code, always be precise with the version of your dependencies.

So if your application has been developed and tested against coffee-script 1.4.2, there is no harm in specify that exact version in the package.json. On the other hand, if you trust the developer of the npm your app depends on, you can specify your version as 1.4.x. This will ensure that only the latest patched version of the npm will be installed, avoiding minor and major versions to be used instead.

# Scenario 3
 - You DON'T have a global installation of coffee-script
 - You have specified "1.4.x" as the version of coffee-script required by your application.
 - "secondNPM" has "1.3.x" as the version of coffee-script required.

In this case you'll get the latest 1.4 patched version of coffee-script installed locally in your application, the latest 1.3 patched version of coffee-script installed under "secondNPM".

    -YourApp
        -node_modules
            -coffee-script (latest 1.4 patched version available)
            -secondNPM
                -node_modules
                    -coffee-script (latest 1.3 patched version available)

# Scenario 4
 - You have coffee-script v1.4.5 globally installed.
 - You have specified "1.4.x" as the version of coffee-script required by your application.
 - "secondNPM" has "1.3.x" as the version of coffee-script required.

In this case you won't get any version of coffee-script installed locally for your application as the global version satisfies the requirements, but then you will get the latest 1.3 patched version of coffee-script installed under "secondNPM".

    -YourApp
        -node_modules
            -secondNPM
                -node_modules
                    -coffee-script (latest 1.3 patched version available)


These are not all the scenarios you can stumble upon, but just some that I encountered and made me think about which strategy I wanted to adopt for my specific case.

Any suggestion/amendment/error please let me know.


