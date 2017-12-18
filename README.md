# SELinux Summary

This summary has been adapted from the [Gentoo SELinux Tutorial](https://wiki.gentoo.org/wiki/SELinux/Tutorials).

**[Seriously, stop disabling SELinux.](https://stopdisablingselinux.com/)**

SELinux is an amazing technology and it's not that hard at all to understand
and to start using it, so why disable it? Most people see it as an additional
burden to whatever they are doing and don't really understand that can be
easily tuned after having understood some of its functionalities, so they
just disable it.

This summary is here to give a quick and clear introduction to SELinux with the
hope that people, after having read it, will be more comfortable keeping it
enabled and enforcing.

## The security context of a process

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/The_security_context_of_a_process)

- A security context is a specific label assigned to a process, which informs
  SELinux about the rights and privileges that are allowed to be granted on the
  process.
- A security context cannot change at the discretion of the process, but is
  instead governed by the SELinux policy itself.
- Several processes can be assigned the same label.
- A process is always assigned one and exactly one label.

## How SELinux controls file and directory accesses

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/How_SELinux_controls_file_and_directory_accesses)

- In SELinux vocabulary, we say that:
    - the context of the process that is acting upon something is called the
      *domain*;
    - the context of the resource on which the process is acting is called the
      *type*;
    - the type or class of the resource is called the *class*;
    - the permission or permissions that are allowed given the domain, type and
      class are the *permissions*.
- It uses contexts for processes (domains) and contexts for files (types) as
  part of its internal language for allowing access.
- The suffix `_t` is a naming convention to tell that the given label is a type
  (or domain when it is for a running process), like in
  `system_u:system_r:auditd_t` the part `auditd_t` is the domain name.
- It uses the `allow <domain> <type>:<class> { <permissions> };` syntax for
  this access. This syntax is called "type enforcement";
- It stores the security context (or SELinux context) of a file or directory as
  an extended attribute of this file.

## Where to find SELinux permission denial details

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Where_to_find_SELinux_permission_denial_details)

- Denials are logged in the `avc.log` (no audit daemon running) or `audit.log`
  (audit daemon running) log files inside `/var/log`.
- Denials might be obscured through dontaudit statements, which you can disable
  using `semodule -DB` and re-enable through `semodule -B`.
- The denial logging gives you great detail about who (process information,
  including security context) is trying to do what (permission) against
  something (target information, including security context).
- Useful tools to work with denials are: `seinfo`, `semodule`, `sesearch`,
  `ausearch` and `sealert` (where installed).

## Controlling file contexts yourself

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Controlling_file_contexts_yourself)

- The context of a file is one of the most important parts of a SELinux secured
  system. Wrong contexts are the most common source of SELinux-related denials
  and permission problems.
- SELinux uses file context definitions using regular expressions to set the
  context definition of a file or directory.
- To query SELinux for existing file context definitions use the command
  `semanage fcontext -l`
- The `restorecon` utility checks the contexts of the files and match those
  against the contexts definitions stored in the SELinux context and then apply
  the correct context.

## How does a process get into a certain context

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/How_does_a_process_get_into_a_certain_context)

- SELinux by default inherits contexts, be it from processes (on fork) or
  parent files/directories.
- This inheritance is always the case, even if the SELinux management utilities
  (but not SELinux policy rules) say otherwise. The right context can be later
  applied using `restorecon`.
- Contexts of processes can change on execute of a command from that process'
  context, but only under the conditions that:
    - the target file context is executable for the source domain;
    - the target file context is marked as an entrypoint for the target domain;
    - the source domain is allowed to transition to the target domain;

## Using SELinux booleans

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Using_SELinux_booleans)

 - A SELinux boolean is a single string that changes how SELinux reacts to
   dynamically update the run-time policy.
 - With `semanage boolean -l` you can get the description of a boolean.
 - Changing SELinux booleans can be done through `setsebool`, where you add the
   desired state of the boolean such as on or off, or `togglesebool` which
   flips the current value of a boolean.
 - The values of these booleans can be persisted across reboots.
 - You can use `sesearch` to display the consequences of a boolean or to see if
   a boolean is available to allow certain statements.

## Working with customizable types

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Working_with_customizable_types)

- Customizable types exist for files and resources that have no fixed location
  on a file system.
- Customizable types are type contexts which can be assigned on files and other
  resources, and where this context is not reset during a standard relabel
  operation.
- They are most frequently used on files where the path of the file itself is
  not really fixed on Linux system.
- The list of current customizable types can be found in
  `/etc/selinux/*/contexts/customizable_types`.
- The context of files with a customizable type context can be reset if you
  use the force (`restorecon -F`) option during relabel operations.

## Permissive versus enforcing

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Permissive_versus_enforcing)

- SELinux has two "modes" of operation: permissive and enforcing.
- In permissive mode SELinux does not enforce its policy but only logs what
  it would have blocked (or granted).
- To get information about the current state, you can use `getenforce` or
  `sestatus`. To switch from enforcing to permissive and back, you can use the
  `setenforce` command.
- The default value when the system boots is defined in the
  `/etc/selinux/config` file. Another method to define how to boot is using the
  `enforcing=` kernel boot parameter.
- Applications that are SELinux-aware might still behave differently with
  permissive mode than when SELinux is completely disabled.
- Specific types can be marked as permissive while the rest of the system is
  in enforcing mode.
- Completely disabling SELinux has consequences on the file contexts so an
  entire system relabeling is needed afterwards.

## What is this unconfined thingie

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/What_is_this_unconfined_thingie_and_tell_me_about_attributes)

- Unconfined domains run virtually without SELinux protection.
- Unconfined domains are meant to have users run with little SELinux
  interference, whereas the network-facing daemons still run in confined,
  SELinux-protected domains.
- Policy writers can mark any domain as being an unconfined domain and they can
  be checked using `seinfo -aunconfined_domain_type -x`.
- Unconfined domains are only supported when the unconfined SELinux module is
  loaded and can be checked using `seinfo -tunconfined_t`.
- SELinux has support for type attributes, which can group multiple different
  types and assign privileges to the entire group of types. You can query the
  type attributes currently in the policy using `seinfo -a<domain> -x`.

## How is the policy provided and loaded?

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/How_is_the_policy_provided_and_loaded)

- SELinux uses a modular design for its policies, similar as how the Linux
  kernel uses modules.
- A policy store is used to keep track of loaded policy modules and settings,
  and is governed through the `SELINUXTYPE` setting in `/etc/selinux/config`.
- The `semodule` tool is used to load, unload, ... SELinux policies.

## The purpose of SELinux roles

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/The_purpose_of_SELinux_roles)

- Roles dictate which domains the user (or session) is allowed to go in. Domain
  transitions still govern which domains the user launches in next.
- The naming convention states that the role names should end with `_r`, like
  in `user_u:user_r:user_t` the part `user_r` is the role name.
- With the `seinfo -r<role_name> -x`, you can list what domains are allowed
  for a particular role. With `semanage user -l` you can see what roles the
  SELinux user is allowed to "be" in.
- Changing roles is done using `newrole`.
- The SELinux user definitions declare what the supported roles are that a
  user can "go" to.

## Defining SELinux users

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Defining_SELinux_users)

- SELinux users define what roles a user is allowed to go to.
- Linux accounts are mapped to SELinux users (but they are not the same) in a
  many-to-few relationship (usually).
- The roles that are assigned to a particular SELinux user can be seen from
  `semanage user -l`. The mapping of Linux accounts to SELinux users using can
  be displayed using `semanage login -l`.
- The catch-all account `__default__` ensures that all newly created Linux
  account users are mapped to a specific SELinux user.
- SELinux user information rarely changes in a session (mostly only when
  services are launched where they become `system_u`).
- The `semanage login` and `semanage user` set of commands are also used to
  manipulate these settings.

## Linux services and the system_u SELinux user

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Linux_services_and_the_system_u_SELinux_user)

- The `system_u` SELinux user is meant to be used for running system services.
  In general, most daemons will be running as the `system_u` SELinux user in
  the `system_r` role. Most access controls are situated around the domain part
  of the context.
- This is because SELinux user and SELinux role constraints are to govern the
  activities that local users can do, as local user access is often considered
  a risk.
- Calling linux service scripts (init scripts) causes a change in SELinux
  user, role and domain, partially due to policy and partially due to the
  `run_init` command.
- Some service scripts are not labelled the regular `initrc_exec_t` and require
  a different approach for calling them, and might result in the process to
  remain running under the users' SELinux user (but that's okay).
- It is the SELinux policy that governs changes in roles and users.

## Putting constraints on operations

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/Putting_constraints_on_operations)

- Constraints are an integral part of the SELinux policy.
- When something is denied even though there are (type enforcement) rules that
  allow it, chances are very high that a constraint is involved.
- Unlike type enforcement, constraints use the entire context as their rules
  and are more targeting operations rather than domains.
- Constrains don't limit (blacklist) what is allowed, but filter (whitelist)
  what is.
- It is not possible to disable constraint.
- You can ask your system to list the constraints using `seinfo --constrain`.

## SELinux Multi-Level Security

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/SELinux_Multi-Level_Security)

- SELinux supports sensitivity levels (hierarchical) and categories (labels).
- The sensitivity level is the hierarchical part of the multi-level security
  approach. With the sensitivity labels, an MLS-enabled system can mark certain
  content as being of a certain sensitivity and have processes marked as
  supporting a sensitivity.
- The second part of the multi-level security are the categories which can be
  seen as labels assigned to a resource. Unlike sensitivity levels, categories
  do not have any correlation with each other.
- The SELinux policy dictates what is allowed based on the dominance between
  two contexts.
- Distributions usually do not provide a starting set of sensitivities and
  categories.

## SELinux Multi-Category Security

 - MCS is a 'smaller' version of MLS
 - Users (or applications) are responsible for handling the sensitivity (within
   the limits of what is allowed by policy), SELinux does not automatically
   'deduce' the right context

## Managing network port labels

 - SELinux also labels udp/tcp ports as it is one of the many resources it
   governs
 - You can assign existing labels to other ports using semanage port

## Glossary

- **Booleans**: Runtime setting to enable customization of SELinux policy.

- **Domain**: All processes and files are labelled with a type. A type defines
  a domain for processes, and a type for files. Processes are separated from
  each other by running in their own domains, and SELinux policy rules define
  how processes interact with files, as well as how processes interact with
  each other. Access is only allowed if an SELinux policy rule exists that
  specifically allows it.

- **Domain Transitions**: A process in one domain transitions to another domain
  by executing an application that has the entrypoint type for the new domain.
  The entrypoint permission is used in SELinux policy, and controls which
  applications can be used to enter a domain.

- **MLS/MCS**: MLS enforces the Bell-La Padula Mandatory Access Model, and is
  used in Labeled Security Protection Profile (LSPP) environments. An MLS range
  is a pair of levels, written as *lowlevel-highlevel* if the levels differ, or
  *lowlevel* if the levels are identical (`s0-s0` is the same as `s0`). Each
  level is a *sensitivity-category* pair, with categories being optional. If
  there are categories, the level is written as *sensitivity:category-set*. If
  there are no categories, it is written as *sensitivity*. An example of full
  MLS level is `system_u:system_r:sshd_t:s0-s0:c0.c1023`.

- **Policy**: A set of rules that guide the SELinux security engine. It defines
  types for file objects and domains for processes, uses roles to limit the
  domains that can be entered, and has user identities to specify the roles
  that can be attained. A domain is what a type is called when it is applied to
  a process.

- **Role**: The second component of a SELinux security context. SELinux users
  are authorized for roles, and roles are authorized for domains. The role
  serves as an intermediary between domains and SELinux users. The roles that
  can be entered determine which domains can be entered; ultimately, this
  controls which object types can be accessed.

- **Security Context**: Processes and files are labelled with an SELinux
  context that contains additional information, such as an SELinux user, role,
  type, and, optionally, a level. When running SELinux, all of this information
  is used to make access control decisions.

- **Type Enforcement**: Domains and their allowed permissions towards other
  domains and types.

- **Type**: The type defines a domain for processes, and a type for files.
  SELinux policy rules define how types can access each other, whether it be a
  domain accessing a type, or a domain accessing another domain. Access is
  only allowed if a specific SELinux policy rule exists that allows it.

- **User**: The SELinux user is an identity known to the policy that is
  authorized for a specific set of roles, and for a specific MLS/MCS range.
  Each Linux user is mapped to an SELinux user via SELinux policy. It defines
  what roles are allowed for a specific SELinux user.

## Cheat sheet

**ps auxZ**

The option `-Z` shows the security context of the running processes.

**ls -lZ**

The option `-Z` shows the security context of the files and directories.

**id -Z**

The option `-Z` shows the security context of the current user.

**ausearch --interpret --message AVC,USER_AVC --success no**

To display the denials from the logs.

## More resources

- [SELinux For Dummies](https://www.youtube.com/watch?v=q_y30qZ_plQ) (YouTube Video)
- [Security-enhanced Linux for mere mortals](https://www.youtube.com/watch?v=cNoVgDqqJmM&feature=youtu.be) (YouTube Video)
- [Security-Enhanced Linux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/)
- [A collection of notes on SElinux](http://equivocation.org/selinux)
- [HowTos/SELinux - CentOS Wiki](https://wiki.centos.org/HowTos/SELinux)
- [Security Enhanced Linux Reference Policy](http://oss.tresys.com/docs/refpolicy/api/interfaces.html)
- [The SELinux Notebook - 4th Edition](https://selinuxproject.org/page/Category:Notebook)
- [Type Enforcement File](https://danwalsh.livejournal.com/14442.html)
- [SELinux Tutorial](https://hackinglinux.blogspot.co.uk/2007/05/selinux-tutorial.html)

## Credits

[Fabrizio Colonna](mailto:colofabrix@tin.it)