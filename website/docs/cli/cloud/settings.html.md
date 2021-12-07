---
layout: "docs"
page_title: "Terraform Cloud Settings - Terraform CLI"
---

# Terraform Cloud Settings

Terraform CLI can integrate with Terraform Cloud, acting as a client for Terraform Cloud's
[CLI-driven run workflow](https://www.terraform.io/docs/cloud/run/cli.html).

To use Terraform Cloud for a particular working directory, you must configure the following settings:

- Add a `cloud` block to the directory's Terraform configuration, to specify
  which organization and workspace(s) to use.
- Provide credentials to access Terraform Cloud, preferably by using the
  [`terraform login`](/docs/cli/commands/login.html) command.
- Optionally, use a `.terraformignore` file to specify files that shouldn't be
  uploaded with the Terraform configuration when running plans and applies.

After adding or changing a `cloud` block, you must run `terraform init`.

~> **Important:** If you are enabling Terraform Cloud for an existing working
directory that already has Terraform state for managed resources (either stored
locally, or in a remote state backend), `terraform init` might prompt you to
rename existing workspaces. See
[Initializing and Migrating](/docs/cli/cloud/migrating.html) for more details.

## The `cloud` Block

The `cloud` block is a nested block within the top-level `terraform` settings
block. It specifies which Terraform Cloud workspaces to use for the current
working directory.

```hcl
terraform {
  cloud {
    organization = "my-org"
    hostname = "app.terraform.io" # Optional; defaults to app.terraform.io

    workspaces {
      tags = ["networking", "source:cli"] # Selects all workspaces that have both tags
      # --- OR: ---
      name = "vpc-us-west" # Selects exactly one workspace
    }
  }
}
```

The `cloud` block has some special restrictions:

- A configuration can only provide one `cloud` block.
- A `cloud` block cannot be used with [state backends](/docs/language/settings/backends/index.html).
  A configuration can use one or the other, but not both.
- A cloud block cannot refer to named values (like input variables, locals, or
  data source attributes).

The `cloud` block only affects Terraform CLI's behavior. When Terraform Cloud
uses a configuration that contains a cloud  block, it ignores it and behaves
according to its own workspace settings.

### Arguments

The `cloud` block supports the following configuration arguments:

* `organization` - (Required) The name of the organization containing the
  workspace(s) the current configuration should use.

* `workspaces` - (Required) A nested block that specifies which remote
  Terraform Cloud workspaces to use. The `workspaces` block must contain
  **exactly one** of the following arguments:

    * `tags` - (Optional) A set of Terraform Cloud workspace tags. You will be able to use
      this working directory with any workspaces that have all of the specified tags,
      and can use [the `terraform workspace` commands](/docs/cli/workspaces/index.html)
      to switch between them or create new workspaces; new workspaces will automatically have
      the specified tags. This option conflicts with "name".

    * `name` - (Optional) The name of a single Terraform Cloud workspace. You will
      only be able to use the workspace specified in the configuration with this working
      directory, and cannot use `terraform workspace select` or `terraform workspace new`.
      This option conflicts with "tags".

* `hostname` - (Optional) The hostname of a Terraform Enterprise installation, if using Terraform
  Enterprise. Defaults to Terraform Cloud (app.terraform.io).

* `token` - (Optional) The token used to authenticate with Terraform Cloud.
  We recommend omitting the token from the configuration, and instead using
  [`terraform login`](/docs/cli/commands/login.html) or manually configuring
  `credentials` in the
  [CLI config file](/docs/cli/config/config-file.html#credentials).

### Organizing Workspaces

You should only connect a configuration to Terraform Cloud workspaces that use
that same configuration.

This is straightforward when using `name` to select a single workspace. But when
using `tags` to specify multiple workspaces, it's important to ensure your tags
will only match workspaces that use this configuration.

This is ultimately determined by the layout and tagging scheme of your Terraform
Cloud organization's workspaces, so we suggest taking care to consider CLI use
when organizing your workspaces.


## Excluding Files from Upload with .terraformignore

When executing a remote `plan` or `apply` in a [CLI-driven run](/docs/cloud/run/cli.html),
a copy of your configuration directory is uploaded to Terraform Cloud. You can define
paths to exclude from upload via a `.terraformignore` file at the root of your
configuration directory. If this file is not present, the upload will exclude
the following by default:

* `.git/` directories
* `.terraform/` directories (exclusive of `.terraform/modules`)

The rules in `.terraformignore` file resemble the rules allowed in a
[.gitignore file](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository#_ignoring):

* Comments (starting with `#`) or blank lines are ignored.
* End a pattern with a forward slash / to specify a directory.
* Negate a pattern by starting it with an exclamation point `!`.

Note that unlike `.gitignore`, only the `.terraformignore` at the root of the configuration
directory is considered.