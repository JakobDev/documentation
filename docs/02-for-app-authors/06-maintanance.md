# Maintenance

This is a guide in how to maintain your application once it is on Flathub. It assumes your application is already on Flathub, and that you have access rights to its repository. If this is not true, please read the [submission](submission) page first and check your email for GitHub repository access requests.

## The repository

The build information for each application on Flathub is stored in a repository on GitHub in the Flathub organization. For example, the Blender one is here: https://github.com/flathub/org.blender.Blender. On the `master` branch of this repository the primary build version of the app is stored. The `beta` branch is built into the beta repository (if you want to use that). Branches named `branch/XXX` are special cases used for things that have specific flatpak branch names such as extensions. All of these branches are automatically <em>protected</em> which mean that you can only merge pull requests and not push directly to them. Other branch names are free to use however you see fit.

### Required files

Every repository must contain a manifest describing how to build the application. If the application, and hence repository, is called `com.example.MyCoolApp`, this file must be called `com.example.MyCoolApp.json` or `com.example.MyCoolApp.yaml`. This is the only required file!

This manifest may import other manifest files in the same repository. It's quite common to add the [shared-modules](shared-modules) repository as a submodule.

### Optional

- `README.md` with whatever information you see fit, but be advised that users are unlikely to ever read this. However packaging notes are valuable.
- `flathub.json` (described below)

### Acceptable, but should be submitted upstream

- Any patches necessary to build and run the application in the Flatpak environment
- Additional metadata files (`com.example.MyCoolApp.metainfo.xml` and `com.example.MyCoolApp.desktop`) which are mandatory for Flathub apps but not shipped by some (primarily cross-platform) applications

### Strongly discouraged

Patches which add (or remove) application functionality do not belong in the Flathub repository. The application on Flathub should reflect, as closely as possible, the application in its unadulterated form, direct from its authors.

There is a grey area here for functionality which is only appropriate in a Flatpak environment. The policy on Flathub is that this should live in the upstream version of the application, for several reasons:

- The Flatpak environment can be detected at runtime by looking for a file named `/.flatpak-info`.
- Code maintained outside the main application tree is likely to break as the application evolves.
- It is entirely possible to build and distribute Flatpak applications through channels other than Flathub. In particular, tools like [GNOME Builder](https://wiki.gnome.org/Apps/Builder) take advantage of Flatpak to automatically fetch, build and run the application with a single click on any (Linux) computer. To enable this, not only does all functionality need to live upstream, but so too does the Flatpak manifest!

For cross-platform applications which have a policy of not including platform-specific code in their main tree, the recommended approach is for the application author to create a separate repository for the Flatpak (or perhaps desktop Linux in general) version of the app.

## Buildbot

There is a Buildbot instance running on https://flathub.org/builds, which monitors the GitHub repositories. Each time that `master`, `beta` or `branch/*` branch changes it queues a build of the application, and if the build succeeds on all the architectures, then a test repository is generated where you can download and test the build. The build is published (i.e. signed and imported) into the Flathub Flatpak repo manually (via the web ui) or automatically after 3 hours, and the build will be available to your users.

You can track your build status, follow the build log for current and historic builds, start builds or publishe build on the Buildbot instance website.

## Test builds and pull requests

Buildbot also monitors the comments on any pull requests in your repository, and if they include the magic phrase `bot, build` (by a repo collaborator or owner) then it will start a test build. Test builds are similar to regular builds, except the results will never be published into the Flathub repo. You can however install the app from the test repo, where it will be available for 5 days or until you delete it.

This is a great way to do updates, you do an update locally and tests that it works. Then you can make a pull request against master to verify that it builds on all architectures before you merge it.

## `flathub.json`

You can create a file called `flathub.json` to control various parameters of the build infrastructure.

### Limiting the set of architectures to build on.

Flathub has builders for `x86_64`, and `aarch64` as current runtimes (based on Freedesktop.org SDK 20.08 or later) only support `x86_64` and `aarch64`. By default all applications build on all these. If your application does not work on some architectures, you can configure it to skip or build certain architectures.

#### Don’t build on `aarch64`

```json title="flathub.json"
{
  "skip-arches": ["aarch64"]
}
```

#### Only build on `x86_64`

```json title="flathub.json"
{
  "only-arches": ["x86_64"]
}
```

If you build for both `x86_64` and `aarch64` you do not need a `flathub.json` file. There will be no new architecture add or removed on current runtimes, which mean that if that situation ever occured, it would only happen when changing the runtime version in your package.

### End of life

There may come a point where an application is no longer maintained. In order to inform users at update or install time that it will no longer get updates, create `flathub.json` with these contents:

```json title="flathub.json"
{
  "end-of-life": "This application is no longer maintained because..."
}
```

If the application has been renamed, you can additionally include `end-of-life-rebase` with the new ID. Recent flatpak versions will prompt user if they'd like to switch to the renamed app.

```json title="flathub.json"
{
  "end-of-life": "The application has been renamed to the.new.appid.",
  "end-of-life-rebase": "the.new.appid"
}
```

The `end-of-life-rebase` will tell flatpak to automatically migrate the user data from the old package to the new package, making the process transparent for the user.

Please also try to contact a flathub admin to archive the repo by either via pinging them in issues, [Matrix](https://matrix.to/#/#flatpak:matrix.org), or emailing flathub-admins@lists.freedesktop.org

### Download statistics

Flathub publishes download statistics for every app or runtime. The raw JSON files are available at [flathub.org/stats](https://flathub.org/stats/). These break out app downloads and updates. This is also the basis for the data shown on flathub.org, additionally there are some community members that generously provide frontends to interpret the data and make it more useful for app developers at [https://ahayzen.com/direct/flathub.html](https://ahayzen.com/direct/flathub.html) and [https://klausenbusk.github.io/flathub-stats/](https://klausenbusk.github.io/flathub-stats/)

## Getting Help

If anything is not working or there is some behaviour you don’t understand, come to the [Matrix channel](https://matrix.to/#/#flatpak:matrix.org) or start a discussion on the [Flathub forum](https://discourse.flathub.org/).
