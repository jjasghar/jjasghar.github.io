---
layout: post
title: "GitHub Actions and Terraform"
date: 2019-11-05 16:32:18
categories: linux sysadmin terraform
---

I was at a recent conference when people were discussing their usage of `terraform`.
If you don't know, [terraform][terraform] has become the defacto standard for declaring
cloud infrastructures in a consistent and repeatable fashion. This is nothing new, and
I've used it to declare [GCP][gcp], [AWS][aws], and even [IBM Cloud][ibm]. What caught my interest was
using terraform to declare [GitHub resources][github] like repositories and team memberships.

## GitHub Terraform

Some one said it was possible and they had played with it. I decided to give it a shot with
my GitHub organization [asgharlabs][asgharlabs]. All in all the basics of it was straight forward:
```hcl
# This is just an example public repository terraform owns from the get-go.
resource "github_repository" "terraform_public_repository" {
  name        = "terraform_public_repository"
  description = "My example repository controlled by terraform public, and archived"

  private      = false
  has_issues   = true
  has_projects = true
  has_wiki     = false

  default_branch = "master"
  archived       = true
}
```

The above would create a repository like [this][publicgithub], and the actual configuration was pretty easy:

```hcl
# Configure the GitHub Provider
provider "github" {
  token        = "${var.github_token}"
  organization = "${var.github_organization}"
}
variable "github_token" {
  default = ""
}

variable "github_organization" {
  default = "asgharlabs"
}
```

I had to create some `ENV` (environment variables) like this, so I could use the `var.github_token`.
```bash
export TF_VAR_github_token=ca8eb690ec656MY_FAKE_TOKEN!!201a9b866e750
export TF_VAR_github_organization=asgharlabs
```

All in all it wasn't too exciting, but the next part was where the real money is.

## GitHub Actions

Ok, let me paint a little picture here. My theory was this: I make a Pull Request to my [asgharlabs][asgharlabsgit]
repository, say add a repository for the organization, and GitHub Actions runs it through some validation.
I check the `terraform plan` and if I agree that it'll do what expect, I can click the green merge button, and it
merges into master completing the workflow with a `terraform apply`.

I've always been a fan of [travisci][travisci], but with GitHub Actions, and the flexibility it has this
was a great opportunity for me to learn how to do this. These are the steps and gotchas I found to figure this out.

### Enabling GitHub Actions

First thing I noticed was that I didn't have [GitHub Actions][actions] enabled on my account, believe it or
not, but it was surprisingly hard to figure out how to sign up for that beta. This won't be forever, but if
you're doing it early like me, there's the link. :)

### Discovering there are _two_ different styles of Actions

I don't know where or when they [announced a different way of doing](https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-github-actions) Actions, but it seems as of writing this
post (2019-11-05) they are in a migration from some `{ }` declarative style to a `yaml` based with no GUI
involved. Honestly it feels closer to what most people are used to, but if you look at some of the documentation
a lot of it still references the `{ }` style and it's very confusing. Keep this in mind on your journey,
and look find examples (the few that are out there, minus [here][examples]) that are `yaml`.

### Structure of the `.github/workflows/` directory

This bit me a couple times, unlike in Travis, where you can have some branching logic in your `.travis.yml`,
each `.yaml` file in your `workflows/` directory is one "path." So for my case, I had to have one file
dedicated to my Pull Request, and one file for when something is merged into master from the Pull Request.
You can take a look [here][workflows] to see the two files I created, `deploy.yml` and `pull_request.yml`.

### Check on Pull Request

Ok, the meat of the learning here was when something happened with Pull Request was submitted/created. The
following is the "Do this when the Pull Request is created."
```yaml
name: 'Terraform Pull Request Workflow'
on:
  pull_request:
jobs:
  validate_terraform:
    name: 'Terraform Validate Actions'
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout'
        uses: actions/checkout@master
```
This has a unique `name:`, `on:` the `pull_request:` and `runs-on: ubuntu-latest`. If you want something different,
GitHub claims, Linux, Windows and MacOS, you can check the [documentation](https://help.github.com/en/github/automating-your-workflow-with-github-actions/configuring-a-workflow#choosing-a-virtual-environment). After all of this then the `steps:` is where the
actual magic happens. If you take a look at the [direct file][pullrequest], the most interesting `step` is this
one:
```yaml
      - name: 'Terraform Plan'
        uses: hashicorp/terraform-github-actions@master
        with:
          tf_actions_version: 0.12.13
          tf_actions_subcommand: 'plan'
          tf_actions_working_dir: '.'
          tf_actions_comment: true
        env:
          GITHUB_TOKEN: REMOVED
          TF_VAR_github_token: REMOVED
```
This one required me to create a secret, in the repository settings called `TF_VAR_github_token`, so the `terraform plan` can work.
It will also comment to the Pull Request, such as [here][planexample] if it passes so you can look in one location.

### Run `apply` on Merge

The next step was to get my `terraform apply` to happen. If you want to see exactly what I did, look [here][deploy], and
I'll explain the important parts here. I learned that there isn't Pull Request "state" for `merged`. This seemed
odd, and it is a more specific state called `pushed to master`. This also triggers if someone pushes directly
to master, which in it's self could be bad, but that's a different blog post. In this "perfect" world I only
plan to use Pull Requests and when I hit the green button that's only when `terraform apply` should happen.
```yaml
name: 'Deploy Terraform Workflow'
on:
  push:
    branches:
      - master
jobs:
  deploy_terraform:
    name: 'Terraform Deploy Actions'
    runs-on: ubuntu-latest
```
As you can see, it's just a specific tree of logic, for the `on:`, which you could change to whatever your workflow is.
As soon as I figured it out, it wasn't so hard to understand what needed to get done, the only other major difference
was the `apply`:
```yaml
      - name: 'Terraform Apply'
        uses: hashicorp/terraform-github-actions@master
        with:
          tf_actions_version: 0.12.13
          tf_actions_subcommand: 'apply'
          tf_actions_working_dir: '.'
          tf_actions_comment: true
        env:
          GITHUB_TOKEN: REMOVED
          TF_VAR_github_token: REMOVED
```
Just like above, it pulls in the same `TF_VAR_github_token` from the secrets store, and that's...really it.

So all in all, this was a fun couple hours playing around with this workflow. Now I can run `terraform` from
my GitHub repository, and in turn make changes to my GitHub Organization. All in all gotta say it's pretty neat.

[actions]: https://github.com/features/actions
[examples]: https://github.com/actions
[workflows]: https://github.com/asgharlabs/github-terraform/tree/master/.github/workflows
[pullrequest]: https://github.com/asgharlabs/github-terraform/blob/master/.github/workflows/pull_request.yml
[planexample]: https://github.com/asgharlabs/github-terraform/pull/2#issuecomment-549969375
[deploy]: https://github.com/asgharlabs/github-terraform/blob/master/.github/workflows/deploy.yml
[terraform]: https://www.terraform.io/
[gcp]: https://www.terraform.io/docs/providers/google/index.html
[aws]: https://www.terraform.io/docs/providers/aws/index.html
[ibm]: https://ibm-cloud.github.io/tf-ibm-docs/v0.18.0/
[github]: https://www.terraform.io/docs/providers/github/
[asgharlabs]: https://github.com/asgharlabs/
[travisci]: https://travis-ci.org/
[publicgithub]: https://github.com/asgharlabs/terraform_public_repository
[asgharlabsgit]: https://github.com/asgharlabs/github-terraform
