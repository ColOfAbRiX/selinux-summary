# SELinux Summary

This summary is taken from the [Gentoo SELinux Tutorial](https://wiki.gentoo.org/wiki/SELinux/Tutorials) and it's been adapted from it.

## The security context of a process

[Full online page on Gentoo](https://wiki.gentoo.org/wiki/SELinux/Tutorials/The_security_context_of_a_process)

- A process is assigned a security context which, just like with the user under
  which the process runs, helps Linux in identifying what the application
  should and shouldn't be allowed to do.
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

 - Customizable types exist for files and resources that have no fixed location
   on a file system
 - The list of current customizable types can be found in
   /etc/selinux/*/contexts/customizable_types
 - The context of files with a customizable type context can be reset if you
   use the force (-F) option during relabel operations

## Permissive versus enforcing

 - SELinux has two "modes" of operation: permissive and enforcing
 - In permissive mode SELinux does not enforce its policy, but only logs what
   it would have blocked (or granted)
 - Applications that are SELinux-aware might still behave differently with
   permissive mode than when SELinux is completely disabled
 - Specific types can be marked as permissive while the rest of the system is
   in enforcing mode
 - Completely disabling SELinux has consequences on the file contexts so an
   entire system relabeling is needed afterwards

## What is this unconfined thingie

 - Unconfined domains run virtually without SELinux protection
 - Unconfined domains are meant to have users run with little SELinux
   interference, whereas the network-facing daemons still run in confined,
   SELinux-protected domains
 - SELinux has support for type attributes, which can group multiple different
   types and assign privileges to the entire group of types

## How is the policy provided and loaded?

 - SELinux uses a modular design for its policies, similar as how the Linux
   kernel uses modules
 - A policy store is used to keep track of loaded policy modules and settings,
   and is governed through the SELINUXTYPE setting in /etc/selinux/config
 - The semodule tool is used to load, unload, ... SELinux policies

## The purpose of SELinux roles

 - Roles dictate which domains that the user (or session) is allowed to go in
 - Changing roles is done using newrole
 - The SELinux user definitions declare what the supported roles are that a
   user can "go" to

## Defining SELinux users

 - SELinux users define what roles a user is allowed to go to
   Linux accounts are mapped to SELinux users (but they are not the same) in a
   many-to-few relationship (usually)
 - The semanage login and semanage user set of commands are used to manipulate
   these settings
 - SELinux user information rarely changes in a session (mostly only when
   services are launched where they become system_u)

## Linux services and the system_u SELinux user

 - Calling linux service scripts (init scripts) causes a change in SELinux user,
   role and domain, partially due to policy and partially due to the run_init
   command
 - Some service scripts are not labeled the regular initrc_exec_t and require a
   different approach for calling them, and might result in the process to
   remain running under the users' SELinux user (but that's okay)
 - It is the SELinux policy that governs changes in roles and users

## Putting constraints on operations

 - Constraints are an integral part of the SELinux policy
 - When something is denied even though there are (type enforcement) rules that
   allow it, chances are very high that a constraint is involved
 - Constrains don't limit (blacklist) what is allowed, but filter (whitelist)
   what is

## SELinux Multi-Level Security

 - SELinux supports sensitivity levels (hierarchical) and categories (labels)
 - The SELinux policy dictates what is allowed based on the dominance between
   two contexts
 - Distributions usually do not provide a starting set of sensitivities and
   categories

## SELinux Multi-Category Security

 - MCS is a 'smaller' version of MLS
 - Users (or applications) are responsible for handling the sensitivity (within
   the limits of what is allowed by policy), SELinux does not automatically
   'deduce' the right context

## Managing network port labels

 - SELinux also labels udp/tcp ports as it is one of the many resources it
   governs
 - You can assign existing labels to other ports using semanage port

## Credits

[Fabrizio Colonna](mailto:colofabrix@tin.it)