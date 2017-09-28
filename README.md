# ansible-role-oraclejdk
An Ansible role to install Oracle JDK.

[![Platform](http://img.shields.io/badge/platform-ubuntu-dd4814.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-debian-a80030.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-redhat-cc0000.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-centos-932279.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-fedora-333fff.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-suse-008000.svg?style=flat)](#)
[![Platform](http://img.shields.io/badge/platform-opensuse-49BD31.svg?style=flat)](#)

[![OracleJDK](http://img.shields.io/badge/oracle%20jdk-%20%208%20%20-cc0000.svg?style=flat)](#)
[![OracleJDK](http://img.shields.io/badge/oracle%20jdk-%20%209%20%20-cc0000.svg?style=flat)](#)

## Summary

This Ansible role installs Oracle JDK and Java Cryptography Extension (optional) on the target host. OpenJDK is not supported. It has been tested with Oracle JDK 8u144 and 9u181 on the following Linux distributions.

* Ubuntu 16.04
* Debian 9
* RHEL 7.4
* CentOS 7.3
* Fedora 26
* SLE 12 SP2
* openSUSE 42.2

## Dependencies

Ansible >= 2.x

In addition this role requires the following packages to be installed on the target host.

* `tar` - Always required to extract the JDK tar.gz archive
* `unzip` - Only required if JCE should be installed (JDK8 or lower only)

## Installation

To use this role it must first get downloaded from [Ansible Galaxy](https://galaxy.ansible.com). To install the role execute the command below on your Ansible controle machine. For more information please refer to the [official documentation](https://galaxy.ansible.com/intro#download).

`ansible-galaxy install frieder.oraclejdk`

## Role Variables

All role variables are defined in `defaults/main.yml`. One can overwrite these values in the playbook (see examples at the bottom). By default the role will try to install Oracle JDK 8 Update 144 (the latest version as of Sep 24).

| Variable | Default | Description |
|--:|:-:|:--|
| oraclejdk_license_accept | false | Must be set to true or the play will fail. |
| oraclejdk_cleanup | true | Remove all temp files/folders (both local and remote) after installation. When set to true this will result in "changed" plays. |
| oraclejdk_install_prereq   | false | If set to true it will install packages (see requirements) that are required for this role to succeed.|
| oraclejdk_force_install | false | By default the role will check if the JDK installation folder already exists and skips the extraction of the archive if it does exist. Setting this value to true will force the role to extract the archive even when the folder already exists. Please be aware that this may overwrite any previously installed JCE policy files (JDK8 only). |
| oraclejdk_dl_dir | /tmp/oraclejdk | The folder (both local and remote) to which the archives get downloaded/copied/extracted. |
| oraclejdk_base_root | /opt/oraclejdk | The folder in which the JDK archives get extracted. All JDK folders will be in here. |
| oraclejdk_profile_file | /etc/profile.d/java.sh | The file in which the role will set the JAVA_HOME and PATH export. |
| oraclejdk_cookie | Cookie:oraclelicense=accept-securebackup-cookie | The cookie required for automated downloads from oracle.com. License check is done with a different variable. |
| oraclejdk_name | jdk1.8.0_144 | The name of the folder within the JDK archive. The role also uses it as the JDK folder name inside the {{ oraclejdk_base_root }} base directory. |
| oraclejdk_url | http://download.oracle.com/... | The URL of th JDK archive at oracle.com. |
| oraclejdk_checksum | sha256:... | The SHA256 checksum of the JDK archive. This is used to verify that the downloaded file is valid. To disable this check just provide an empty checksum value (`oraclejdk_checksum: ''`). |
| oraclejdk_sethome | true | When set to true it will update the global variable JAVA_HOME to point to the installation directory of the JDK and add the binaries to the PATH variable. Also have a look at the `oraclejdk_profile_file` variable. |
| oraclejdk_alternative_upd | true | When set to true it will set the alternative for java (`update-alternatives --config java`) to the current JDK. In addition it will also update the alternatives for `jar, javac, jcmd, jconsole, jmap, jps, jstack, jstat, jstatd`.|
| oraclejdk_alternative_prio | 8144 | The priority used for the `update-alternatives` command. The JDK with the highest priority wins. |
| oraclejdk_jce_install | true | When set to true it will install and add the latest Java Cryptography Extension (JCE) to the JDK. Please note that this is only required for JDK 8 or earlier versions. JDK9 already comes with the latest policy files which can be activated by [setting a property](http://mail.openjdk.java.net/pipermail/security-dev/2016-October/014943.html). |
| oraclejdk_jce_name | UnlimitedJCEPolicyJDK8 | The name of the folder inside the JCE archive. |
| oraclejdk_jce_url | http://download.oracle.com/... | The URL of the JCE archive at oracle.com. |
| oraclejdk_jce_checksum | sha256:... | The SHA256 checksum of the JCE archive. To disable this check just provide an empty checksum value (`oraclejdk_jce_checksum: ''`). |

## Example Playbooks

Following are some examples how to use this role in an Ansible playbook.

**JDK8 with minimal configuration**
```yaml
- hosts: servers
  roles:
     - role: frieder.oraclejdk
       oraclejdk_license_accept: true
```

**JDK8 & JCE8 & Prerequisites & No Cleanup**
```yaml
- hosts: servers
  roles:
    - role: frieder.oraclejdk
      oraclejdk_license_accept: true
      oraclejdk_jce_install:    true
      oraclejdk_install_prereq: true
      oraclejdk_cleanup:        false
```

**JDK9 & No Cleanup**
```yaml
- hosts: servers
  roles:
    - role: frieder.oraclejdk
      oraclejdk_license_accept:   true
      oraclejdk_name:             jdk-9
      oraclejdk_url:              'http://download.oracle.com/otn-pub/java/jdk/9+181/jdk-9_linux-x64_bin.tar.gz'
      oraclejdk_checksum:         'sha256:1c6d783a54fcc0673ed1f8c5e8650b1d8977ca3e856a03fba0090198e0f16f6d'
      oraclejdk_alternative_prio: 9181
```