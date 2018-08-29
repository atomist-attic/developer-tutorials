# Deploying your application locally

Building your application is a necessary step in order to be able to deploy your application. Atomist provides a lot of possibilities to deploy your application. You can deploy locally, to a CloudFoundry or even to Kubernetes. In this tutorial we'll show you how to easily deploy your application locally if you have a Spring Boot application. And it couldn't be simpler.

The only thing you need to add to your SDM are the following lines:

``` typescript
if (isInLocalMode()) {
        configureMavenPerBranchSpringBootDeploy(sdm);
        sdm.addCommand(ListBranchDeploys);
    }
```

The `configureMavenPerBranchSpringBootDeploy` method will add a goal and an implementation:

``` typescript
sdm.addGoalContributions(whenPushSatisfies(HasSpringBootApplicationClass)
        .setGoals(MavenPerBranchSpringBootDeploymentGoal));
sdm.addGoalImplementation("Maven deployment", MavenPerBranchSpringBootDeploymentGoal,
    executeMavenPerBranchSpringBootDeploy(sdm.configuration.sdm.projectLoader, options));
```

The per branch Spring Boot deployer is one built especially with the local SDM in mind. It will pick a port and deploy your application locally using the Spring Boot maven plugin. It will transparently add a goal to your SDM and an accompanying goal implementation that deploys your application. So now, every time you commit in your local repository, Atomist will automatically deploy your application.

The `ListBranchDeploys` command allows you to query the SDM to find out which branches are currently deployed. In order to invoke the command, type in `atomist list branch deploys` in your terminal.

