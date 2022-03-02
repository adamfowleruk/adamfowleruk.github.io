---
layout: page
title:  Level 100 Introduction
categories:
  - "level-100"
---

# CI/CD

CI/CD stands for "Continuous Integration / Continuous Deployment"
and refers to the two broad steps in taking code from a
source code repository (usually Git based) and getting it ready
for running somewhere.

An individual CI/CD process definition is usually called a 'pipeline'.

## Continuous Integration (CI)

CI pipelines typically specify how to build source code
into an executable form, and any tests and packaging
that must be applied to this built code.

It is usual for a CI pipeline to be ran on a new source code
contribution (Pull Request) before accepting it into a
repository. This is to ensure the contribution doesn't
break anything, and all tests for the project still pass.

This stage can also include static and dynamic code analysis
to spot performance and security problems, the generation
of documentation and API definitions based on source code,
or any single-repository actions that need to occur.

### In repo or out of repo CI

There are two common types of CI pipelines in use.

In repo pipelines exist within the code repository to
which the steps apply. If the code is on GitHub, for example,
you may see a .github/workflows folder. An example of this is
the [folder containing a pipeline used to build this website!](https://github.com/adamfowleruk/adamfowleruk.github.io/tree/main/.github/workflows)

Out of repo pipelines exist where an organisation has
several projects that all need to use a common build pipeline
(often called a buildpack) in order to ensure consistent
build, containerisation, and testing standards are applied.

Which you need to use depends on your org. If you're just starting
out, then in-repo are supremely convenient. Once you get to 
multiple teams of developers, however, you may benefit from a 
more standardised approach that means you don't have to keep 
redefining the same workflow per project.

## Continuous Delivery (CD)

Once the thing is built, you need to run the thing somewhere.
This is typically to allow integration testing with other
components...

"WAIT why doesn't Continuous INTEGRATION do that?" - 
continuous integration integrates small software libraries into
a project, but doesn't integrate the built thing to other built
things. This is what CD is for. You need to deploy something
in order to perform this level os integration testing.

"Oh, ok. Carry on!"

The first target for component testing may be in a development 
test environment. Should your new component pass this, then
the code branch may be 'promoted' as ready for a decision to
be published onto a 'pre prod' (Pre production) or 'test'
system. 

A development cluster is typically the target of a CD pipeline.
This takes the latest 'develop' branch of code, grabs the
'debug' built version from the CI process, and runs it alongside
the latest version of the other components in your complex
software project.

A 'pre prod' or 'non prod' or 'qa' or 'pen test' environment
is then used for components which have passed all tests and
where the overall software product manager (NOT project manager)
decides its ready for external review.

Whilst you may think this is for users to review, typically
in an agile project users will have been involved waaaaaay
before this in the dev cycle. So pre-prod testing is typically
reserved for testing performance, running penetration testing
to ensure system security, and testing failover, backup, restore,
upgrade, database migration, and other processes prior to
inflicting the new software version on users in the production
system.

The CD system is what stands between you and pushing out a flawed
release onto your users. So it's crucially important although
few people manage this very well. (personal opinion) This is
perhaps because every organisations environment is different,
and so there's a lot of variation to manage.

## Approaches: Imperative versus declarative

An imperative approach is like a recipe. Do A then B then C and if
the cake rises, happy days!

A declarative approach says "A finished cake depends upon 
ingredients being prepared and placed in the oven for a while. 
Has that happened yet?". You typically specify 'inputs' and 'outputs' and tag which inputs cause a step to be triggered
if the input is changed.

For small projects - like this website using GitHub Actions to 
build it - an imperative approach is fine.

For large projects with many build versions, options,
deployment platforms, and complexity, then using the declarative
model and leaving the ordering of tasks up to your CI/CD
platform is by far the best approach. It's much more stable,
and you can build up your process over time by adding building
blocks.

It won't surprise you to learn that I prefer the declarative
approach for all multi-repository projects.

## Declarative CI/CDtools:-

### [Concourse](https://concourse-ci.org)
The generic thing do-er from and supported by VMware Tanzu (formerly Pivotal Labs). (Declaration of interest: I work at VMware). This is an opensource tool that can run in Kubernetes, on a VM, or on your laptop. 

Particularly good when you need 'remote workers'. A good example of this is needing to run MacOS workers
as they're the only computers that can build MacOS and iPadOS and iOS mobile applications. 

No pipeline build UI, but has a great UI for putting on a monitor
at the edge of your dev team to monitor live progress. We do this
in our VMware Tanzu offices all the time. Here's a couple of
examples:-

- My [Aircraft tracking demo](/level-300/aircraft) has its [own Concourse pipeline](https://concourse.shared.12factor.xyz/teams/main/pipelines/adsb-images)
- Concourse itself uses [Concourse to build itself!](https://ci.concourse-ci.org/teams/main/pipelines/concourse)

Very strong documentation and examples are provided, and a
list of Concourse 'resources' (action types) are provided on
their website so finding plugins is easy. 

## Tekton

The new kid on the block. Think Concourse but re-engineered
to fit in with exactly how Kubernetes works. Inputs/outputs
look eerily familiar if you've used Concourse.

As the new kid on the block, the documentation is god awful
and disjointed. It uses K8S CRDs and K8S yaml format to
define pipelines. This may or may not be desirable to your
teams.

## Imperative CI/CD tools:-

### GitHub Actions

An in-repo options for building your projects. CI tool with no 
UI to create the pipelines. I use this for building this website 
and publishing to GitHub Pages. Simple,
commonly used, and Just Works. The Ronseal of imperative CI.

### GitLab Actions

Another in-repo option. GitLab is opensource, although most
people graduate to the Enterprise (paid for) edition pretty
quickly. CI tool with no UI to create the pipelines. Similar 
to GitHub Actions. Kinda OK if you like GitLab anyway.

### Azure DevOps Actions

Another in-repo option. Very nice UI for defining the
processes. If using Azure, definitely consider this. Many
of my customers use Azure DevOps Actions as it has nice
integration with Azure AD and 2 factor authentication for
their system single sign on. Not as many plugins in my 
experience as GitHub Actions or Concourse.

## Ansible

God awful imperative way of doing things. Uses the concept
of 'playbooks'. Seems easy at first, but as soon as you
go beyond trivial projects, the playbook concept becomes 
quite fragile and hard to maintain.

Ansible can be used for standing up environments, CI and CD.
Because of its broad use it's very common, especially in IBM
shops as RedHat push this very hard in their platform.

Please don't inflict Ansible on your DevOps teams.