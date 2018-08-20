## How does the SDM work?

If you look at the seed SDM, the entry point for Atomist resides in the `atomist.config.ts` file:

``` typescript
const machineOptions: ConfigureOptions = {
    requiredConfigurationValues: [
    ],
};

export const configuration: Configuration = {
    postProcessors: [
        configureSdm(machine, machineOptions),
    ],
};
```

The `Configuration` here is what makes Atomist tick. The `configureSdm` method will call the `machine` method (defined in `machine.ts`) which will add all the functionality to the SDM.

``` typescript
export function machine(
    configuration: SoftwareDeliveryMachineConfiguration,
): SoftwareDeliveryMachine {

    const sdm = createSoftwareDeliveryMachine({
        name: "Minimal Seed Software Delivery Machine",
        configuration,
    });
    addSpringSupport(sdm);

    summarizeGoalsInGitHubStatus(sdm);

    return sdm;
}
```

Here, the `addSpringSupport` will add all the functionality needed to build and locally deploy a Spring Boot application.

## Events

Atomist reacts to events. These events can come from the outside world, for example pushes to a Git repository or internally. In response to those events, it will trigger a set of goals grouped in goalsets, using a set of defined preconditions to determine which goalsets need to be trigger. For example, you have the goal and goalset definition in the seed SDM:

``` typescript
export const CheckGoals = new Goals(
    "Check",
    VersionGoal,
    ReviewGoal,
    AutofixGoal,
    FingerprintGoal,
    PushReactionGoal,
);

export const BuildGoals = new Goals(
    "Build",
    ...CheckGoals.goals,
    BuildGoal,
    ArtifactGoal,
);

export const BuildWithLocalDeploymentGoals = new Goals(
    "Build",
    ...CheckGoals.goals,
    BuildGoal,
    ArtifactGoal,
    new GoalWithPrecondition(LocalDeploymentGoal.definition, ArtifactGoal),
    new GoalWithPrecondition(LocalEndpointGoal.definition, LocalDeploymentGoal),
);
```

Mapping those goals to events in Atomist is done through adding goal contributions on the SDM instance:

``` typescript
sdm.addGoalContributions(goalContributors(
        doNothingOnNoMaterialChange(),
        springBootApplicationBuild(),
        defaultMavenBuild(),
    ));
```s

If we zoom in more closely in the `springBootApplicationBuild()`, we can exactly see how the mapping is done:

``` typescript
return whenPushSatisfies(IsMaven, HasSpringBootApplicationClass)
        .itMeans("Build Spring Boot")
        .setGoals(BuildWithLocalDeploymentGoals);
```

This mapping does the following
* It triggers on a Git push
* It checks 2 preconditions:
    * It checks whether the project is a Maven project (has a POM)
    * It checks whether the project has a Spring Boot application class (so essentially is a Spring Boot application)
* If those preconditions match, it triggers the goals in the defined goalset

It is important to know that Atomist will iterate through all of the goal contributions and will see who adds goals to the event. This is why the default Maven build has a `not` clause in the push tests:

``` typescript
return whenPushSatisfies(IsMaven, not(HasSpringBootApplicationClass))
        .itMeans("Build Maven")
        .setGoals(BuildGoals);
```

## Providing goal implementations

For most of the standard goals in Atomist, there are default implementations. However, some goals need specific implementations, like the `BuildGoal`, as it needs to know _how_ it should build the code. In the case of Maven, you need to link the specific `MavenBuilder` to the build related goals:

``` typescript
function enableMavenBuilder(sdm: SoftwareDeliveryMachine) {
    const mavenBuilder = new MavenBuilder(sdm);
    addBuilderForGoals(sdm, mavenBuilder, [BuildGoal, JustBuildGoal]);
}

function addBuilderForGoals(sdm: SoftwareDeliveryMachine, builder: Builder, goals: Goal[]) {
    goals.forEach(goal => {
        sdm.addGoalImplementation("Maven build", goal, 
                                  executeBuild(sdm.configuration.sdm.projectLoader, builder),
            {
                pushTest: IsMaven,
                logInterpreter: builder.logInterpreter,
            });
    });
}
```

The same mechanism can been seen being used with the `VersionGoal`, which as specific behavior when it's a Maven build:

``` typescript
function versioningWithMaven(sdm: SoftwareDeliveryMachine) {
    sdm.addGoalImplementation("mvnVersioner", VersionGoal,
        executeVersioner(sdm.configuration.sdm.projectLoader, MavenProjectVersioner), 
                         {pushTest: IsMaven});
}
```