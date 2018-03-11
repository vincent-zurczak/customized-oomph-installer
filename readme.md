# Customized OOMPH Installer

This project shows how to automatically download and pre-configure an OOMPH installer.  
Preconfiguring includes proxy settings, initial preferences for OOMPH, custom catalog, etc.

> At the beginning, this project was used by an organization with a custom
> OOMPH catalog, private p2 repositories and mirrors of all the Eclipse repositories.
> This example was reworked a little bit to be published here.

The build is made with Maven: `mvn clean package`.  
Built installers are attached as Maven artifacts.

The following actions are performed during the build:

* Download the official installers in the **cache** directory.
* Download SHA-12 signatures of the installers and verify them.
* Extract archives in the **target/eclipse-installer** directory.
* Configure network settings for the installer.
* Configure the installer itself (**eclipse-inst.ini** file).
* Build archives of the modified installers.

> Self-extracting archives (.exe) for Windows are converted into ZIP files.


## Environments and Properties

The POM file defines a set of properties.  
Most of them are intended to be injected in the installer configuration. Since this project
is a copy of the real project, reworked to serve as an example, you can consider these properties
as not relevant (or if you prefer, as examples).

For the record...

* **oomph.update.location**: the location of the p2 repository to download Eclipse packages.  
By default, this is the official Eclipse repository. But one could target mirrors.

* **plugins.location**: the location of the p2 repository than contains custom OOMPH extensions developed by the organization.  
This property is optional.

* **setups.location**: the location of the custom OOMPH catalog.  
**This property is mandatory**. Otherwise, the installer will not show anything.  
Besides, **the value of this property MUST end with `/`.**


## Hack for the Internal Proxy

The original project used profiles.  
One targeted no specific environment: packages were downloaded from Eclipse.org, etc.
Another one was made to be invoked inside the internal environment. To download installers from
Eclipse.org, we had to use the organization's proxy. Since ANT is in charge of the download, we rely
on the **setProxy** ANT task. It works, but one should know it can have side effects on other projects running
in the same JVM.

Alternatives (not retained) were:

* Pass everything in the Maven command.  
`mvn clean install -DproxySet=true -Dhttp.proxyHost=proxy.my-organization -Dhttp.proxyPort=3128`

* Define an environment variable called MAVEN_OPTS (...), with all the proxy settings.
* Define the proxy settings in the **settings.xml** file. In that case, the command would have been `mvn clean install`.


## About Cache Management

Most of the developers are used to run `mvn clean`.  
To prevent undesired deletions of the downloaded artifacts, there are saved separately, under the **cache**
directory. This directory is obviously not committed to the Git repository.

SHA-12 signatures are always downloaded.  
In case of update at Eclipse.org, the cache will be marked as invalid and a manual reset will be requested during the build process.

Eclipse.org does not keep old versions of the installer.  
So, this project always downloads the last version, considered as the best one. These files have a life span
of few months (they do not change every day).
