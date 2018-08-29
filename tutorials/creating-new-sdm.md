# Creating a new automation client

Automation clients are the bread and butter of Atomist. They provide the functionality Atomist can use to perform automations. A software delivery machine, or SDM for short, is an automation client that, as the name suggest, adds functionality to deliver software. 

SDMs can do a lot of things, from building software using various build mechanisms to fixing code automatically and even deploying your software across multiple environments.

## Creating a new SDM

Atomist provides the functionality right out of the box to create a brand new SDM, based on an existing one. This mechanism allows you to quickly build a new SDM but with the same base functionality. This process is called seeding.

Open up a console and type in

```
atomist --help
```

The Atomist CLI will reply with all the intents (or chat commands) it can currently handle. In this case, as you do not have your own automation client running against your Atomist account, it will show the default commands it can handle.

One of these commands is `create sdm`. This will create a brand new SDM based on a seed. So type in:

```
@atomist create sdm
```

Atomist will now reply to this message and ask you a couple of questions. First it will ask you for the type of SDM you want to create. You can use the `blank` SDM which will not contain any commands, the `spring` SDM which supports delivering Spring Boot applications out of the box and the `sample` SDM which has a lot of cool features inside of it, think of it like a feature demo. It will then ask you for a name for your SDM and which Git user you'd like to use in order to map Atomist commits. Pick a name. After this, Atomist will create the new SDM.

As you can see, it will use the chosen SDM as a source - or seed - repository. We suggest using the `spring` SDM to start with if you just want to build a Spring Boot application. 
z
## Running the new SDM

By default, Atomist will have created a folder called `atomist` inside your home folder and in that it will have created a folder with the name of the mapped user you entered during creation. Inside that folder, you can find the SDM you just created.

To run the SDM, the only thinh you need to so is

``` bash
atomist start --local
```

## Checking the SDM functionality in Slack

Once the SDM has started up, you can ask Atomist to list its skills in a new command prompt.

```
atomist list skills
```

In the list, you should now see your new SDM command in the reply.