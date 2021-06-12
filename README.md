#### 修改地方
这是自己项目的配置，实现ldap认证，主要做了如下更改
1. 依赖包
        ```
            // CAS dependencies/modules may be listed here statically...
            implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
            implementation "org.apereo.cas:cas-server-support-ldap:${project.'cas.version'}"   // add  用于ldap认证
            implementation "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"  // add  用于服务注册
        ```
2. 其他的都是配置文件的修改
   - `cas.properties`
        ```
        cas.server.name=https://127.0.0.1:8443
        cas.server.prefix=${cas.server.name}/cas

        logging.config=file:/etc/cas/config/log4j2.xml

        cas.service-registry.core.init-from-json=true
        cas.serviceRegistry.json.location=file:/etc/cas/services

        cas.authn.oauth.grants.resourceOwner.requireServiceHeader=true
        cas.authn.oauth.userProfileViewType=NESTED

        cas.authn.policy.requiredHandlerAuthenticationPolicyEnabled=false

        cas.authn.attributeRepository.stub.attributes.email=casuser@example.org
        #REST API JSON
        cas.rest.attributeName=email
        cas.rest.attributeValue=.+example.*


        # 开启debug 模式
        logging.level.org.apereo.cas=DEBUG

        # don't allow login of built-in users
        cas.authn.accept.enabled=false

        cas.tgc.crypto.encryption.key=WHlJ0RAQFlWO2xIqBk6DWJikhztVkLdc4ZH9FAEfgCs
        cas.tgc.crypto.signing.key=8aPt64LTmOSjIK-5_TafK0XmRmp_NgCv67tXKs73pKhDGOtt6zygWlrYc-AC0LEDV7-x32yDM44gDDoZ7QhPwg
        cas.webflow.crypto.signing.key=2GsxWaSAG5YBYNEbROHPgkZxgNR3Fu7BQQiifj1VrI-4ZtwEb5YnIHxaopgpc_Gs4E-ywpM_7jqx9s7gY6PLhw
        cas.webflow.crypto.encryption.key=i22Sc5RvsXRRHqAiaLjjCA

        ldap-url=ldap://xxxx:389
        ldap-dnformat=cn=%s,cn=users,dc=xxxx,dc=com
        ldap-base-dn=CN=Users,DC=xxxx,DC=com
        # 这里要参考自己的ad域接口
        ldap-bind-dn=CN=xxxx,OU=xxx,OU=xxxx,DC=xxxx,DC=com
        # password
        ldap-bind-credential=xxxxxx    

        cas.authn.ldap[0].password-policy.groovy.location=
        cas.authn.ldap[0].principal-transformation.groovy.location=
        cas.authn.ldap[0].base-dn=${ldap-base-dn}
        cas.authn.ldap[0].bind-dn=${ldap-bind-dn}
        cas.authn.ldap[0].bind-credential=${ldap-bind-credential}
        cas.authn.ldap[0].dn-format=${ldap-dnformat}
        cas.authn.ldap[0].ldap-url=${ldap-url}
        cas.authn.ldap[0].search-filter=(cn={user})
        cas.authn.ldap[0].type=DIRECT
        cas.authn.ldap[0].password-encoder.encoding-algorithm=
        cas.authn.ldap[0].password-encoder.type=NONE

        ```

CAS Overlay Template [![Build Status](https://travis-ci.org/apereo/cas-overlay-template.svg?branch=master)](https://travis-ci.org/apereo/cas-overlay-template)
=======================

Generic CAS WAR overlay to exercise the latest versions of CAS. This overlay could be freely used as a starting template for local CAS war overlays.

# Versions

- CAS `6.4.x`
- JDK `11`

# Overview

To build the project, use:

```bash
# Use --refresh-dependencies to force-update SNAPSHOT versions
./gradlew[.bat] clean build
```

To see what commands are available to the build script, run:

```bash
./gradlew[.bat] tasks
```

To launch into the CAS command-line shell:

```bash
./gradlew[.bat] downloadShell runShell
```

To fetch and overlay a CAS resource or view, use:

```bash
./gradlew[.bat] getResource -PresourceName=[resource-name]
```

To list all available CAS views and templates:

```bash
./gradlew[.bat] listTemplateViews
```

To unzip and explode the CAS web application file and the internal resources jar:

```bash
./gradlew[.bat] explodeWar
```

# Configuration

- The `etc` directory contains the configuration files and directories that need to be copied to `/etc/cas/config`.

```bash
./gradlew[.bat] copyCasConfiguration
```

- The specifics of the build are controlled using the `gradle.properties` file.

## Adding Modules

CAS modules may be specified under the `dependencies` block of the [Gradle build script](build.gradle):

```gradle
dependencies {
    implmentation "org.apereo.cas:cas-server-some-module:${project.casVersion}"
    ...
}
```

To collect the list of all project modules and dependencies:

```bash
./gradlew[.bat] allDependencies
```

You could also add modules and dependencies dynamically on the fly using the `casModules` project property. For example, to include support for OpenID Connect and Duo Security, you could invoke the build using `-PcasModules=oidc,duo` and have it auto-include modules that provide requested functionality. Needless, to say, you will need to know the module name beforehand.

### Clear Gradle Cache

If you need to, on Linux/Unix systems, you can delete all the existing artifacts (artifacts and metadata) Gradle has downloaded using:

```bash
# Only do this when absolutely necessary
rm -rf $HOME/.gradle/caches/
```

Same strategy applies to Windows too, provided you switch `$HOME` to its equivalent in the above command.

# Deployment

- Create a keystore file `thekeystore` under `/etc/cas`. Use the password `changeit` for both the keystore and the key/certificate entries. This can either be done using the JDK's `keytool` utility or via the following command:

```bash
./gradlew[.bat] createKeystore
```

- Ensure the keystore is loaded up with keys and certificates of the server.

On a successful deployment via the following methods, CAS will be available at:

* `https://cas.server.name:8443/cas`

## Executable WAR

Run the CAS web application as an executable WAR:

```bash
./gradlew[.bat] run
```

Debug the CAS web application as an executable WAR:

```bash
./gradlew[.bat] debug
```

Run the CAS web application as a *standalone* executable WAR:

```bash
./gradlew[.bat] clean executable
```

## External

Deploy the binary web application file `cas.war` after a successful build to a servlet container of choice.

## Docker

The following strategies outline how to build and deploy CAS Docker images.

### Jib

The overlay embraces the [Jib Gradle Plugin](https://github.com/GoogleContainerTools/jib) to provide easy-to-use out-of-the-box tooling for building CAS docker images. Jib is an open-source Java containerizer from Google that lets Java developers build containers using the tools they know. It is a container image builder that handles all the steps of packaging your application into a container image. It does not require you to write a Dockerfile or have Docker installed, and it is directly integrated into the overlay.

```bash
./gradlew build jibDockerBuild
```

### Dockerfile

You can also use the native Docker tooling and the provided `Dockerfile` to build and run CAS.

```bash
chmod +x *.sh
./docker-build.sh
./docker-run.sh
```
