# Creating a new project through the SDM

The Spring SDM is capable of creating new projects based on an existing project that can be used as a seed, just like you could do with the seed SDM itself.

One of the skills that you'll see with your new SDM is:

```
create spring
```

This intent will generate a new project based on another GitHub repository (by default `atomist-seeds/spring-rest-seed`), and change a couple of things in the POM and code in order to customize it. For example, it will change the package structure to one that you can define and rename some the classes.

So type in:

```
atomist create spring
```

Atomist will first ask you the name of your new repository where you want to create the seed. Next, it will ask for a root package. This package will be used to restructure the packages in the newly created project. It will now start generating the project.

## Atomist doing its thing

Immediately after creating the project, you'll see the Atomist SDM doing work in the feed. If everything works correctly, it will have deployed your new application locally.