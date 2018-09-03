# What you'll need to get started

First of all, great to see you here! It’s awesome you want to try out Atomist and see for yourself what you can achieve using the automation tooling it provides.

In order to start to use Atomist, you need to take a couple of steps first.

## Install Git

Atomist requires using Git. You can either download the installer from the Git website or use one of the package managers (apt, yum, brew, Chocolatey) available on your system.

## Install NodeJS

The Software Delivery Machines in Atomist run as Node.JS applications. This means you’ll have to [install NPM and NodeJS](https://www.npmjs.com/get-npm) on your system.

### Windows

(Warning: Windows is not officially suppported yet)

Use the Windows installer to add NodeJS to your system. You can find the download here: [https://nodejs.org/en/download/](https://nodejs.org/en/download/).

If you have Chocolatey installed, run the following command from your terminal:

```
choco install 
nodejs.install
```

### Linux

Use your favorite package manager (apt, yum, pacman, ...) to install NodeJS on your system.

### Mac OSX

You can either use the installer found at the same URL as mentioned in the Windows install or if you have brew installed, run the following command:

```
brew install nodejs
```

## Install the Atomist CLI

The Atomist CLI is what enables your code to talk to Atomist running locally. Using Node.JS, you can install the CLI 
like this:

```
npm install -g @atomist/cli
```

You can verify the installation by typing 

```
atomist --help
```

You're ready now to start using Atomist locally.
