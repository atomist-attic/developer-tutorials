# Adding reviews: SDM, the code police

We all know that there are certain things we should not do. And some companies have very specific rules about how code should be written. We've shown before how you can use Checkstyle to add reviews to your code, but you can also be very specific about certain reviews. That is when you start writing your own reviewers.

Say that we want to give someone a notice when they do a star import. We write the following reviewer:

``` typescript
export const ImportDotStarCategory = "Lazy import";

export const ImportDotStarReviewer: ReviewerRegistration = patternMatchReviewer(
    ImportDotStarCategory,
    {globPattern: JavaAndKotlinSource, severity: "info"},
    {
        name: ImportDotStarCategory,
        antiPattern: /import .*\.\*/,
        comment: "Don't import .*. Import types individually",
    },
);
```

Adding the reviewer to the SDM is very simple:

``` typescript
sdm.addReviewerRegistration(ImportDotStarReviewer);
```

Assuming we have a review listener in the SDM (like `sdm.addReviewListener(slackReviewListener());`), we now get notified when someone commits code that has a star import.

You can use reviewers also to call third party services that can review your code and process the results of that review. If you want to limit when reviews are triggered, you can add a `PushTest` as well.

Atomist provides a couple of reviewers out of the box, like for example the `ImportDotStarReviewer`. But there's also the `HardCodedPropertyReviewer` which checks whether you haven't hardcoded certain properties in the `application.properties`. Have a look in the `sdm-pack-spring` for more examples of reviewers.






