---
title: "Terraform Versioning"
datePublished: Wed Mar 02 2022 19:07:42 GMT+0000 (Coordinated Universal Time)
cuid: cl09xlaz105torwnv5nw49o4r
slug: terraform-versioning
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1646247858528/CR2eHYcuo.webp
tags: infrastructure, versioning, terraform

---

Hi Here is a quick article on how to make use of terraform providers versions. 

Hashicorp terraform use a declarative syntax. 
It works with a state file that allows him to check for the state of the infrastructure, compare it with the desired state. If the states matches, no modifications will be made. If not then it will update the state according to the newest state. 

You can use providers in your terraform configuration file to specify the infrastructure to modify. In your configuration file, you can create for example a terraform block.
>**Note : **You can modify any block where the version is required. 


```
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version= "1.4.0"
    }
  }
}
```


The *** version ***
argument is set to 1.4.0 in our sample. 
We can specify a version by using the mathematical comparison signs.


- **" > " greater than** : version greater than the specified version 

```
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version= "> 1.4.0"
    }
  }
}
```


- **" < " lesser than :** version lesser than the specified one 
```
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version= "< 1.4.0"
    }
  }
}
``` 
- **" != " not equal to **: any version except the one specified
```
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version= "!= 1.4.0"
    }
  }
}
``` 
- It may happen that you have a bit complex requirements, like a version that's not the 1.4.0 and who is lesser than the 2.0.0. you can 
```
terraform {
  required_providers {
    local = {
      source = "hashicorp/local"
      version= " !=1.4.0, < 2.0.0 "
    }
  }
}
``` 
- You can also make use of the **~>** operator. It will install any version greater than the one you specified.  


In sum to define a version we can either set the version or use comparison operators (">", "<", "!=", "~>"...) for a better flexibility in our choices. 
You can twist this as you please. 

