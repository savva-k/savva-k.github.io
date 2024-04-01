---
title: "Using sed in Jenkins Groovy pipeline"
date: "2020-05-22"
author: "Savva Kodeikin"
categories: ["DevOps"]
tags: ["Sed", "Jenkins", "Groovy"]
image: images/sed_vs_jenkins.png
---

I spent some time trying to figure out how to fix **unexpected slash char** error in my pipeline.

My aim was to change some value in a config file after checking out a Git repository. I have a Jenkins pipeline and I used sed to perform the substitution.

The pipeline wouldn't even compile because of Groovy's way of handling slashes. I was constantly getting "unexpected slash char" error.

Eventually, I made it work this way:

```groovy
stage('Some stage') {
    steps {
        sh script: $/
            sed -i 's/^\(SOME_VAR=*\).*$/\1SOME_VAL/' ./config/default.conf
        /$
    }
}
```

Having the following default.conf:

```properties
SOME_VAR=Literally any string value
```

The pipeline's step will turn it into:

```properties
SOME_VAR=SOME_VAL
```

Good job, I think :-)
