# What you'll need to get started

First of all, great to see you here! It’s awesome you want to try out Atomist and see for yourself what you can achieve using the automation tooling to provides.

In order to start to use Atomist, you need to take a couple of steps first.

## Install Git

Atomist requires using Git. You can either download the installer from the Git website or use one of the package managers (apt, yum, brew, Chocolatey) available on your system.

## Install NodeJS

The automation clients in Atomist run as Node.JS applications. This means you’ll have to install NodeJS on your system.

### Windows

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

## A Slack workspace

Install Slack ([https://slack.com](https://slack.com)) and create a new workspace where you’ll be using Atomist. You can also add Atomist to an existing Slack workspace.

## Finally, an Atomist account

You need an active Atomist account. You can link your existing GitHub account to an Atomist account here: [https://app.atomist.com](https://app.atomist.com).

You’ll need to link your Slack workspace to your Atomist workspace. Click on the settings icon in the Atomist web interface and click the Add to Slack button. You’ll be directed to the Slack website where you need to confirm linking Atomist to your Slack workspace (make sure you use the correct one if you have multiple workspaces).