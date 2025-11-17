# Deprecation Notice
This repository is deprecated after tssc-sample-templates [PR 197](https://github.com/redhat-appstudio/tssc-sample-templates/pull/197) merged.
There is no need to duplicate scripts that are already present in our runner image.
 

# tssc-sample-jenkins shared library 

This git repository contains a shared library of packaged TSSC Jenkins steps that match the default pipeline in Redhat Trusted Application Pipeline. 
 

## Files
### API Functions (rhtap.groovy)

| Function   |      Description |
| --- | --- | 
|  init () |  run the Init stage of a pipeline  |  
| buildah_rhtap() |   run a container build and create SBOM     |    
| acs_deploy_check() | use ACS to check deployment yaml  |       
| acs_image_check() |  use ACS to validate image |       
| acs_image_scan() | use ACS to scan image |       
| update_deployment() | update the gitops repository - used to deploy build application  |       
| show_sbom_rhdh() | show SBOM between eyecatches  SBOM_EYECATHER_BEGIN/END |       
| summary() | print a summary of the pipeline results |     

    
 
## How-to

### Setup
[Here](https://jenkins.io/doc/book/pipeline/shared-libraries/) are the instructions on how to add the library to Jenkins
This library can also be loaded dynamically directly from the git url in your `Jenkinsfile` which is the default for TSSC `Jenkinsfiles`
```
library identifier: 'RHTAP_Jenkins@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/redhat-appstudio/tssc-sample-jenkins.git'])
```

### Usage
You can install a shared library into the global shared library pool, and load that libary with the following  ```@Library('RHTAP_Jenkins') _``` to import the API.
 

 ### Sample Jenkins file Usage 

 ```
 library identifier: 'RHTAP_Jenkins@main', retriever: modernSCM(
  [$class: 'GitSCMSource',
   remote: 'https://github.com/redhat-appstudio/tssc-sample-jenkins.git'])
   

pipeline { 
    agent any
    environment {
        ROX_API_TOKEN     = credentials('ROX_API_TOKEN')
        ROX_CENTRAL_ENDPOINT = credentials('ROX_CENTRAL_ENDPOINT')
        GITOPS_AUTH_PASSWORD = credentials('GITOPS_AUTH_PASSWORD')
        QUAY_IO_CREDS = credentials('QUAY_IO_CREDS')
    }   
    stages { 
        stage('init.sh') {
            steps {
                script { 
                    rhtap.info ("Init")
                    rhtap.init() 
                }
            }
        } 
        stage('build') {
            steps {
                script { 
                    rhtap.info( 'build_container..') 
                    rhtap.buildah_rhtap()  
                }
            }
        }
        stage('scan') {
            steps {
                script { 
                    rhtap.info('acs_scans' )
                    rhtap.acs_deploy_check()  
                    rhtap.acs_image_check()  
                    rhtap.acs_image_scan()  
                }
            }
        }
        stage('deploy') {
            steps {
                script { 
                    rhtap.info('deploy' ) 
                    rhtap.update_deployment()  
                }
            }
        }
        stage('summary') {
            steps {
                script { 
                    rhtap.info('summary' )  
                    rhtap.show_sbom_rhdh()  
                    rhtap.summary()  
                }
            }
        }
    }
}
```

