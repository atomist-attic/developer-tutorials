# Deploying your application on CloudFoundry


While local deployment is something that is definitely great, having your application deployed in the cloud is even better. Atomist makes this a breeze as well, as it supports Pivotal CloudFoundry out of the box. First of all, you need to add the CloudFoundry extension pack

``` 
npm install @atomist/sdm-pack-cloudfoundry
```

First, add the extension to your SDM:

``` typescript
sdm.addExtensionPacks(
        CloudFoundrySupport,
    );
```

Next, add a couple of goals to add deployment to your goalset so that the SDM will react on your commits by deploying on CloudFoundry:

``` typescript
sdm.addGoalContributions(goalContributors(
        whenPushSatisfies(anySatisfied(HasSpringBootApplicationClass, HasCloudFoundryManifest))
            .setGoals(new Goals("Deploy to Pivotal CF", ArtifactGoal, StagingDeploymentGoal, StagingEndpointGoal)),
    ));
```

Basically, our deployment pipeline looks like this:

```
Artifact > Deploy to staging > Communicate endpoint
```

After adding the goals you need to instruct the Atomist SDM how it should handle these goals. You do this by adding goal implementations.

``` typescript
const deployToCloudFoundry = {
    deployer: new CloudFoundryBlueGreenDeployer(configuration.sdm.projectLoader),
    targeter: () => new EnvironmentCloudFoundryTarget("staging"),
    deployGoal: StagingDeploymentGoal,
    endpointGoal: StagingEndpointGoal,
    undeployGoal: StagingUndeploymentGoal,
};
sdm.addGoalImplementation("CloudFoundry deployer",
    deployToCloudFoundry.deployGoal,
    executeDeploy(
        sdm.configuration.sdm.artifactStore,
        sdm.configuration.sdm.repoRefResolver,
        deployToCloudFoundry.endpointGoal, deployToCloudFoundry),
    {
        pushTest: IsMaven,
        logInterpreter: deployToCloudFoundry.deployer.logInterpreter,
    },
);
sdm.addKnownSideEffect(
    deployToCloudFoundry.endpointGoal,
    deployToCloudFoundry.deployGoal.definition.displayName,
    AnyPush);
sdm.addGoalImplementation("CloudFoundry undeployer",
    deployToCloudFoundry.undeployGoal,
    executeUndeploy(deployToCloudFoundry),
    {
        pushTest: IsMaven,
        logInterpreter: deployToCloudFoundry.deployer.logInterpreter,
    },
);
```

In order to turn on deployment and have the possibility to enable and disable deployment, you need to add a couple of commands.

``` typescript
sdm.addCommand(EnableDeploy)
    .addCommand(DisableDeploy)
    .addCommand(DisplayDeployEnablement)
```

If your application doesn't have a CloudFoundry manifest, the extension pack provides a command to add a manifest to your application. Once added, you can instruct Atomist to react to a push that adds that manifest to automatically enable deployment.

``` typescript
sdm.addPushImpactListener(enableDeployOnCloudFoundryManifestAddition(sdm));
```

Pivotal CloudFoundry requires you to enter some information regarding credentials and environment. You can configure these in your `~/.atomist/client.config.js`.

```
{
    "sdm": {
        "cloudfoundry": {
            "user": "foo",
            "password": "bar",
            "org": "baz",
            "spaces": {
              "staging": "staging",
              "production": "production"
            }
          }
    }
}
```

Once you have all these features implemented in your SDM, start it up using `atomist start --local`.

Go to your project folder and instruct Atomist to add a manifest by typing the following command in your terminal

```
atomist add Cloud Foundry manifest
```

Atomist will now add a manifest file to your repository in a separate branch and deploy your application on CloudFoundry and display the deployment URL in the feed. 
