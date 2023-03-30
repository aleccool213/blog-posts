---
title: "Version management with the changesets NPM package"
datePublished: Tue Dec 27 2022 13:31:07 GMT+0000 (Coordinated Universal Time)
cuid: clfv3hako000v09l9h8sufpbd
slug: version-management-with-the-changesets-npm-package
canonical: https://blog.logrocket.com/version-management-changesets/
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/sJa0qmawWnM/upload/c0297949de7f7893406a1ea4f97ae784.jpeg
tags: opensource, npm, monorepo

---

One of the most frustrating aspects of developing software can be upgrading packages. Vague release notes can bog down the process and make it hard to know how to upgrade. Package maintainers use a pattern called [Semantic Versioning](https://semver.org/) (semver) to describe changes in new versions. It tells consuming applications how to handle updates. If a project is small, maintainers have an easy time choosing the semver type for a release. Performing other tasks like writing release notes also doesn't take much time. In contrast, when a project gets larger, maintainers find this to be a time-consuming task. For example, [Jest](https://github.com/facebook/jest) is a large mono-repo with packages that depend on each other, are public, and are consumed individually. With pull requests merging frequently, maintainers have a tough time figuring out what gets shipped in a package release. Maintainers need to track merging changes, document how apps should upgrade, update packages within the repository and publish packages. There are tools to make this easier, with one of the prominent ones being [changesets](https://github.com/changesets/changesets).

### changesets

Without using a tool, maintainers can provide checklists in pull request descriptions to remind contributors to add details to their changes. The goal is to reduce the burden and the work a maintainer has to do. This strategy is decent but reviewers miss things and it still leaves work to the maintainer to merge everything into a single release/document when multiple pull requests are merged. They could have little context as the time between merges and when a package gets released could be weeks. Maintainers want to encourage many contributions and want to make them seamless and easy. changesets is a tool created to help with the work of version management inside mono-repos. It provides a CLI interface for contributors to describe their changes in a pull request alongside the semver bump type. These bits of information are called, aptly, “changesets”. The tool is then used to perform versioning, which includes consuming all of the “changesets” since the last release, finding out the maximum semver bump type, updating the changelog and updating the appropriate internal packages.

### Example

To demonstrate the capabilities and advantages of using changesets, we can see how different changes to a codebase are handled with and without it. There is an open-source e-commerce store platform that is specifically made for pet stores. The team behind the application is having a hard time organizing the code and wants to make parts of the code base more usable for different applications. They decide a mono-repo suits this well and split up pieces of code they see as generic into different packages.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147166126/9e493fa7-9a24-4aeb-ae12-cabdebbd7bde.png align="center")

Packages like `ordering` become adopted outside the codebase and make the lives of other developers easier by handling the ordering of items in their stores. During the course of the next week, developers on the core team and outside contributors make changes to the package. An internal developer adds a new feature that makes it easy to update the stock of an item.

```jsx
// Updates the stock of an item in the database
export function updateStock(item, quantity) {}
```

This change requires a minor semver bump as apps or packages that get this update can safely upgrade with no breaking changes to any existing APIs.

This is not the only change that happens, an external contributor reports a critical bug where `sku` cannot be entered into the database when items are being inserted. `sku` is now a required field on the `addItem` function.

```jsx
// Adds an item in the database
export function addItem(item, sku) {}
```

This change requires a major semver bump as apps or packages must make code changes to upgrade to this new version safely.

Let’s see how these two changes can be released with and without changesets.

### Without changesets

After a recent release, an external developer opens a pull request to the monorepo package `ordering` to add a new feature:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147062559/7790b4f1-7dbb-4192-8fa7-8d583ef87e3b.png align="center")

The maintainer of the repository is pleased but notices that the contributor didn’t update the release notes in the pull request. This is outlined in the guidelines but contributors sometimes miss this which isn’t to fault them, they are focused on making changes.

The maintainer asks for the contributor to add the notes:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147029802/84f4006e-be1e-4cd6-9f1c-f0baaa91248c.png align="center")

This change along with other changes gets merged and the maintainer proceeds to release the package a few weeks later. As part of this process, they need to figure out the semver bump type. They need to look over every pull request and see if there were any breaking changes, feature additions and/or bug fixes. They find the `sku` addition to `addItem` change and decide a full version bump is needed. As part of the release they then need to find every package which depends on the `ordering` package and bump it from 1.0.0 to 2.0.0, like the `pet-store` app. The maintainer is tasked with upgrading the app to comply with the changes. It takes a while as the contributor didn’t write much documentation for it.

Afterwards they need to create a change-log entry, by again, going through all of the pull requests merged since the last release. This alongside possibly other tasks is making the maintainer's job tedious, prone to error, and cumbersome.

### With changesets

The contributor runs `yarn changeset` and creates a “changeset”. They describe the new `updateStock` function, how consuming packages can use, which package was affected by the change, and the semver bump type. They push the change and open a pull request.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147107847/79a0ea00-a768-455b-bcc7-29e409afde18.png align="center")

The maintainer reviews the pull request and merges it, no questions asked! The “changeset” file contained everything that is needed to make the release process easy. After this, other pull requests come in and one includes a major version bump, this is due to the `sku` argument addition to `addItem`.

```markdown
---
"changesets-package-ordering": major
---

Add sku argument to addItem function

The `sku` argument was added to the `addItem` function. It is a required argument due to changing business needs.

Example usage:
...
```

Now the maintainer is ready to release the next version of the `ordering` package. To do this, all they must do is run the `yarn changeset version` command. This command removes all of the local “changeset” files, creates the change-log entry, and automatically bumps the version of the `ordering` package in the package itself and in the dependants like the `pet-store` app.

```jsx
## 2.0.0

### Major Changes

- ea13cc5: Add sku argument to addItem function

  The `sku` argument was added to the `addItem` function. It is a required argument due to changing business needs.

### Minor Changes

- 7034f8a: Add the updateStock utility function

  The `updateStock` utility function is used by applications to update the stock of an item.
```

Because the contributor needed to write detailed documentation for their breaking change, the maintainer has an easy time upgrading the `pet-store` app to comply with the changes.

With the help of changesets, the maintainer was easily able to create a release and update apps across the monorepo in a standardized and reproducible way.

### Automation

We can make working with changesets even easier with automation. We always want to ensure that the correct steps are being followed by contributors and maintainers.

We can make sure contributors always create a “changeset” by installing [the changesets Github Bot](https://github.com/apps/changeset-bot). This bot will comment on pull requests when one is missing:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147132946/b20f3ccd-c63f-4937-92b2-70f57da80590.png align="center")

We will then also want to ensure that no pull requests can be merged without a “changeset” by running the `yarn changeset status` inside of a Github Action. This can fail when contributors forget to create a changeset.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680147152770/cfb82f48-8534-4a33-99fa-fe626376e23c.png align="center")

When a new version is ready to be created, maintainers can further automate this step by utilizing the `changesets` Github Action. This action will run the `yarn changeset version` command mentioned earlier and create a pull request with all of the file changes it creates.

### In Conclusion

Using the changesets package, many of the steps which make the release process a burden on maintainers can be lifted. Contributors can be reminded to make detailed notes about their changes and maintainers can easily create new versions of packages using a single CLI command. With high-quality and recurring package releases, consumers will have an easy time upgrading to new versions.