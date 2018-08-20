# Adding code inspection commands

Atomist can react to events, but also to commands you can give it. For example, a command you can give Atomist out of the box is:

```
atomist list skills
```

Commands can do many things. They can trigger internal SDM behavior or call external services. If it's an action you can automate, you can turn it into a command.

# Creating your own command

Let's say we want to create a command to quickly query what the current version is in a POM xml. Atomist has a couple of command types, one of which is a code inspection command.

We first make an inspection

``` typescript
const PomVersionInspection: CodeInspection<any> = async (p: Project, ctx: SdmContext) => {
    const pom = p.findFileSync("pom.xml");
    const parser = new xml2js.Parser();
    type parseStringType = (str: string, cb: (err: Error, result: any) => void) => void;
    return promisify(parser.parseString as parseStringType)(pom.getContentSync())
        .then((parsed: any) => {
            if (!!parsed.project) {
                const projectVersion = parsed.project.version[0];
                const message = `The POM version is ${projectVersion}`;
                return sendMessage(ctx, message);
            } else {
                return sendMessage(ctx, "The project has no valid POM");
            }
        });
};

export const PomVersionCommandHandler: CodeInspectionRegistration<any> = {
    name: "Get POM version",
    description: "Get POM version",
    intent: ["get pom version"],
    inspection: PomVersionInspection,
};

function sendMessage(ctx: SdmContext, message: string) {
    return ctx.context.messageClient.respond(message);
}
```

And register the command handler in our SDM:

``` typescript
sdm.addCodeInspectionCommand(PomVersionInspection);
```

If we now run our SDM and give Atomist the command in a project directory:

```
atomist get pom version
```

It will return this:

```
➜  myproject git:(master) ✗ atomist get pom version
Get POM version
# myproject 2018-08-20T15:15: 10.875Z The POM version is 0.1.0-SNAPSHOT
```

Code inspection commands are very useful for extracting information out of the code or to get reports based on the current codebase. You can then relay that information back to the user in a form you seem fit.