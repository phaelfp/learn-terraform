# Build infrastructure

On this page:

- [Build infrastructure](#build-infrastructure)
  - [Prerequisites](#prerequisites)
- [Review the configuration](#review-the-configuration)
  - [Terraform Block](#terraform-block)
  - [Provider](#providers)
  - [Resources](#resources)
- [Initialize the directory](#initialize-the-directory)
- [Format and validate the configuration](#format-and-validate-the-configuration)
- [Create infrastructure](#create-infrastructure)
- [Inspect state](#inspect-state)
- [Next Steps](#next-steps)


With Terraform installed, you are ready to create your first infrastructure.

In this tutorial, you will review the previous configuration for your Docker container.

## Prerequisites

This tutorial assumes that you are continuing from the previous tutorials. If not, follow the steps below before continuing.

Install the Terraform CLI (0.15+), and Docker as described in the [last tutorial](https://developer.hashicorp.com/terraform/tutorials/docker-get-started/install-cli).

Create a directory named `learn-terraform-docker-container`.

```bash
mkdir learn-terraform-docker-container
```

Change into the directory.

```bash
cd learn-terraform-docker-container
```

Create a file to define your infrastructure.

```bash
touch main.tf
```

Open main.tf in your text editor, paste in the configuration below, and save the file.

### MAC or Linux

```tf
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}

```

### Windows

```tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {
  host    = "npipe:////.//pipe//docker_engine"
}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}

```

## Review the configuration
The set of files used to describe infrastructure in Terraform is known as a Terraform configuration.

Each Terraform configuration must be in its own working directory. You created a working directory previously in `learn-terraform-docker-container`. Review the `main.tf` file.

This is a complete configuration that you can deploy with Terraform. In this tutorial, you will learn about each block of this file you deployed previously in more detail.

### Terraform Block
The `terraform {}` block contains Terraform settings, including the required providers Terraform will use to provision your infrastructure. For each provider, the source attribute defines an optional hostname, a namespace, and the provider type. Terraform installs providers from the [Terraform Registry](https://registry.terraform.io/) by default. In this example configuration, the docker provider's source is defined as `kreuzwerker/docker`, which is shorthand for `registry.terraform.io/kreuzwerker/docker`.

You can also set a version constraint for each provider defined in the `required_providers` block. The `version` attribute is optional, but we recommend using it to constrain the provider version so that Terraform does not install a version of the provider that does not work with your configuration. If you do not specify a provider version, Terraform will automatically download the most recent version during initialization.

To learn more, reference the [provider source documentation](https://developer.hashicorp.com/terraform/language/providers/requirements).

### Providers
The `provider` block configures the specified provider, in this case `docker`. A provider is a plugin that Terraform uses to create and manage your resources.

You can use multiple provider blocks in your Terraform configuration to manage resources from different providers. You can even use different providers together. For example, you could pass the Docker image ID to a Kubernetes service.

### Resources
Use `resource` blocks to define components of your infrastructure. A resource might be a physical or virtual component such as a Docker container, or it can be a logical resource such as a Heroku application.

Resource blocks have two strings before the block: the resource type and the resource name. In this example, the first resource type is `docker_image` and the name is `nginx`. The prefix of the type maps to the name of the provider. In the example configuration, Terraform manages the `docker_image` resource with the docker provider. Together, the resource type and resource name form a unique ID for the resource. For example, the ID for your Docker image is `docker_image.nginx`.

Resource blocks contain arguments which you use to configure the resource. Arguments can include things like machine sizes, disk image names, or VPC IDs. Our [providers reference](https://developer.hashicorp.com/terraform/language/providers) documents the required and optional arguments for each resource. For your container, the example configuration sets the Docker image as the image source for your docker_container resource.

## Initialize the directory
When you create a new configuration — or check out an existing configuration from version control — you need to initialize the directory with `terraform init`.

Initializing a configuration directory downloads and installs the providers defined in the configuration, which in this case is the `docker` provider.

If you did not deploy the Quick Start steps in the previous tutorial, initialize the directory now.

```bash
terraform init


Initializing the backend...


Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "~> 3.0.1"...
- Installing kreuzwerker/docker v3.0.1...
- Installed kreuzwerker/docker v3.0.1 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```


Terraform downloads the `docker` provider and installs it in a hidden subdirectory of your current working directory, named `.terraform`. The `terraform init` command prints out which version of the provider was installed. Terraform also creates a lock file named `.terraform.lock.hcl` which specifies the exact provider versions used, so that you can control when you want to update the providers used for your project.

## Format and validate the configuration
We recommend using consistent formatting in all of your configuration files. The `terraform fmt` command automatically updates configurations in the current directory for readability and consistency.

Format your configuration. Terraform will print out the names of the files it modified, if any. In this case, your configuration file was already formatted correctly, so Terraform won't return any file names.

```bash
terraform fmt
```

You can also make sure your configuration is syntactically valid and internally consistent by using the `terraform validate` command.

Validate your configuration. The example configuration provided above is valid, so Terraform will return a success message.

```bash
terraform validate
Success! The configuration is valid.
```

## Create infrastructure
Apply the configuration now with the `terraform apply` command. Terraform will print output similar to what is shown below. We have truncated some of the output to save space.

```bash
terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

   docker_container.nginx will be created
  + resource "docker_container" "nginx" {
#...
      + ports {
          + external = 8000
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

   docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id           = (known after apply)
      + keep_locally = false
      + latest       = (known after apply)
      + name         = "nginx:latest"
      + output       = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

Before it applies any changes, Terraform prints out the execution plan which describes the actions Terraform will take in order to change your infrastructure to match the configuration.

The output format is similar to the diff format generated by tools such as Git. The output has a `+` next to `docker_container.nginx`, meaning that Terraform will create this resource. Beneath that, it shows the attributes that will be set. When the value displayed is `(known after apply)`, it means that the value will not be known until the resource is created. For example, Docker assigns a random ID to images upon creation, so Terraform cannot know the value of the `id` attribute until you apply the change and the Docker provider returns that value.

Terraform will now pause and wait for your approval before proceeding. If anything in the plan seems incorrect or dangerous, it is safe to abort here with no changes made to your infrastructure.

In this case the plan is acceptable, so type `yes` at the confirmation prompt to proceed.

```bash
  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Still creating... [10s elapsed]
docker_image.nginx: Creation complete after 13s [id=sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdeenginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=2834ad6283372ceb61121739ce71d31cb0237ad50f4dc234e3445c9445439181]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

You have now created infrastructure using Terraform! Visit [`localhost:8000`](http://localhost:8000) in your web browser to verify the container started.

## Inspect state
When you applied your configuration, Terraform wrote data into a file called `terraform.tfstate`. Terraform stores the IDs and properties of the resources it manages in this file, so that it can update or destroy those resources going forward.

The Terraform state file is the only way Terraform can track which resources it manages, and often contains sensitive information, so you must store your state file securely and restrict access to only trusted team members who need to manage your infrastructure. In production, we recommend [storing your state remotely](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-migrate) with HCP Terraform or Terraform Enterprise. Terraform also supports several other [remote backends](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) you can use to store and manage your state.

Inspect the current state using `terraform show`.

```bash
terraform show
 docker_container.nginx:
resource "docker_container" "nginx" {
    attach            = false
    command           = [
        "nginx",
        "-g",
        "daemon off;",
    ]
    cpu_shares        = 0
    entrypoint        = [
        "/docker-entrypoint.sh",
    ]
    env               = []
    gateway           = "172.17.0.1"
    hostname          = "2834ad628337"
    id                = "2834ad6283372ceb61121739ce71d31cb0237ad50f4dc234e3445c9445439181"
    image             = "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee"
    init              = false
    ip_address        = "172.17.0.2"
    ip_prefix_length  = 16
    ipc_mode          = "private"
    log_driver        = "json-file"
    logs              = false
    max_retry_count   = 0
    memory            = 0
    memory_swap       = 0
    must_run          = true
    name              = "tutorial"
    network_data      = [
        {
            gateway                   = "172.17.0.1"
            global_ipv6_address       = ""
            global_ipv6_prefix_length = 0
            ip_address                = "172.17.0.2"
            ip_prefix_length          = 16
            ipv6_gateway              = ""
            network_name              = "bridge"
        },
    ]
    network_mode      = "default"
    privileged        = false
    publish_all_ports = false
    read_only         = false
    remove_volumes    = true
    restart           = "no"
    rm                = false
    security_opts     = []
    shm_size          = 64
    start             = true
    stdin_open        = false
    tty               = false

    ports {
        external = 8000
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}

 docker_image.nginx:
resource "docker_image" "nginx" {
    id           = "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdeenginx:latest"
    keep_locally = false
    latest       = "sha256:d1a364dc548d5357f0da3268c888e1971bbdb957ee3f028fe7194f1d61c6fdee"
    name         = "nginx:latest"
}
```

When Terraform created this container, it also gathered the resource's metadata from the Docker provider and wrote the metadata to the state file. Later, you will modify your configuration to reference these values to configure other resources and output values.

## Manually Managing State
Terraform has a built-in command called `terraform state` for advanced state management. Use the list subcommand to `list` of the resources in your project's state.

```bash
 terraform state list
docker_container.nginx
docker_image.nginx
```

# Next Steps
Now that you have created your first infrastructure using Terraform, continue to [the next tutorial](step2.md) to modify your infrastructure.

For more detail on the concepts used in this tutorial:

- Read about the Terraform configuration language in the [Terraform documentation](https://developer.hashicorp.com/terraform/language).
- Learn more about Terraform [provider](https://developer.hashicorp.com/terraform/language/providers).
- Find examples of other uses for Terraform in the documentation [use cases section](https://developer.hashicorp.com/terraform/intro/use-cases).
- Read [the Docker provider documentation](https://registry.terraform.io/providers/kreuzwerker/docker/latest) to learn more.
- For more information about the terraform state command and subcommands for moving or removing resources from state, see the [CLI `state` command documentation](https://developer.hashicorp.com/terraform/cli/commands/state).