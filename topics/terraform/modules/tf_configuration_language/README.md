# HashiCorp Configuration language

<!--TOC_START-->
## Contents
- [Overview](#overview)
- [Modules and Resources](#modules-and-resources)
- [Provider](#provider)
- [Basic syntax](#basic-syntax)
- [Outputs](#outputs)
- [Data source](#data-source)
- [Configuration files](#configuration-files)
- [Order of configuration](#order-of-configuration)
- [Comments](#comments)
- [Formatting](#formatting)
- [Tasks](#tasks)
	- [AWS task](#aws-task)
		- [Prerequisites](#prerequisites)
		- [Authenticating](#authenticating)
		- [Creating the directory and configuration file](#creating-the-directory-and-configuration-file)
		- [Adding the provider](#adding-the-provider)
		- [Adding variables](#adding-variables)
		- [Adding resource](#adding-resource)
		- [Formatting](#formatting-1)
		- [Final configuration file state](#final-configuration-file-state)
		- [Running the configuration file](#running-the-configuration-file)
		- [Clean up](#clean-up)

<!--TOC_END-->
## Overview
Terraform uses it's own configuration language called *Hashicorp Configuration Language (HCL)*. 

It's designed to describe infrastructure in a concise way. 

The language is declarative meaning it describes an intended goal rather than the steps of how to reach that goal.

## Modules and Resources

Main purpose of Terraform language is to declare resources. 

Any additional features that the language has are meant to aid the primary purpose. 

A *resource* describes a single object of infrastructure, but multiple resources can be combined into a *module* where relationships between them could be defined. 

In short a *resource* is a component of your infrastructure.

<details>

<summary>Here is an example of a <i>virtual machine instance</i> resource in AWS:</summary>

```hcl
resource "aws_instance" "example" {
	ami = "ami-2757f631"
	instance_type = "t2.micro"
}
```

</details>

Here is an example project structure where reusable module would be within a project:
```text
.
|---main.tf
|---db.tf
|---backend.tf
|---frontend.tf
|---modules
|---|---fullStackModule
|---|---|---main.tf
|---|---|---frontend.tf
|---|---|---db.tf
|---|---|---backend.tf
```

## Provider

Resources become available through providers. 

When working with providers, there is always a form of credentials required to authenticate with the provider.
This can usually be done by creating an access key on the provider and then using it in a provider configuration in Terraform.

<details>

<summary>Configure AWS as a provider example</summary>

```hcl
provider "aws" {
	access_key = "AKIBIWX7DKIDGMCHPG4A"
	secret_key = "3gSerUT5rreC989K5l4f3WcGZ0yUNaltaw4C8r/1"
	region = "eu-west-2"
}
```

</details>

## Basic syntax

There are three main building blocks to Terraform syntax they are: Arguments, Blocks and Expressions.

**Blocks** 
- block is like a container that holds the configuration for an object like a resource. 
- Blocks have a type. 
- Can contain zero or more labels.
- Body can contain zero or more arguments or nested blocks.
- Majority or Terraform features are controlled from configuration files by top level blocks.

**Arguments** 
- allows to assign a value to a name. 
- They appear within blocks.

**Expressions** 
- are like variables, their purpose is to represent a value, either literally, referencing or combining with other values. 
- Expressions appear like values for arguments as well as within other expressions. 

Now let's take a look at how they would look like in use:
```
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

<details>

<summary>Block resource example with AWS</summary>

```hcl
resource "aws_instance" "example" {
	ami = "ami-2757f631"
	instance_type = "t2.micro"
}
```

</details>

Here is an example of a variable:
```hcl
variable "version_number" {
    type        = "string"
    default     = "0.0.1"
    description = "version number" 
}
```
This example declares a string variable with a description and a default value, if the user wouldn't provide a value then the default value would be used and you wouldn't get an error thrown.

## Outputs

Outputs are infrastructure values you want to easily access. 

For example, the DNS name of a load balancer, the ssh command used to connect to an instance created by the configuration, etc. 

When you apply changes in your configuration Terraform displays the outputs that you configure. 

You can also access outputs at any time using the Terraform output command. 

The output command is useful for integrating Terraform with scripting environments. 

<details>

<summary>Here's an AWS example output:</summary>

```hcl
output "instance_ip_addr" {
  value = aws_instance.server.private_ip
}
```

</details>

This output would tell you the private IP of the instance created.

## Data source

Data sources allow Terraform to fetch information that is defined outside of the current Terraform configuration. 

For example, if you have divided your infrastructure into separate projects, you can retrieve information from other Terraform projects using data sources. 

The information retrieved using data sources can be made available via providers. 

<details>

<summary>Here's an example data source in AWS:</summary>

```hcl
data "aws_ami" "web" {
  filter {
    name   = "state"
    values = ["available"]
  }

  filter {
    name   = "tag:Component"
    values = ["db"]
  }

  most_recent = true
}
```

This data source would look for the latest AMI, which is tagged "db". 

</details>

## Configuration files

Terraform language configuration files uses an extension of `.tf`, an example file name would be `main.tf`. 

Although, that isn't the only supported file extension, there is also the JSON syntax where the files have the extension of `.tf.json`, an example file name would be `main.tf.json`. 
Terraform allows for JSON to be used because it is machine-readable, meaning that Terraform configurations themselves could be automated.

Configuration files need to use the `UTF-8` encoding. 

Even though both windows and unix style line endings are supported you should use the unix style ones. 

This is to make sure that Terraform is reading the configuration file that you made correctly.

Windows line endings are represented as `\r\n` where unix are `\n`.

*Module* is a one or more configuration files in the same directory where the configuration files are either with `.tf` or `.tf.json` extension. 

The root directory would be built from the files and in the in the current working directory. 

Other child modules can be referenced from child directories.

In it's simplest form the root module would contain a single file of `.tf` or `.tf.json` extension. 

Configuration can expand by adding new configuration files within the root module, or by organising resources through child modules.

e.g If you required 3 VM's on AWS but you wanted to have separate configuration files, your projects root directory file structure could look like this:
```
example_2
|---main.tf
|---db.tf
|---backend.tf
|---frontend.tf
```
Where each file would be a separate configuration file for a resource.

## Order of configuration

The order of blocks is not significant as the language is declarative. 

There is only one exception to this rule which is the `provisioner` block.

Resources will be automatically processed in the correct order based on what are the relationships between the resources. 

Hence, it's up to the implementer to decide on how to structure the files and how many to have.

## Comments

There are three types of comments supported:
- `#` used for single line comments
- `//` also used for single line comments, an alternative way to `#`
- `/*` start of the multi line comment where `*/` ends the multi line comment

`#` is the preferred way of leaving comments in the configuration files. 

Formatting tool converts the `//` into `#` comments, the reason for this is that the `//` style is not idiomatic.

## Formatting

Although the amount of whitespace before or after the = sign doesn't have an effect on the configuration file, it's recommended to align them.

Here's an example of = symbols aligned in one line:
```hcl
variable "version_number" {
    type        = "string"
    default     = "0.0.1"
    description = "version number" 
}
```
As you can see the equal signs are in one line, the amount of whitespace before the sign depends on the length of the word, there is also a whitespace after it, although this may seem needles it helps out with readability. 

There's no need to do this manually as there is a command which formats the file automatically:
```
terraform fmt
```

## Tasks
Please complete the task below that is most convenient for you.

### AWS task

<details>

<summary>Deploying a resource with comments and variables in AWS</summary>

You will now create a resource on AWS, using some functionality from this module.

#### Prerequisites

1. Have **aws cli** installed
    2. You can install it by running the following python command, keep in mind you need to have python installed:
    `pip install awscli`
3. Know your AWS `access` and `secret` keys

#### Authenticating
First let's authenticate with aws so that terraform could execute the configuration file, run the following command:
`aws configure`
You will be asked to provide the following things:
* **AWS Access Key ID** this is where you would need to provide your *access* key
* **AWS Secret Access Key ID** this is the *secret* key
* **Default region name** would be **eu-west-2**
You might get asked additionally to specify what formatting you want to use, enter **json**.

#### Creating the directory and configuration file
For the next step create a new folder, you can pick any name for it but a suggested one would be `example_2`.

Within the newly created folder, create a new file called `main.tf`.

Open the `main.tf` with a text editor of your choosing.

#### Adding the provider

Now paste the following contents into the `main.tf` file:
```hcl
provider "aws" {
	region = "eu-west-2"
}
``` 
You may have noticed that there is only the region declared and no `access_key` or `secret_key` declared, this is done on purpose. 

We're doing it in this way so that when you will be uploading these configuration files to GitHub you wouldn't accidentally expose them.

In later steps we will configure the *access* and *secret* keys using *aws cli* which is the more secure way of doing it.

Now let's declare two variables that we'll use for the resource declaration. 

#### Adding variables

Paste the following below the *provider* block:
```hcl
variable "ami" {
	description = "machine image"
	default = "ami-f976839e"
}

variable "type" {
	# instance type
	/*
		This will determine how many CPU and RAM will be assigned
	*/
	default = "t2.micro"
}
``` 
We're declaring two variables, the first variable is called **ami**, and the value it holds is under the **default** argument. 

It also holds a description about this variable.

The second variable is **type**, we can see that it has two types of comments, a single line comment and a multi line comment.

The default argument is holding the value of what will be the type of instance we'll be creating.

#### Adding resource

Paste the following below the variables:
```hcl
resource "aws_instance" "example" {
	ami = var.ami
	instance_type = var.type
}
```

In this resource block we're specifying what amazon machine image to use for the operating system, it's value is held by **ami** argument where the value is received by making a reference to **var**, it allows us to get a specific variables value by referring to it's name. 

In this case when we want to use the value of the **ami** variable we need to make a reference to it like this: `var.ami`.

The second argument is **instance_type** which specifies which machine configuration to use, it will determine how many vCPU's will be assigned as well as the amount of RAM. 

Similarly in order to get the value of the variable we need to make a reference to it like this: `var.type`.

#### Formatting

Format the configuration file by running the command:
```bash
terraform fmt
```

#### Final configuration file state

Your configuration file at the end of all steps should look similar to this:
```hcl
provider "aws" {
  region     = "eu-west-2"
}

resource "aws_instance" "example" {
  ami           = var.ami
  instance_type = var.type
}

variable "ami" {
  description = "machine image"
  default     = "ami-f976839e"
}

variable "type" {
  # instance type
  /*
		This will determine how many CPU and RAM will be assigned
	*/
  default = "t2.micro"
}
```

#### Running the configuration file

Next switch to the terminal, if you have closed it already, re-open it in the directory where the `main.tf` file is located at. 

First let's execute the following command to download the AWS provider plugin so that Terraform can communicate with AWS:

`terraform init`

Next let's execute this to see what Terraform plans on doing:

`terraform plan`

Finally, let's apply the configured resources by executing:

`terraform apply`

Once terraform will give you a prompt about the successful operation in the *AWS console* under *Compute* and then *EC2* check that the resource has been created. 

Make sure that you are within the correct region, otherwise you won't be able to see the resource.

#### Clean up

To delete the created resource run the following command in the terminal, make sure that the terminal is in the directory where `main.tf` is located:
    `terraform destroy` 

Check in the *AWS console* under *Compute* and then *EC2* check that the resource has been deleted.

Make sure that you are within the correct region, otherwise you won't be able to see the resource - even if it wasn't deleted!

</details>
