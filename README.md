# terragrunt-run-all-with-better-output-logs

A custom Terraform wrapper for Terragrunt to save the output into a file for each executed Terraform command.

## Problem Set

By default, when running `terragrunt run-all (apply|destroy) ...`, there is no way for the built-in solution to write
the output into the log file. When piping the output to a log file from `terragrunt` command, all module logs are saved
into the same log file, resulting in unreadable logs when running against many modules:

```shell
terragrunt run-all apply 2>&1 | tee tfapply.log  # NOT READABLE LOGS
```

## Solution

To ensure each module has its own log files, Terragrunt can be configured to use this custom Terraform wrapper to write
the output to both the console and the log file:

```shell
terragrunt run-all plan --terragrunt-tfpath loggable-terraform  # BRILLIANT!
```

This will create log files in each module's cache dir, ex:

```text
module-1/.terragrunt-cache/.../(tf-init.log|tf-plan.log|tf-apply.log|tf-destroy.log)
module-2/.terragrunt-cache/.../(tf-init.log|tf-plan.log|tf-apply.log|tf-destroy.log)
module-3/.terragrunt-cache/.../(tf-init.log|tf-plan.log|tf-apply.log|tf-destroy.log)
```

Note: Full credit to @justyns. [See issue.](https://github.com/gruntwork-io/terragrunt/issues/74#issuecomment-471082576)

## Getting Started

- Make Terraform wrapper executable:

```shell
chmod +x terraform-wrappers/loggable-terraform  
```

- Add `terraform-wrappers/` dir into `PATH` environment variable:

```shell
export PATH=path/to/terraform-wrappers:$PATH  
```

- Run `terragrunt run-all` with the Terraform wrapper, ex:

```shell
terragrunt run-all apply --terragrunt-tfpath loggable-terraform  
terragrunt run-all destroy --terragrunt-tfpath loggable-terraform  
```

- The log files are located in each module's `.terragrunt-cache/` dir:

```text
modules
├── module-1
│   ├── .terragrunt-cache
│   │   └── [HASH]
│   │       └── [HASH]
│   │           ├── ...
│   │           └── project
│   │               ├── ...
│   │               ├── backend.tf
│   │               ├── main.tf
│   │               ├── output.tf
│   │               ├── terragrunt.hcl
│   │               ├── tf-apply.log    <=== Generated log file
│   │               ├── tf-init.log     <=== Generated log file
│   │               ├── tf-plan.log     <=== Generated log file
│   │               ├── variables.tf
│   │               └── versions.tf
│   └── terragrunt.hcl
└── module-2
    ├── .terragrunt-cache
    │   └── [HASH]
    │       └── [HASH]
    │           ├── ...
    │           └── project
    │               ├── ...
    │               ├── backend.tf
    │               ├── main.tf
    │               ├── output.tf
    │               ├── terragrunt.hcl
    │               ├── tf-apply.log    <=== Generated log file
    │               ├── tf-init.log     <=== Generated log file
    │               ├── tf-plan.log     <=== Generated log file
    │               ├── variables.tf
    │               └── versions.tf
    └── terragrunt.hcl
```