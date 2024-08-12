## Jenkins Credential Manipulation Groovy Scripts

- This repository contains Groovy scripts designed to simplify the management of Jenkins credentials. These scripts offer convenient methods to handle credentials without dealing with the complexities of the Jenkins API.

### Use Cases

- **Implementation in Jenkins Jobs**

  - Integrate these scripts directly into your Jenkins workflows for seamless credential management.

- **Quick Decryption for Debugging**

  - Decrypt credentials quickly to facilitate troubleshooting and debugging.

- **Migration of Secrets**

  - Easily transfer credentials from one Jenkins server to another, simplifying server migrations.

- **Helm Implementation**
  - Utilize these scripts to manage credentials within Helm charts, streamlining the deployment process.

<h2>Decrypt all BasicSSHUserPrivateKey Credentials</h2>
 
* This script will decrypt all `BasicSSHUserPrivateKey Credentials` stored in Jenkins Credential Store

```groovy
def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
    com.cloudbees.plugins.credentials.common.StandardUsernameCredentials.class,
    Jenkins.instance,
    null,
    null
)

for(c in creds) {
  if(c instanceof com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey){
    println(String.format("id=%s desc=%s key=%s\n", c.id, c.description, c.privateKeySource.getPrivateKeys()))
  }
  if (c instanceof com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl){
    println(String.format("id=%s desc=%s user=%s pass=%s\n", c.id, c.description, c.username, c.password))
  }
}
```

<h2>Decrypt Keys for AWSCredentials</h2>
 
* This script will decrypt `AWSCredentials` stored in Jenkins Credential Store

```groovy
import com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl

def getKeys = { id ->
    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl.class,
        Jenkins.instance
    )

    def c = creds.findResult { it.id == id ? it : null }

    if ( c ) {
        println "found credential ${c.id}"
        println(c.accessKey)
        println(c.secretKey)

    } else {
      error("could not find credential for ${id}, failing build ...")
    }
}
getKeys('${credential_id}')
```

<h2>Decrypt Keys for StringCredentials</h2>
 
* This script will decrypt `StringCredentials` stored in Jenkins Credential Store

```groovy
import org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl
def getSecret = { id ->
    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.plugins.credentials.common.StandardCredentials.class,
        Jenkins.instance
    )

    def c = creds.findResult { it.id == id ? it : null }

    if ( c ) {
        println "found credential ${c.id}"
        println (c.secret)

    } else {
      error("could not find credential for ${id}, failing build ...")
    }
}
getSecret("${credential_id}")
```

<h2>Decrypt Keys for Jenkins Secrets</h2>
 
* This script will decrypt secrets stored in Jenkins xml

```groovy
import hudson.util.Secret;
println Secret.decrypt("xxxx"); //Find value in plugin xml in jenkins server
```
