---
title: "Terraform Variables "
slug: terraform-variables

---

Variables are value that we can set to be defined by the users. They generally have a default value. If not then when we are executing the command **terraform apply** we will be asked to insert the values of the empty variables before applying our changes. 

Where can we define variables ? 
In a file named 
+ **variables.tf**
+ **terraform.tfvars**
+ **as environment variables**
+ **with the -var while running commands** 

Those variables are set in a variable block within the configuration file we choose to use. 
```
variable "docker_ports"{
  type = list(object({
    internal = number
    external = number
}))
  default = [
    {
      internal = 5000
      external = 5200
    }
  ]
}
```
We have the name of the variable **docker_ports**, we have set a **type** and some **default** values. It is possible to set an empty variable without default values but it's not a good practice.
>Good practice : set description to your variables.

##Variable Types##

Variables can be of any of this types : 
- string : It accept alphanumerical values, characters and so on. 
- number: any numerical value positive or negative
- bool : boolean values either true or false
- list : they uses indexes from 0 to identify a specific list values.
```
variable "servers" {
  type = list
  default = ["webapp1", "webapp2", "bd1"] 
}
```
>In the sample  we can access to a **webapp2** like this : var.server[1]
- map : data represented in the format of key value pairs.
- set : it's like a list without duplicate elements. 
- object : 
- tuple : its like a list where elements can be of different types. 



There is a an order of precedence for terraform variables. 
- Environment variables
- The terraform.tfvars file, if present.
- The terraform.tfvars.json file, if present.
- Any *.auto.tfvars or *.auto.tfvars.json files, processed in lexical order of their filenames.
- Any -var and -var-file options on the command line, in the order they are provided. (This includes variables set by a Terraform Cloud workspace.)

output variables##

They can be use to display information about a resource, they are displayed each time we run the commands
They are declared using an **output** block 
```
terraform apply
```
To display all the outputs variables we can use the command 
```
terraform output
```
If we want to specify one output we will have to put the name after the output command 
```
terraform output ipv4_address
```
You can get more info [here](https://www.terraform.io/language/values/variables)