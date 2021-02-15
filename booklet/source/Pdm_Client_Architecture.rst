
.. _pdm-client-architecture:

Pdm Client Architecture
***********************

.. 
  References would be done like this: :ref:`pdm-client-architecture`.
  :ref:`PdmAbstractRootAppBuilder <pharo-class-pdmabstractrootappbuilder>`

Overview
========

The executable software performing operations on document management is called the pdm client. To perform its operations, it has to be connected with the following services:

* A PostgreSQL database service providing access to all persistent application data
* A delta file service managing download and upload of remotely stored document files as well as preview images

To deal with document files on client side, e.g. to open them from within the pdm executable, a special executable is called that is part of every pdm installation (PdmOfficeTool.exe).

Abstract architecture
---------------------

..
  :pharo:mref:`PdmSubsystem class>>#checkPrerequisitesIn:` and
  :ref:`PdmSubsystem class>>#checkPrerequisitesIn: <pharo-compiledMethod-PdmSubsystem-class-checkPrerequisitesIn->`

The pdm client is based on an abstract architecture providing support for desktop applications of any purpose:

* A system class based on  :pharo:cref:`PdmAbstractRootAppBuilder` performs the startup via :pharo:mref:`PdmAbstractSystem class>>#start`.
* System startup first performs a search of the available system configuration (e.g. via environment variable
  or predefined file name) to find the respective subclass of :pharo:cref:`PdmAbstractSystemConfig` providing 
  the concrete configuration of the system (see :ref:`pdm-system-configuration-types`).
* All concrete functionality expected to be available in the started application has to be defined inside of 
  the chosen configuration instance. Available functionalities are organized by means of subsytems 
  (see :pharo:cref:`PdmSubsystem` and :ref:`pdm-configuration-of-subsystems`). So what a system config contains is 
  in fact a list of subsystem configurations (see class :pharo:cref:`PdmSubsystemConfig`).
* A subsystem configuration is a generic key-value-store that is associated with the name of the subsystem.

Based on this architecture, all subsytems are created and setup in the given order. Most important message herefor are:

* :pharo:mref:`PdmSubsystem>>#checkSubsystemConfiguration` to validate the specific configuration settings, returning
  either a string describing the problem or nil.
* :pharo:mref:`PdmSubsystem>>#privateSetup` to make the subsystem ready for being started. It is only called after successful
  run of :pharo:mref:`PdmSubsystem>>#checkSubsystemConfiguration`.

Subclass of PdmAbstractSystem representing the pdm application is :pharo:cref:`PdmSystem`.

Subsystem setup errors
^^^^^^^^^^^^^^^^^^^^^^

After trying to prepare a subsystem, any occurring problem text is kept in
instance variable ``setupProblemText`` of :pharo:cref:`PdmSubsystem` for
further handling. Mind that a user-friendly error handling is not yet possible
as the needed subsystems will not have been started at that moment.

Subsystem dependencies
^^^^^^^^^^^^^^^^^^^^^^

Some of the systems typically expect others to exist and being started before they can be started either. Default implementation to check dependencies is

.. pharo:autocompiledmethod:: PdmSubsystem_class>>#checkPrerequisitesIn:


Default implementation of :pharo:mref:`PdmSubsystem class>>#prerequisiteSubsystemKeys` is to return an empty array. A subclass may override this by returning the Array of name strings of those subsystems they expect to already have been defined.

Starting and stopping the system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After all subsystems have been setup successfully, the system is started by sending :pharo:mref:`PdmSubsystem>>#startUp` to each of the configured subsystems in the given order of definition.

When the concrete system instance later will be closed, it receives message

.. pharo:autocompiledmethod:: PdmAbstractSystem>>#stopSubsystems

By that it will stop the subsystems in reverse order of definition. Normally, a subsytem will not need to perform any action here, but may disconnect from external resources.

Root application builder
^^^^^^^^^^^^^^^^^^^^^^^^

Wheras most of the subsystems are modules to provide certain functionality like database access or a client to load document files from the delta server, a subsystem inheriting from class :pharo:cref:`PdmAbstractRootAppBuilder` is always expected to exist. This root application builder subclass has the purpose of opening an application for interaction, in our case the application for pdm document manangement which is created by the subclass :pharo:cref:`PdmAbstractRootAppBuilder`.

As our application especially supports a GUI design based on a master ribbon panel containing switchable tabs for several subapplications, already the abstract base class supports this pattern of a root application at the top level managing several subapplications:

.. pharo:autoclass:: PdmAbstractRootAppBuilder

As the root application builder is still one subsystem among several others (which it is mostly a dependent of), it has to be recognized by the whole framework in certain cases, and thus implements

.. pharo:autocompiledmethod:: PdmAbstractRootAppBuilder>>#isRootAppBuilder

.. note::

  When this need changes and the referred method will be deleted, which is expected to happen in the near future, the paragraph above will be obsolete, which should be detected by the documentation generator.
 
From a software design point of view, one might think that "the application itself" is logically not a subsystem, but something different instead that always exists. On a closer look, every application instance somehow has to be constructed, typically based on a configuration. Modern applications will also not only provide one general-purpose layout, but support different types of appearance, maybe even depending of what subsystems in which configurations are available. From that perspective, representing the builder of the main application as a subsystem of a specific subclass (:pharo:cref:`PdmAbstractRootAppBuilder`) can be considered as acceptable as well as useful: The root application builder thus can reuse the basic subsystem behavior, especially for checking configuration and dependencies.

.. _pdm-system-configuration-types:

System configuration types
==========================

A system configuration by itself only decides how configuration data are retrieved on startup, according to the different subclasses preimplemented below :pharo:cref:`PdmSystemConfig`. All of them implement mainly the same method, as it is shown here for class :pharo:cref:`PdmSystemConfigDefault` which provides a default configuration:

.. pharo:autocompiledmethod:: PdmSystemConfigDefault>>#readSubsystemConfigsForTokens:

This default configuration is used in case no other configuration can be found, and also serves the purpose to document the available settings directly in the code. Reading configuration data from JSON file is done by

.. pharo:autocompiledmethod:: PdmSystemConfigJsonFile>>#readSubsystemConfigsForTokens:

An other possibility is 

.. pharo:autocompiledmethod:: PdmSystemConfigCode>>#readSubsystemConfigsForTokens:

Please mind that the implementation in any subclass is not a normal overwrite: As the name of the class to use is not known in advance, but passed by an environment variable, the superclass implementation is always called first. By that, it is also made sure that only a subclass of PdmSystemConfigCode may be called this ways.

(to be continued: Using the environment variable ``PdmSystemConfigMode``)

.. _pdm-configuration-of-subsystems:

Configuration of subsystems
===========================

(to be continued)



