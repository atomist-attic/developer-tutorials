# Transforming code through commands

In the previous tutorial, we've show how we can inspect code and send messages to Slack based on that inspection. However, Atomist can go a step further and even change your code through a command.

# Creating a code transformation command

Code transformation registration are very similar to code inspections. There's a nice example of a code transformation in the spring extension pack.

``` typescript
@Parameters()
export class UpgradeSpringBootParameters implements EditModeSuggestion {

    @Parameter({
        displayName: "Desired Spring Boot version",
        description: "The desired Spring Boot version across these repos",
        pattern: /^.+$/,
        validInput: "Semantic version",
        required: false,
    })
    // TODO this should be in a object goals
    public desiredBootVersion: string = "2.0.1.RELEASE";

    private readonly guid = "" + new Date().getTime();

    get desiredBranchName() {
        return `boot-upgrade-${this.desiredBootVersion}-${this.guid}`;
    }

    get desiredPullRequestTitle() {
        return `Upgrade Spring Boot version to ${this.desiredBootVersion}`;
    }

    get desiredCommitMessage() {
        return this.desiredPullRequestTitle;
    }
}

/**
 * Wrap Spring Boot set version editor in a dryRunEditor, causing an event
 * handler to respond to the build with either a PR and Issue
 * @type {HandleCommand<EditOneOrAllParameters>}
 */
export const TryToUpgradeSpringBootVersion: CodeTransformRegistration<UpgradeSpringBootParameters> = makeBuildAware({
    transform: SetSpringBootVersionTransform,
    paramsMaker: UpgradeSpringBootParameters,
    name: "boot-upgrade",
    description: `Upgrade Spring Boot version`,
    intent: "try to upgrade Spring Boot",
});
```

The transformation itself looks like this:

``` typescript
export const SetSpringBootVersionTransform: CodeTransform<{ desiredBootVersion: string }> =
    async (p, ctx, params) =>
        doWithMatches(p, "**/pom.xml",
            parentStanzaOfGrammar(SpringBootStarter), m => {
                if (m.version.value !== params.desiredBootVersion) {
                    logger.info("Updating Spring Boot version from [%s] to [%s]",
                        m.version.value, params.desiredBootVersion);
                    m.version.value = params.desiredBootVersion;
                }
            });

```

And register the command handler in our SDM:

``` typescript
sdm.addCodeTransformCommand(TryToUpgradeSpringBootVersion);
```

If we now run our SDM and give Atomist the command:

```
atomist try to upgrade Spring Boot
```

It will create a branch, apply the transformation and if the build succeeds, it will create a pull request for that branch. This mechanism works through the `makeBuildAware` in the `CodeTransformRegistration` definition. This can use a parameter object that extends `EditModeSuggestion` in order to get the correct information to make the commit, branch and pull request.

## Steps to create a code transform:

* Create a `CodeTransform` object and possible parameters
* Create a `CodeTransformRegistration` that links the transformation to an intent and parameters
    * Use the `makeBuildAware` method and the `EditModeSuggestion` to get the smart pull request mechanism
* Register the `CodeTransformRegistration` in the SDM


