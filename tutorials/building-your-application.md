# Building your application

The purpose of a CI/CD tools is offcourse to build your software. If you look at the standard goalset in the `spring-seed-sdm`, you can see it adds the build step when there is a Maven POM detected in the project:

``` typescript
sdm.addGoalContributions(goalContributors(
        onAnyPush().setGoals(new Goals("Checks", ReviewGoal, PushReactionGoal, AutofixGoal)),
        whenPushSatisfies(IsDeploymentFrozen)
            .setGoals(ExplainDeploymentFreezeGoal),
        whenPushSatisfies(anySatisfied(IsMaven))
            .setGoals(BuildGoal),
    ));
```

Off course, having just the `BuildGoal` by itself is not sufficient. You need to have a goal implementation that actually wires the Maven build process to the goal execution. Doing this is very straightforward:

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

The `MavenBuilder` is a builder implementation that does an `mvn package`. By default, it skips the tests but you can 
configure the builder that it doesn't. The goal implementation executes the builder process, taking into account that it should only trigger on a Maven build. In our case, it's redundant, but if you have multiple builders in your SDM, this is necessary.

Out of the box, Atomist currently supports building your software using Maven or NPM.