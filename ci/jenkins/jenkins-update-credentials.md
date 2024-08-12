
<h2>Updating Keys for AWSCredentials</h2>
 
* This script updates `AWSCredentials` stored in Jenkins Credentials Store

```groovy
import com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl

def updateKeys = { id, new_AccessKey, new_SecretKey ->
    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.jenkins.plugins.awscredentials.AWSCredentialsImpl.class,
        Jenkins.instance
    )

    def c = creds.findResult { it.id == id ? it : null }

    if ( c ) {
        println "found credential ${c.id}"
        def credentials_store = Jenkins.instance.getExtensionList(
            'com.cloudbees.plugins.credentials.SystemCredentialsProvider'
            )[0].getStore()

        def result = credentials_store.updateCredentials(
            com.cloudbees.plugins.credentials.domains.Domain.global(),
            c,
            new AWSCredentialsImpl(c.scope, c.id, new_AccessKey, new_SecretKey, c.description)
            )
        println "Updated credential ${c.id}"

    } else {
      error("could not find credential for ${id}, failing build ...")
    }
}  
updateKeys('${credentialId}', '${accessKey}', '${secreKey}')
```

<h2>Updating Password for UsernamePassword Credentials</h2>
 
* This script will update the Password of a `UsernamePasswordCredentials` stored in the Jenkins Credential Store

```groovy
import com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl
def changePassword = { id, new_password ->
    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.plugins.credentials.common.StandardUsernameCredentials.class,
        Jenkins.instance
    )

    def c = creds.findResult { it.id == id ? it : null }

    if ( c ) {
        println "found credential ${c.id}"

        def credentials_store = Jenkins.instance.getExtensionList(
            'com.cloudbees.plugins.credentials.SystemCredentialsProvider'
            )[0].getStore()

        def result = credentials_store.updateCredentials(
            com.cloudbees.plugins.credentials.domains.Domain.global(),
            c,
            new UsernamePasswordCredentialsImpl(c.scope, c.id, c.description, c.username, new_password)
            )

        if (result) {
            println "password changed for ${id}"
        } else {
            error("failed to change password for ${id}, failing build ...")

        }
    } else {
      error("could not find credential for ${id}, failing build ...")
    }
}
changePassword('${credID}', '${RANDOM_STRING}')
```

<h2>Updating Description of FileCredentials</h2>
 
* This script will update the description of a `FileCredentials` stored in the Jenkins Credential Store

```groovy
import org.jenkinsci.plugins.plaincredentials.impl.FileCredentialsImpl
import com.cloudbees.plugins.credentials.SecretBytes
def updateSecretFileDescription = { cred_id, file_name ->
    // def result = false
    def newDescription = 'CREDS_FILE for rotation'
    def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
        com.cloudbees.plugins.credentials.common.StandardCredentials.class,
        Jenkins.instance
    )
    def credentials_store = Jenkins.instance.getExtensionList(
      'com.cloudbees.plugins.credentials.SystemCredentialsProvider'
    )[0].getStore()
    def c = creds.findResult { it.id == cred_id ? it : null }

    if (c) {
        /*def result = credentials_store.updateCredentials(
            com.cloudbees.plugins.credentials.domains.Domain.global(),
            c,
            new FileCredentialsImpl(c.scope, c.id, newDescription, c.fileName, c.secretBytes)
          )
        println "${result}" 
        println "found credential ${c.id}"*/
        try {
          def result = credentials_store.updateCredentials(
            com.cloudbees.plugins.credentials.domains.Domain.global(),
            c,
            new FileCredentialsImpl(c.scope, c.id, newDescription, c.fileName, c.secretBytes)
          )
        println "${result}" 
        println "updated credential ${c.id}'s description"
        } catch (Exception e) {
          result = false
          println "${result}"
          println "failed to update credential ${c.id}'s description"
        }
    }
   // return result
}

updateSecretFileDescription('${credId}', '${fileName}')
```

<h2>Updating Content of FileCredentials</h2>
 
* This script will update the content of a `FileCredentials` stored in the Jenkins Credential Store

```groovy
import org.jenkinsci.plugins.plaincredentials.impl.FileCredentialsImpl
import com.cloudbees.plugins.credentials.SecretBytes
def password_expired = false
def updateSecretContent = { cred_id, file_name ->
    if (password_expired == true) {
      def String file_content = readFile(file_name)
      def String content = file_content.bytes.encodeBase64().toString()
      def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
          com.cloudbees.plugins.credentials.common.StandardCredentials.class,
          Jenkins.instance
      )
      def credentials_store = Jenkins.instance.getExtensionList(
        'com.cloudbees.plugins.credentials.SystemCredentialsProvider'
      )[0].getStore()
      def c = creds.findResult { it.id == cred_id ? it : null }
      if (c) {
          try {
            def result = credentials_store.updateCredentials(
              com.cloudbees.plugins.credentials.domains.Domain.global(),
              c,
              new FileCredentialsImpl(c.scope, c.id, c.description, c.fileName, SecretBytes.fromString(content))
            )
          println "updated credential ${c.id}'s content"
          } catch (Exception e) {
            error "failed to update credential ${c.id}'s content"
          }
      }
  } else {
      println "Password for ${cred_id} has not expired, skipping..."
  }
}
updateSecretContent('${credId}', '${fileName}')
```