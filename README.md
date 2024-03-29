Autoport Management Schema - RDF Schema for Port Information
============================================================


# Overview

The [Poudriere][pou] build automation tool provides a management
interface for configuration and compilation of _ports_ on the
[FreeBSD][fbsd] operating system.

## Poudriere Logs

[Poudriere][pou] produces a substantial number of _log_ records, such
as may serve to provide information about the configuration state of
invidual _ports_ presented for compilation during any single _build
session_. The _log_ records are generated in JSON format.
Additionally, Poudriere generates HTML files during a _build session_,
such as my serve to present the _log_ records of a _build session_,
for easy viewing with a web browser.

The _log_ data generated by Poudriere may be processed in its JSON
format, towards consructing an RDF _graph_, such as may serve to
illustrate tthe contents of the _log_ data for semantic query and
presentation in a format or RDF.

An RDF _graph_ for Poudriere log data may be presented in a manner
semantically consistent to the data contents of the _log_
data. Juxtaposed to a data model onto an SQL _schema_, an RDF _graph_
for Poudriere log data may be created in a manner aligned with RDF
_classes_, such as to create an arbitrary set of OWL _Indivudal_
instances for each record in the _log_ data. Semantic annotations may
be created in the consequent _graph_, as to illustrate relations among
entries in the corresponding _log_ data.

With an appropriate presentation in a graphical user interface, a
_graph_ of Poudriere_log_ data may be traversed interactively, such as
for purposes of:

* Port Dependency Analysis
* Analysis of Build Failures
* Configuration Dependency Analysis
* Configuration Review

# Format of this Document

This README file is developed in a manner as to be semantically
compatible with the Darwin Information Typing Architecture (DITA). A
set of _concepts_ are defined -- as to provide a manner of reference 
about the Poudriere architecture, with reference to concepts defined
of the FreeBSD _base system_ architecture. These _concept_
definitions, though _non-normative_, may serve to provide a manner of
structure to the development of the Autoport management schema and
management tooling.


# Concepts

## Concept: Build Master

Effectively, a Poudriere _build master_ is a _trie_, comprised of --
at a minimum -- a  _process jail_ and a _ports tree_, with optional
_build set_ identifer

* **Property:** Process Jail
* **Property:** Ports Tree
* **Property [Optional]:** Build Set

## Concept: Process Jail

References

* [Managing Jails with ezjail](https://www.freebsd.org/doc/handbook/jails-ezjail.html)

See Also
* Concept: Sandbox


## Concept: Reference Jail

Functionally, a Poudriere _reference jail_ provides a source tree for
a Poudriere _master jail_ and Poudriere _builder jails_. Poudriere
_reference jails_ may be operated on with the `poudriere jail` 
command.

**Poudriere Commands**

* List reference jails installed on the host: `poudriere jail -l`

* Create a reference jail: `poudriere jail -c`

    * **Jail Property:** _Jail Name_

    * **Jail Property:** _Jail source_ type, or _jail method_. The
      following examples assume a `null` type _jail source_. See also:
      Manual page poudriere(8)

    * **Jail Property:** _FreeBSD version_, denoted in a syntax
      characteristic to the type of _jail source_, e.g.
      `9.0-RELEASE`, or -- for SVN type jails -- an SVN _branch_
      identifier, e.g `stable/9` or `head`

    * **Jail Property:** _FreeBSD architecture_, e.g. `amd64`

    * **Jail Property:** _Jail Pathname_, the pathname of the
      _reference jail_, on the _jail host_

* Update reference jail `poudriere jail -u`


**Notes**

If a _build host_ is to be configured such that Poudriere, itself,
will run within a _process jail_ on the _build host_, each _reference 
jail_ for Poudriere may be created within a containing _process
jail_. The containing _process jail_ may be managed with `ezjail`

## Concept: Master Jail

Poudriere initializes a _master jail_ for purpose of build
automation. To each _master jail_, one or more _builder jails_ may be
initialized for actual port building.

**Poudriere Commands**
* `poudriere jail -s`
* `poudriere jail -k`
* `poudriere queue`
    * See also: `poudriered`

**[Poudriere Hooks][pou_hook]**
* `jail` / `mount`
* `jail` / `start`
* `jail` / `stop`

**Notes**

* The _reference jail_ for a _build master_ set is installed at a
  pathname `/usr/local/poudriere/data/.m/${BUILDMASTER_NAME}/ref/`

* Some ports may require that a populated `/usr/src` is available
  within each _builder jail_ -- for example, ports in the VirualBox
  OSE distribution on FreeBSD. In an example of creating a _null_
  type  Poudriere _master jail_, the contents of the host's `/usr/src`
  diredtory may be copied  -- as with `cp -a` --  to the `usr/src`
  directory of the _master jail_ installation directory, as at the
  time when the _null_ type _master jail_ is first created. Thus, the
  source code can be ensured to be synchronized with the _base system_
  installed to the installation directory of the _null_ type _master
  jail_
  
* **Macine-specific Optimizations**
    * Optimizations for all builds can be applied in `/etc/make.conf`

    * **A host's `/etc/make.conf` can file be insalled into the Poudriere
      _Master Jail_**, as towards applying such optimizations within
      individual _builder jails_
      
    * Optimizations specifically for building the FreeBSD _kernel_ and
    _base system_ can be applied in `/etc/src.conf`
    
    * For machine-specific optimizations, the makefile parameters,
      `CPUTYPE` and `MACHINE_CPU` may be of some particular note. Of
      course, for a multi-machine installation, those values may be
      selected as to include the "Interesection set" of supported
      processor features, across the individual computers of the
      multi-machine installation.
    
    * See also: Reference resources about chracteristics of individual
      microprocessor architectures, e.g.
        * [List of Intel Core i3 microprocessors](https://en.wikipedia.org/wiki/List_of_Intel_Core_i3_microprocessors) 
        * [List of Intel Core i7 microprocessors](https://en.wikipedia.org/wiki/List_of_Intel_Core_i7_microprocessors)
    

## Concept: Builder Jail

**Poudriere Configuration Options**
* `PARALLEL_JOBS`
* `PREPARE_PARALLEL_JOBS`

**[Poudriere Hooks][pou_hook]**
* `builder` / `start`
* `builder` / `stop`

**Notes**

See also, pathname `/usr/local/poudriere/data/.m/${BUILDMASTER_NAME}/[0-9]*/`

## Concept: Bulk Builds

A Poudriere _bulk build_ may be initilized with the Poudriere `bulk` command.

**[Poudriere Hooks][pou_hook]**
* `bulk` / `start`
* `bulk` / `done`

## Concept: Port Builds

**Port Build States and Poudriere Hooks**

* built - `pkgbuild` / `success`

    * **Ed. Note:** Once a port has been successfully built, the port
      should then be available for installation from within the
      pathname, `/usr/local/poudriere/data/packages/${BUILDMASTER_NAME}`. 
      That pathname may be published -- as when published to _trusted
      peer_ hosts on a LAN -- may be published via the Network
      Filesystem (NFS). Alternately, the pathname may be made
      available as via an HTTP or FTP service. If the pathname is
      published as an NFS share, it may be advisable to unmount any
      peers to the share when bulk builds are proceeding.

   * **Ed. Note:** Due to the _restrictive licensing_ terms
     established of some individual ports, and as to the liability of
     the individual institution managing a _package repository_,
     redistribution of packages compiled of those _restritcted ports_
     may be effectively limited to the network of the individual
     institution, or otherwise as to comply with the licensing terms
     of each individual _restricted port_. See also: Pourdiere
     configuration option, `NO_RESTRICTED`


* failed - `pkgbuild` / `failed`

    * **Ed. Note:** A port may _fail_ to build, as due to any
      arbitrary number of possible _causes of failure_. In the event
      of a _failed build_, as within any single _bulk build_, the
      Poudriere logs for the individual _bulk build_ may be consulted
      as to review the _build log_ of the indiviual _port_, under the
      specific _bulk build_.

* ignored - `pkgbuild` / `ignored`

    * **Ed. Note:** A port build may be _ignored_ as when a
      configuration option effectively prevents the port from being
      compiled, for instance as when the `CPU_CLIP` option is enabled
      in the `audio/libsamplerate` port
      
* skipped - `pkgbuild` / `skipped`
      
    * **Ed. Note:**  A port `B` may be _skipped_ as when a
      port `A` in the _build depends_ of port `B` fails to build --
      such as could be due to a _build failure_ in the port `A`, or if
      the port `A` is itself either _skipped_ or _ignored_.

# Notifying of Build Failures

**Ed. Note:** ... 'Push' notification ... messaging ... Jabber ... Kannel ...

# Traversing Build Failures

**Ed. Note:** ... log presentation ... log review ...

# Traversing Host File Provenance Structures

**Ed. Note:** This concept represents the primary thesis developed of
the Autoport management schema. Towards a manner of a short synopsis
of the thesis: A concept of _provenance_ may be defined, as with
regards to _issue tracking_, if not moreover _forensic analysis_ of
files installed on an arbitrary host machine.

For any files installed via a software distribution method such as
with FreeBSD `pkgng`, a file -- whether modified locally or applied as
installed -- may be _source traced_ to the _software package_ from
which the file was installed. The individual _software package_ may
furthermore be _source traced_ to the _software repository_ from which
the _software package_ was retrieved. Subsequently -- assuming that a
complete, trusted _metadata graph_ would be available for any forensic
analysis onto the respective _software distribution service_ -- the
_port source_ may be determined as from which the _software package_
was produced. If the _software package_ was produced within a
Poudriere _bulk build_, then any configuration options that would have
been enabled for the _software package_, at time of _bulk build_
may be available as denoted in the Poudriere _log data_ for the
individual _bulk build_. For purpose of _issue tracking_, the _source
trace_ traversal might then be effectively complete, as in instances
in which an issue may be traced to an individual configuration
option.

In other instances -- assuming a complete, trusted metadata graph --
the _ports repository_ may be located, as from which the _port source_
was retrieved. If any _patches_ were applied onto the sources
initially retrieved from the _ports repository_ -- as may be in
application of _patch files_ available of a single _patch repository_
-- then the repsective _patch repository_ may be located, furthermore.
Of course, the _distfiles repository_ may need to be located,
likewise, as in instances of _packages_ compiled from FreeBSD 
_ports_.

Once the complete set of _source files_ have been located for any
single _package_ produced of a single _port_, then a further analysis
may be conducted as onto the _source code_ of the _port_. Likewise,
any _build dependcies_ or _runtime dependencies_ may be analyzed, so
far as may be relevant for _issue tracking_ or other _forensic
analysis_.

As a manner of an analytical process, the technical traversal of a
_port source_ may not necessarily be complete, at that time. Considering
each of the _distfiles repository_, the _patches repository_ and the
_ports repository_ as, in each repository, representing an
_intermediate software distribution service_, it may be assumed that
any files published of the respective _software repository_ would have
been transmitted from -- in each -- a previous _origin_, as _viz a
viz_ an authorized _principal_ on a  network as would have been
accessible to the respective _software repository_. If that
_next origin_ _principal_ is represented of a _software change and
configuration management_ (SCCM) service, then the _source trace_ 
traversal may proceed onto the _log records_ of the respective SCCM
service. If the file was tranmistted individually, as without an SCCM
service, then if it may be possible to more immediately determine
which exact principal  had transmitted the file. 

Assuming that a complete and trusted _metadata graph_ may be
constructed about any single file, it may be possible to 
_source trace_ a file completely to its origins.

As in regards to applications of _source trace_ traversal in a context
of_issue tracking_ -- as when an _issue  tracking_ issue may be traced
indiviually to a _program errror_ -- it may be helpful if the _issue
tracking_ expert may traverse the origins of the compiled binary file
producing the _program error_, as in order to determine any possible
causes of the _program error_.

As in regards to forensic anlalsys, as in an event of a discovery of
an object of _malicious software code_, the _source trace_ may then
serve as to furthermore comprise a study as in regards to
_cybersecurity practices_, on any network potetntially affected of the
object of _malicious software code_.

The procedures of a _source trace_ analysis must naturally depend, to
a substantial extent, on the integrity of any _log data_ as would be
available to the principal executors of the _source trace_. As to
codify a sense of _inetegrity_ as such, ,this thesis adopts a
concept of _trust_, in that manner -- albeit, in more of a rhetorical
than technical manner -- as towards defining a discrete sense of
_trust_ for representing a manner of measurable _data integrity_, such
as would be of relevance for purposes of forensic log data anslysis,
whether in event of simple _issue tracking_ or in even of the
discovery of an object of _malicious software code_.

Of course, the viability of the _source trace_ procedure must also
depend on a manner of a sense of cooperation among parties
participating in the whole _software distribution tree_, such that a
_software distribution tree_ may be traversed in any individual
_source trace_ procedure.

The Autoport Mangement Schema project, thus, is proposed as to define
a manner of data structures for application in  _provenance_
procedures, and for application in _project idenity_, such as may be
supportive of individual _source trace_ procedures, in contexts of
_issue tracking_ and in a broader context of _forensic
analysis_.

It is proposed that the data structures defined of this project will
be defined onto the syntax and semantics of a _Resource Description
Format_ (RDF) _Schema_, such that the data structures defined of the
respective RDF Schema may be applied in constructing an  _RDF graph_,
such as may be stored in RDF _persistence_ formats, or stored of _RDF
tuple_ data within any individual _graph databases_. The RDF schema
may be developed as to not require any specific adoption of any
singular data storage method, thus serving to facilitate a manner 
of information sharing with regards to software sources, moreover
without requiring any formal adoption of any singular software
platform for analysis of the same _graph data_.


# Structure of the Poudriere Logs

Pathname: `/usr/local/poudriere/data/logs/bulk`

* Files

    * `.data.json`

        * Provides an effective index of _build masters_ as entries
          within the `masternames` JSON expression
        
            *  **Ed. Note::* The complete naming patern for _build
               master_ names must be represented as a regular
               expression, _viz a viz_ 
              `"${JAILNAME}-${TREENAME}" [ "-${SETNAME}" ]`
      
        
* Subdirectories

    * One subdirectory for each `masternames` element of `.data.json`

    * `latest-per-pkg` Subdirecdtory

    * `.html` Subdirectory


[pou]: https://github.com/freebsd/poudriere/
[fbsd]: http://www.freebsd.org/
[pou_hook]: https://github.com/freebsd/poudriere/wiki/hooks
