<!--- Provide a general summary of your changes in the Title above -->

[linuxserverurl]: https://linuxserver.io
[![linuxserver.io](https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/linuxserver_medium.png)][linuxserverurl]


<!--- Before submitting a pull request please check the following -->
* If this is a fix for a typo (in code, documentation, or the README) please file an issue and let us sort it out. We do not need a PR
    * This is not a fix for a typo.

* Ask yourself if this modification is something the whole userbase will benefit from, if this is a specific change for corner case functionality or plugins please look at making a Docker Mod or local script  https://blog.linuxserver.io/2019/09/14/customizing-our-containers/
    * I think this modification is something the whole userbase could benefit from, giving more flexibility inside the container itself without having to copy pasta the entire init script for a docker mod to work.
* That if the PR is addressing an existing issue include, closes #<issue number> , in the body of the PR commit message  
    * It does not address an existing issue, but it does come out of the want to specify a self signed certificate so a reverse proxy can use that to encrypt the traffic between the two applications.
<!-- You have included links to any files / patches etc your PR may be using in the body of the PR commit message -->
<!--- We maintain a changelog of major revisions to the container at the end of readme-vars.yml in the root of this repository, please add your changes there if appropriate -->


<!--- Coding guidelines: -->
<!--- 1. Installed packages in the Dockerfiles should be in alphabetical order -->
<!--- 2. Changes to Dockerfile should be replicated in Dockerfile.armhf and Dockerfile.aarch64 if applicable -->
<!--- 3. Indentation style (tabs vs 4 spaces vs 1 space) should match the rest of the document -->
<!--- 4. Readme is auto generated from readme-vars.yml, make your changes there -->

------------------------------

 - [x] I have read the [contributing](https://github.com/linuxserver/docker-unifi-network-application/blob/main/.github/CONTRIBUTING.md) guideline and understand that I have made the correct modifications

------------------------------

<!--- We welcome all PR’s though this doesn’t guarantee it will be accepted. -->

## Description:
<!--- Describe your changes in detail -->
On the first initialization of the unifi network application via this application, it checks to see if there is a mounted keystore.jks file if there is, the script will proceed to importing that jks into the keystore. Rather then auto generating a unique key that is pretty hard to modify once the container is created if using kubernetes, or a different certificate that wasn't generated.

## Benefits of this PR and context:
<!--- Please explain why we should accept this PR. If this fixes an outstanding bug, please reference the issue # -->
The benefits of this PR are in my opinion allow for no hack arounds to occur to get this to work with a reverse proxy that isn't traefik and encrypt the traffic via tls from the proxy to the controller.
This allows users to bring their own certificates as long as they are signed appropriately and will work with unifi.
An example of this is cert-manager.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: self-signed-svc-cert
spec:
  dnsNames:
    - {your-domain-name}
  secretName: unifi-signed-cert
  commonName: unifi
  issuerRef:
    name: self-signed-ca-issuer
    kind: ClusterIssuer
    group: cert-manager.io
  keystores:
    jks:
      alias: unifi
      create: true
      # This is really just aircontrolenterprise as it has to be.
      passwordSecretRef:
        name: unifi-keystore
        key: password
```

This allows the user to create a self signed certificate that allows them to use this on a reverse proxy application for example. NGINX that is using the gatewapi implementation `BackendTLSPolicy`.

This is largely because there is no way at all to turn off insecure ski verfication for some ingress implementations.
https://docs.linuxserver.io/images/docker-unifi-network-application/#strict-reverse-proxies

This solves this problem altogether.
https://docs.linuxserver.io/images/docker-unifi-network-application/#strict-reverse-proxies

Users can now bring their own certificate and not have to worry about configuring it after the fact.
## How Has This Been Tested?
<!--- Please describe in detail how you tested your changes. -->
<!--- Include details of your testing environment, and the tests you ran to -->
<!--- see how your change affects other areas of the code, etc. -->


## Source / References:
<!--- Please include any forum posts/github links relevant to the PR -->
* https://github.com/linuxserver/docker-unifi-network-application/issues/78
