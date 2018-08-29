# Versioning your application

<ul class="steps">
    <li class="active"><a href="">Version</a></li>
    <li class="undone"><a href="">Review</a></li>
    <li class="undone"><a href="">React to push</a></li>
    <li class="undone"><a href="">Autofix</a></li>
    <li class="undone"><a href="">Build</a></li>
</ul>

Most applications are versioned using Semantic Versioning principles (we hope), but sometimes we need more informtion in order to uniquely identify versions for specific builds. 

For example, Maven uses SNAPSHOT versioning and when deploying a version, it uses timestamps to uniquely identify versions. In this case, we don't really need to do anything.

However, other frameworks like NodeJS don't have this functionality and you probably only have semantic versions in your `package.json`. This is where the `VersionGoal` comes into play. This goal will change your `package.json` for the duration of the goalset execution.

In order to enable this, you need to add an implementation for the `VersionGoal`:

``` typescript
sdm.addGoalImplementation(
    "nodeVersioner",
    VersionGoal,
    executeVersioner(sdm.configuration.sdm.projectLoader, NodeProjectVersioner),
    { pushTest: IsNode },
)
```

