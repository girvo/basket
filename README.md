> Because *&#9834; bash is all you need &#9835;*

## What is it?

It's a <em><strong>bas</strong>h thoole <strong>ket</strong></em>, or *bash toolkit* if you're sober. Yeah, we tried. It's also a basket of bash functions so that kinda works.

Basket is like a basket because you can dump little scripts into it with no overhead other than storage. When sourced, only the main (small) basket of bash helper methods is loaded - special-purpose modules are brought in on demand and have a simple dependency management system. (So if basket ever gets too big, you can just blindly delete stuff you don't need out of the `lib` folder).

Basket is pure bash. Basket is for provisioning and bootstrapping as well as everyday use. Basket is a place to dump shell commands where they may one day be useful. Basket is a set of low-level wrappers for writing more robust and portable shell scripts. It is a reference bible for bash techniques. It is all of these things.



## What isn't it?

Basket is not a full deployment solution, cluster management tool or orchestration system. It does not have rollback capabilities or snapshotting or cloud provider integration or anything fancy like that.

It does work very well with these tools, however - we have used it in combination with Vagrant, Terraform, Packer, Docker, CloudFormation and OpsWorks with high degrees of success. In each of these situations one simply plugs basket in to whatever bootstrap action the framework provides and away you go.



## Module System

The tools suite requires that `basket` itself be sourced, but everything else is optional. To load other modules, you use the helper method `basket__include` in your client application scripts to bring them in. Modules are responsible for resolving their own dependencies internally - they give you a simple dependency management solution coupled with reduction in script loading times.

The argument to `basket__include` is simply a path from the `lib/` folder downwards to load into the scripting environment. For example, to load the module file at `lib/firewall/iptables`, simply write `basket__include firewall/iptables`.

If you are going to be doing something all in bash, it's going to get messy if you don't have some conventions. For basket, these conventions are:

- The `lib` folder structure. We try to group things at the top level by software *'type'* - language environments, webservers, relational and document-oriented database engines, caching systems and so on. The library filenames correspond to the thing you're managing, and since most of basket is about provisioning most of these libraries are helpers for installing software packages. Hopefully this convention holds up for a while but we'll see how it pans out. Any breaking changes will result in a major version update.
- The helper method name format. To try to find a balance between separation of basket commands from normal operation of the system and verbosity of those commands, libraries in the lib folder use the same 'double-underscore' format as basket itself. The starting part of the method name should be the name of the package, for example `apache__install`. The remainder is up to the implementor, but we usually mandate at least an `__install` command.



## Configuration behaviour

Since it's a dual-purpose library, basket checks some environment variables when loaded in order to configure itself.

- `BASKET__INTERACTIVE_MODE` (*0*) &mdash; Toggle interactive mode in scripts. Interactive mode means prompts and console spam. You will want to enable this flag if sourcing basket into your bash profile for everyday use.
- `BASKET__PAUSE_AT_HEADERS` (*0*) &mdash; When interactive mode is on, enabling this option asks for user input every time a pretty basket header string is rendered (`basket__header {message}`).

Basket only requires a temporary folder for downloading files somewhere and does otherwise not interact with the filesystem internally. The only exceptions are a few choices on the final destination directories of installed programs. Configuration of these paths can be managed by way of some environment variables, which will use standard defaults unless otherwise set:

- `BASKET__PATH_TEMP` (*~/Downloads/*) &mdash; Temporary folder for files used in scripts. **This folder is intentionally not cleaned up** - the idea is that temporary installation media and external dependencies are retained on the system for referencing later or copying to backup media.
- `BASKET__PATH_MANUAL_INSTALL_DIR` (*/opt/*) &mdash; Installation directory for manually compiled or downloaded applications not managed by the system package manager.
- `BASKET__PATH_BIN` (*/usr/bin/*) &mdash; Sudo-usable system binary path, targeted by some installation targets. You should probably not change this value.
- `BASKET__PATH_USRBIN` (*/usr/local/bin/*) &mdash; Same as above, but for standard users. You should probably not change this value.



## Design philosophy

- If I am trying to create an artifact for a system, I should be able to recreate it easily on even the meagrest of hardware. I should not have to install a massive application stack just to run a deployment.
- Central orchestration systems are complex in their functionality and difficult to debug. I should not have to learn another language purely for the purposes of administering or maintaining software. I should not have to invest significant time in learning a new framework. I should not have to be locked into a service platform or a support contract.
- Servers are throwaway, so deployment need only involve build automation and snapshotting.
- Build commands and shell helpers should be easily usable in your everyday work and not only applicable in particular circumstances.
- Building servers should expose you to the operating system so that over time you become more familiar with its behaviour.



### Todo

- document a compatibility matrix, finish off complementary debian-only and redhat-only helpers
- reduce terminal noise more
- self-documenting library code (`basket__show_packages` / `basket__query_package`?)
- improve testing framework
- more helpers:
    - service manager:
        - autodetect systemv, upstart etc and wrap up daemon management actions
        - start / stop / restart services
        - install / uninstall startup script
    - manual package installation:
        - retrieval of system architecture, ubuntu release name etc for use with install commands
        - verifying key fingerprints and file signatures downloaded files
    - ensuring of file ownerships and permissions
    - create & sync to directory, incremental backups
    - management of scripting hooks:
        - reboot and shutdown
        - power management (sleep, hibernate, resume etc)
        - X login
        - USB device attached
    - install / uninstall desktop application (gnome3, unity)
- cleanup:
    - put nodejs manual installation to the correct location
