<link data-require="bootstrap-css@3.1.1" data-semver="3.1.1" rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css" />
<link rel="stylesheet" href="/styles/website.css">

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
    
    ...

    return sdm;
}
```

## Goals and events

Atomist reacts to events. These events can come from the outside world, for example pushes to a Git repository or internally. In response to those events, it will trigger a set of goals grouped in goalsets, using a set of defined preconditions to determine which goalsets need to be trigger. By default, Atomist provides quite a few goals out of the box, among which:

* `BuildGoal`: Builds your application
* `ArtifactGoal`: Store the built artifact of your application
* `AutofixGoal`: Perform autofixes

Mapping those goals to events in Atomist is done through adding goal contributions on the SDM instance:

``` typescript
sdm.addGoalContributions(goalContributors(
        onAnyPush()
            .setGoals(new Goals("Checks", VersionGoal, ReviewGoal, PushReactionGoal, AutofixGoal)),
        whenPushSatisfies(anySatisfied(IsMaven))
            .setGoals(new Goals("Build", BuildGoal)),
        ),
    ));
```

This mapping does the following
* It triggers on a Git push and always performs 4 goals: `VersionGoal`, `ReviewGoal`, `PushReactionGoal`, `AutofixGoal`
* It checks whether the project is a Maven project (has a POM) and adds the `BuildGoal` if this is the case

It is important to know that Atomist will iterate through all of the goal contributions and will see who adds goals to the event. In other words, the goal contributors are additive.

## Providing goal implementations

For most of the standard goals in Atomist, there are default implementations. However, some goals need specific implementations, like the `BuildGoal`, as it needs to know _how_ it should build the code. In the case of Maven, you need to link the specific `MavenBuilder` to the build related goals:

``` typescript
const mavenBuilder = new MavenBuilder(sdm);
    sdm.addGoalImplementation("Maven build",
        BuildGoal,
        executeBuild(sdm.configuration.sdm.projectLoader, mavenBuilder),
        {
            pushTest: IsMaven,
            logInterpreter: mavenBuilder.logInterpreter,
        });

```

If a goal doesn't have an implementation, it is skipped. In our case, this will be the case for `VersionGoal`

In short, the goal pipeline for a Maven build will look like this:

```
+-------------+      +------------+      +-------------------+      +-------------+       +-----------+
|             |      |            |      |                   |      |             |       |           |
|   VERSION   --------   REVIEW  ---------   REACT TO PUSH   --------   AUTOFIX   ---------   BUILD   |
|             |      |            |      |                   |      |             |       |           |
+-------------+      +------------+      +-------------------+      +-------------+       +-----------+
```