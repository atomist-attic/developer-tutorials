# Autofixing code

So we now know that can have Atomist complain about bad code and we can write commands to change code. But what if we know that we can automatically fix bad code? Like for example formatting or file headers? Well, that's where Atomist's autofixes come into play.

Autofixes change the code automatically for you, adding a new specific autofix commit to the branch you committed the code on.

Say that we want to make sure that every project in our codebase has a license file. We can write an autofix for that in case people forget a `LICENSE` file.

``` typescript
export const LicenseFilename = "LICENSE";

export const AddLicenseFile: AutofixRegistration = {
    name: "License Fix",
    pushTest: not(hasFile(LicenseFilename)),
    transform: async p => {
        const license = await axios.get("https://www.apache.org/licenses/LICENSE-2.0.txt");
        return p.addFile("LICENSE", license.data);
    },
};
```

This autofix will check whether our code has a `LICENSE` file and if not, it will add the Apache 2.0 license file.

But we can also call external systems to do the autofixing for us. For example, TSLint is able to fix code based on its configuration using an `npm` command. With Atomist, we can automate this:

``` typescript
export const tslintFix: AutofixRegistration = spawnedCommandAutofix(
    "tslint",
    allSatisfied(IsTypeScript, IsNode, hasFile("tslint.json")),
    {ignoreFailure: true, considerOnlyChangedFiles: false},
    Install,
    asSpawnCommand("npm run lint:fix", DevelopmentEnvOptions));
```

This will call `npm run lint:fix` and commit the changes in a new autofix commit.