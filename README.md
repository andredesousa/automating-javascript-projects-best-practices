# Best practices for automating JavaScript projects

This is a guideline of best practices that we can apply to our software projects.
Continuous Automation is the practice of automating every aspect of an application's lifecycle to build and deploy software and changes quickly, consistently, and safely.
These tips are based on books, articles and professional experience.

## Table of Contents

1. [Choose the right dependencies](#choose-the-right-dependencies)
2. [Lock down your dependencies versions](#lock-down-your-dependencies-versions)
3. [Check for outdated dependencies](#check-for-outdated-dependencies)
4. [No dev dependencies in production](#no-dev-dependencies-in-production)
5. [Secure your projects and tokens](#secure-your-projects-and-tokens)
6. [Use NVM when you work on multiple projects](#use-nvm-when-you-work-on-multiple-projects)
7. [Use .env file for environment configurations](#use-env-file-for-environment-configurations)
8. [Use pre-commit hooks](#use-pre-commit-hooks)
9. [Define your own npm scripts](#define-your-own-npm-scripts)
10. [Identify tests that can be automated](#identify-tests-that-can-be-automated)
11. [Run the fastest tests early](#run-the-fastest-tests-early)
12. [Run the tests locally](#run-the-tests-locally)
13. [Choose the right automation tools](#choose-the-right-automation-tools)
14. [Publish the results of the latest build](#publish-the-results-of-the-latest-build)
15. [Automate the build and deployment](#automate-the-build-and-deployment)
16. [Decouple releases from deployments](#decouple-releases-from-deployments)
17. [Build only once and promote the result](#build-only-once-and-promote-the-result)
18. [Release early, release often](#release-early-release-often)
19. [Keep the pipelines fast](#keep-the-pipelines-fast)
20. [Parallelize whenever possible](#parallelize-whenever-possible)
21. [Smoke-Test the deployments](#smoke-test-the-deployments)
22. [Version control for all](#version-control-for-all)
23. [Do application monitoring](#do-application-monitoring)
24. [Review metrics on a regular basis](#review-metrics-on-a-regular-basis)
25. [Have an alert system](#have-an-alert-system)

## Choose the right dependencies

Using a JavaScript library can help to avoid unnecessary code repetition.
Libraries can abstract away complex logic, such as date manipulation or financial calculations.
When we choose a library, we need to consider several aspects such as performance, security, accessibility, conventions, maintenance, community, documentation, and licensing.
Understanding which pros and which cons apply in different situations is key to vetting the large number of JavaScript library choices that are available

## Lock down your dependencies versions

Even if you save modules with exact version numbers as shown in the previous section, you should be aware that most npm module authors don't.
It's totally fine, they do it to get patches and features automatically.
The situation can easily become problematic for production deployments.
It's possible to have different versions locally then on production, if in the meantime someone just released a new version.
The problem will arise, when this new version has some bug which will affect your production system.
To solve this issue, you may want to use `package-lock.json` file.
Once you have this file in place, `npm ci` will use it to reproduce the same dependency tree.

## Check for outdated dependencies

To check for outdated dependencies, npm comes with a built-in tool method the `npm outdated` command.
You have to run in the project's directory which you'd like to check.
By default, only the direct dependencies of the root project and direct dependencies of your configured workspaces are shown.
Use `--all` to find all outdated meta-dependencies as well.
If you are looking for an easy way to update outdated packages, you can use `npm update`.
However, be aware that `npm update` doesn't update to MAJOR versions.

## No dev dependencies in production

Development dependencies are called development dependencies because we don't have to install them in production.
It makes your deployment artifacts smaller and more secure, as you will have less modules in production which can have security problems.
To install production dependencies only, we can use `npm ci --production` command.
Alternatively, you can set the `NODE_ENV=production` environment variable.

## Secure your projects and tokens

In case of using npm with a logged in user, your npm token will be placed in the `.npmrc` file.
As a lot of developers store dotfiles, sometimes these tokens get published by accident.
If you have dotfiles in your repositories, double check that your credentials are not pushed.
Another source of possible security issues are the files which are published to npm by accident.
By default npm respects the `.gitignore` file, and files matching those rules won't be published.
However, if you add an `.npmignore` file, it will override the content of `.gitignore`.
In CI/CD environments, you can protect your tokens with environment variables.
`NPM_CONFIG__AUTH` and `NPM_CONFIG_REGISTRY` environment variables can help you with this task.

## Use NVM when you work on multiple projects

Different projects may require different Node runtimes/versions to be built, and developers are probably working on several different projects on their local machines at a time that may require incompatible Node versions.
[NVM](https://github.com/nvm-sh/nvm) allows you to have different versions of Node on your machine, and to easily switch between versions as necessary.
Additionally, by setting up shell integration and adding a `.nvmrc` file to your project, your shell will automatically change to the Node version required by your project when you navigate into it.

## Use .env file for environment configurations

In keeping with the [12 Factor App](https://12factor.net/) methodology, it's best that your app gets any configuration information it may need from the environment (e.g. production vs staging).
Things that vary depending on the environment as well as sensitive things like API keys and DB credentials are great candidates for being provided via the environment.
In Node, environment variables can be accessed via `process.env.<ENV_VAR_NAME_HERE>`.
For local development the usage of a `.env` file along with the [dotenv](https://www.npmjs.com/package/dotenv) package is very common and easy for developers.
This `.env` file should be added to your `.gitignore` file because it contains sensitive information.

## Use pre-commit hooks

Sometimes you'll want to run some commands before a developer can commit to your repository.
Having Prettier and ESLint adjust and fix all JavaScript files that have been staged for commit is a great example.
[lint-staged](https://www.npmjs.com/package/lint-staged) allows to run linting commands on files that are staged to be committed.
When lint-staged is used in combination with [husky](https://www.npmjs.com/package/husky), the linting commands specified with lint-staged can be executed to staged files on pre-commit.
Unfortunately, it is not enough to only rely on lint-staged and husky to prevent linting errors because the git hooks can be by-passed if a user commits with the `--no-verify` flag.
Therefore, it is also recommended to run a command on a CI server that will verify that there are no linting errors.

## Define your own npm scripts

The `package.json` file contains a section called scripts.
It is an object where you can specify various commands and scripts that you want to expose.
They are used to automate tasks like minifying CSS, uglifying JavaScript, building project.
We must define here the tasks that need to be automated and the commands that we need to run in the CI/CD pipeline.
Following naming conventions, common npm scripts are `start`, `lint`, `test`, `e2e`, `clean` and `build`.
Other useful commands are `lint:fix`, `test:watch`, `test:debug`, `e2e:watch` and `e2e:debug`.

## Identify tests that can be automated

It is unlikely that 100% of the tests can be automated since there would at least be some tests where manual testing would be more effective when compared to automation tests.
Since test automation is core to CI/CD pipeline, realizing those test cases which can be automated is a crucial best practice for CI/CD.
We can automate tests that are executed on a frequent basis and tests that require knowledge and depend on a specific set of testers.

## Run the fastest tests early

While keeping the entire pipeline fast is a great general goal, parts of the test suite will inevitably be faster than others.
Discovering failures as early as possible is important to minimize the resources devoted to problematic builds.
To achieve this, prioritize and run the fastest tests first.
Save complex, long-running tests until after we have validated the build with smaller, quick-running tests.
Test prioritization usually means running the project's unit tests first since those tend to be quick, isolated, and component focused.
Afterwards, integration tests typically represent the next level of complexity and speed, and finally e2e tests, which often require some level of interaction.

## Run the tests locally

Developers should be encouraged to run as many tests as possible locally prior to committing to the repository and/or integration branch.
This makes it possible to detect certain problematic changes before they block other team members.
To ensure that developers can test effectively on their own, the test suite should be runnable with a single command that can be run from any environment.
The same command used by developers on their local machines should be used by the CI/CD system to kick off tests on code merged to the repository.

## Choose the right automation tools

There are several tools available for CI/CD, but we should choose the right tool based on our budget, requirements, and experience.
Some of the commonly used CI/CD tools are [Jenkins](https://www.jenkins.io/), [GitHub Actions](https://docs.github.com/actions), [Argo CD](https://argoproj.github.io/argo-cd/), etc.
Other tools provide testing platforms that enables developers to test their websites and mobile applications across on-demand browsers, operating systems and mobile devices.
[Selenium Grid](https://www.selenium.dev/documentation/en/grid/) is the standard tool for cross browser testing.
Tools like [Ansible](https://www.ansible.com/), [Helm](https://helm.sh/) or [Terraform](https://www.terraform.io/) are used to manage application configuration and deployment in a way called Infrastructure as Code.

## Publish the results of the latest build

It should be easy to find out whether the build breaks.
All modern CI servers have the capability to display a dashboard containing the status of the builds and send email notifications when the build completes.
It is recommended that the emails are sent to the whole team when the build fails so that it can be fixed as soon as possible.
The various metrics that can be derived using the CI system by installing extensions can be made use of to improve not just the quality of the software, but also the quality of the development practices.

## Automate the build and deployment

In most situations, it is possible to write a script to build and deploy the application to a live test server that everyone can look at.
Automation of the build should include steps such as compiling the code, executing unit tests and integration tests.
They may also include several other tools as code quality checks, semantic checks, measuring technical debt, etc.
[Continuous Deployment](https://www.atlassian.com/continuous-delivery/continuous-deployment) is the next logical step after the process for CI/CD is in place and working well.
However, not all projects need automated deployment and not all commits result in a shippable product.

## Decouple releases from deployments

When deployment and release are highly coupled, deployment operations could be painful and dangerous.
Adding a deployment stage before release for customers allows us to do smoke tests.
Also, you can implement [blue‑green deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html), [canary](https://martinfowler.com/bliki/CanaryRelease.html) or [A/B testing](https://en.wikipedia.org/wiki/A/B_testing).
It is recommend using the term Deployment when referring to the act of deploying a change to application components or infrastructure.
The term Release should be used when a feature change is released to end users, with a business impact.
Using techniques such as [feature toggles](https://martinfowler.com/articles/feature-toggles.html) and [dark launches](https://martinfowler.com/bliki/DarkLaunching.html), we can deploy changes to production systems more frequently without releasing features.

## Build only once and promote the result

If software requires a building, packaging, or bundling step, that step should be executed only once and the resulting output should be reused throughout the entire pipeline.
This guideline helps prevent problems that arise when software is compiled or packaged multiple times, allowing slight inconsistencies to be injected into the resulting artifacts.
CI systems should include a build process as the first step in the pipeline that creates and packages the software in a clean environment.
The resulting artifact should be versioned and uploaded to an artifact storage system to be pulled down by subsequent stages of the pipeline, ensuring that the build does not change as it progresses through the system.

## Release early, release often

[Release early, release often](https://en.wikipedia.org/wiki/Release_early,_release_often) is a software development philosophy.
It emphasizes the importance of early and frequent releases in creating a tight feedback loop between developers and testers or users, contrary to a feature-based release strategy.
Frequent releases are only possible if the software is in a release-ready state and we have tested it in a production-like environment.
It provides quick feedback, faster release cycles, less pressure and improves the user experience.

## Keep the pipelines fast

Pipelines help shepherd changes through automated testing cycles, out to staging environments, and finally to production.
keeping our pipelines fast and dependable is incredibly important to not inhibit development velocity.
There are some straightforward steps we can take to improve speed, like scaling out our CI/CD infrastructure and optimizing tests.
Sometimes, paring down the test suite by removing tests with low value or with indeterminate conclusions is the smartest way to maintain the speed required by a heavily used pipeline.

## Parallelize whenever possible

Pipeline offers a straight-forward syntax for branching the pipeline into parallel steps.
Parallel builds speed up the build/test feedback loop, allowing the developers to be more productive, speed up deployments, so we can deploy more often.
Pipelines that build multiple artifacts could also build those artifacts in parallel.
When adding parallelization to the jobs, we should always monitor them to make sure it really speeds things up.
In some scenarios, heavy consumption of resources on the machine, for example, CPU, GPU, etc. may delay the execution of other operations running on the machine.

## Smoke-Test the deployments

Smoke testing allows us to quickly assess the status of an application by running a set of end-to-end tests targeted at checking the most important, or the most significant, user flows.
It should be run just after a fresh deploy and ideally at regular intervals after that.
It will give us the confidence that our application runs and passes basic diagnostics.
Smoke testing should be fast compared to end-to-end testing and the coverage is wide and shallow.

## Version control for all

Everything from source code and configuration to infrastructure and the database should be version controlled.
Most of these techniques are better known as X as code, such as Pipeline as Code, Infrastructure as Code, and so forth.
Every change a developer makes must be documented in the source control repository or it is not included in the CI/CD process.
If a developer copies an artifact generated locally to a test environment, the next deployment will override this out-of-process change.
That's why the primary golden rule for effective Continuous Deployment is to version-control everything.
What works well for code also will preserve the integrity of configuration, scripts, databases, website html and even documentation.

## Do application monitoring

It can provide request and response information, database connection information, remote profiling and tracing for slow spots, and other metrics related to the health of the services.
Before you start monitoring your Node services, you need to add instrumentation to them via one of the [Prometheus](https://prometheus.io/) client libraries.
As an integral part of application management, logs contain precious information about the events that happen in the application.
Logs are generated all the time and provide more information than metrics.

## Review metrics on a regular basis

The thresholds we establish might be too high or too low.
If we're receiving too many alerts or we aren't notified when a legitimate issue occurs, it's time to review the threshold settings.
There are countless metrics that we can choose to monitor, and it can be overwhelming to separate the signal from the noise coming in.
The [RED Method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/) provides a general framework for monitoring the health of a request-based service via three metrics: Rate, Errors and Duration.

## Have an alert system

Alerts allow us to identify problems in the system moments after they occur.
By quickly identifying unintended changes to the system, we can minimize service disruptions.
Alerts consists in alert rules and notification channel.
We should always aim to achieve the fastest and most efficient resolution for each type of alert we set up.
This is tied in with prioritization and should consider escalation.

## Bibliography

- [16 CI/CD Best Practices To Speed Up Test Automation](https://www.lambdatest.com/blog/16-best-practices-of-ci-cd-pipeline-to-speed-test-automation/)
- [22 Reasons Why Test Automation Fails For Your Web Application](https://dzone.com/articles/22-reasons-why-test-automation-fails-for-your-web)
- [An Introduction to CI/CD Best Practices](https://www.digitalocean.com/community/tutorials/an-introduction-to-ci-cd-best-practices)
- [An Introduction to Metrics, Monitoring, and Alerting](https://www.digitalocean.com/community/tutorials/an-introduction-to-metrics-monitoring-and-alerting)
- [Best practices for creating a modern npm package](https://snyk.io/blog/best-practices-create-modern-npm-package/)
- [Choose a JavaScript library or framework](https://web.dev/choose-js-library-or-framework/)
- [Continuous Integration Tools](https://www.atlassian.com/continuous-delivery/continuous-integration/tools)
- [CI/CD concepts](https://docs.gitlab.com/ee/ci/introduction/)
- [npm Best Practices](https://blog.risingstack.com/nodejs-at-scale-npm-best-practices/)
- [Some Node/JS package best practices](https://www.useanvil.com/blog/engineering/node-package-best-practices/)
