---
title: Creating a Hello World Nodejs Heroku Project
tags:  heroku, javascript, js, nodejs
article_header:
  type: cover
  image:
    src: images/heroku-largebanner001.jpg
---

## Creating a simple Hello World Nodejs Heroku Project

PENDING PENDING PENDING TUTORIALThis is just a quick guide to follow with the tutorial of the following [video](https://youtu.be/).

With this tutorial you are going to be able to create a simple **repository** hosted in **Heroku** that runs **Nodejs** code.

### Understanding the environment 

If all of these it sounds like Greek to you then read this section:

What we are doing here is using a free-hosting online service such as **Heroku** in order to deploy there a simple **Javascript** project, in this case server-side or **Nodejs** , which will execute a simple "Hello World".

### Understanding Heroku

**Heroku** provides free hosting for web applications, of course this free-hosting is the entry tier of their services so it doesn't come without some quotas and limitations, which you would be able to check on their [website](https://www.heroku.com/home).



### Prerequistes

Assuming that you are working on a `linux` terminal, with `npm`, `git` and `vim` already installed, the prerequisites are:

	1. Heroku CLI Installation
	2. Heroku Sign Up
	3. CLI login to your Heroku Account  
	4. Downloading the buildpacks plugin 


#### Heroku CLI Installation

**Heroku** has a CLI application can be installed in many ways depending on your **OS** but as it is written in **Nodejs** and available in `npm` we can just install it by.

```
npm install -g heroku
```
 
#### Heroku Sign Up

For that you will need to make an account with them [Heroku Sign Up](https://signup.heroku.com/).

#### CLI login to your Heroku Account  

Once you have the account you can authenticate your host CLI application with it.

```
heroku login
```

####  Downloading the buildpacks plugin 

You need to download this two plugins.

```
heroku plugins:install buildpack-registry
heroku plugins:install buildpacks
```

### Creating the Project

Open a terminal and `cd` to a certain path wherever you keep **your own** repositories locally.
```
cd ~/myrepos/path
```
Create a herokuapp and clone it to your host, for local manipulation.
```
heroku apps:create <myapp-name>
heroku git:clone -a <myapp-name>
```
Assign the nodejs buildpack, and create a `package.json` file.
```
heroku buildpacks:set heroku/nodejs
```
Create a simple `index.js` by `vim index.js`
```
console.log("Hello World");
```



### Understanding the layer created by Heroku

Now we need to understand the following functioning.In a nutshell and to simplify it, **Heroku** deploys users Apps in little containers that are further compressed and deployed,these are called [Dynos](https://devcenter.heroku.com/articles/dynos) , just mind that we are create a simple "Hello World" project so it is not a Server that is going to be persistently running, for that this **Dyno**, will be of the type *One-off* as it is a One-off application, its code is going to be executed at a certain time to not to persist.
For Noobs like me it is always recommendable to draw a little diagram of the layers that we are using.

```
 -----------    --------------    -----------    -----------
| Code      |->|Package man   |->|Version man|->|Application|
| Nodejs    |  |npm           |  |git        |  |heroku     |
| ./index.js|  |./package.json|  |./.git/*   |  |./Procfile |
 -----------    --------------    -----------    -----------
```

What it means that we are writting the code in `Nodejs` in the file `./index.js` , the package manager `npm` will be preparing the dependencies and will execute the build script, we will be updating our whole repository using `git` which will be pushing the repository to `heroku` server which will do it's magic.

See the functioning of `heroku` analogous to the way `git` works, all the directives for this program and this repo will be stored in in the folder `./.git/`.
So Heroku introduces two files to files to it `./.slugignore` (which we are not going to see in this tutorial), and `./Procfile` which will determine the type of **Dyno** that we are using and the action that will perform upon Running or Deploying it.

### Setting up the project

We need to create then a `./Procfile` with the shell command at is going to be executed upon `heroku run -a <myapp-name>`
`vim Procfile`
```
worker: npm run-script start
```

We have to specify our Package manager the action of that script.
First we check the version of `Nodejs` that we have localy installed

```
node -v
```

And modify the main body of `vim package.json` (keep the rest the same) , my version is 18 but yours may be different. Don't specify further than 18.subversion , just add .x
```
  "node": "18.x"
  },
  "scripts": {
    "start": "node index.js"
  },
```
Right before you push it you can see the log of your *pseudo server* by opening this command in other terminal.
```
heroku logs -a <myapp-name> --tail
```
Now just push the changes
```
git add .
git commit -m "first"
git push heroku master
```
You should see the building log in both terminals.

Ok, so Hello World, the World has been Helloed, and you can do this anytime from anywhere by just logging into your `Heroku CLI`
```
heroku fun -a <myapp-name> worker
```

It looks like the beginning of something beautifull :D.
