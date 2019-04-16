# Plan for NSLS-II JupyterHub at SDCC

## Goals
- Make it easy for BL staff to deploy new environments+kernels and make them
  available to their users. It is currently possible to make custom environments
  for personal use, but the only way to _share_ them is to say, "Run these conda
  and ipykernel commands," which is not streamlined enough for general users.
- Provide long-term (~3 year) support for kernels. An old notebook should be able to start
  its old kernel, so that users can, for example, update an old plot without
  dealing with any API changes in the related libraries.
- Make it possible for users to switch between beamlines easily and have
  notebooks from multiple beamlines open at once.
- Avoid overwhelming users with a flat list of many kernels.

## Deployment and Computational Resources

### Short term

We will be using the shared "HTC" JupyterHub provided by SDCC. They will add one
"NSLS-II" kernel to the default list. Individual users can, of course, create
their own environments or kernels in the standard way.  (See
[instructions](https://gist.github.com/danielballan/208bdf0a6b2741fed7ce92721d47fade).)

### Long term

We will move the servers designation for scientific computing currently sitting
idle at NSLS-II to SDCC. Once those are configured, SDCC will stand up a
separate JupyterHub specifically for NSLS-II. We can configure it however we
like and create as many kernels as we like.

### Resource Limits

Currently SDCC is taking a "wait and see" approach. Any kernel (or process in
general) that allocates 16 GB of RAM is killed. In the short term this is
sufficient. In the longer term, once NSLS-II has its own Hub, we could consider
Docker or even Kubernetes for resource limiting. Another possiblity is "systemd
spawner" but its resource limiting is known to be quirky.
(See [this talk](https://research.cs.wisc.edu/htcondor/HTCondorWeek2017/presentations/WedDownes_cgroups.pdf).)

## Data Access

### User Interface

Users will access all data through databroker. Currently, that means using
databroker's Python API to load the data into a Python process and either work
on it there or export it to files. In time, an HTTP API, a web interface, and
other GUI interfaces will be developed by DAMA and other bluesky collaborators.

### Databases

We will make the central MongoDB instance visible from SDCC. (???)

### Files from Detectors

We will continually sync the data from beamlines to storage at SDCC, so that any
outages on the NSLS-II side only affect the availability of *new* data, not
*all* saved data. The data will move from the beamline storage through the
NSLS-II central GPFS to the NSLS-II's data transfer node, to SDCC's data
transfer node, to ???

## Kernel Management 

### Status Quo

Currently, DAMA deploys one Juptyer kernel and associated conda environment per
beamline, per cycle---sometimes a couple per cycle, for minor updates. Because
we provide long-term support for these, we have accumulated a large number of
kernels and many environments that are very similar but not identical.

### Long-Term Support Kernels

At the start of a cycle, deploy a small set of "stable" kernels+environments
that will receive long-term support. Instead of making one per beamline,
start with one "NSLS-II base" environment with the union of all requirements
(csxtools, sixtools, etc.) In the future, create additional environments as
needed when we encounter conflicting or significantly different set of
requirements.  This is similar Jupyter's
[docker stacks](https://github.com/jupyter/docker-stacks), with images like
"minimal", "scipy", "data science", "pyspark", etc.

### Fast Delivery of New Code via "Latest" Kernel

Every night, automatically build a new environment with a unique name. This will
pick up the latest versions of all the packages in that environment, such as
sixtools. For situations where "nightly" is not fast enough, where staff want to
deliver a new update immediately, provide a way for staff to trigger the build
manually at any time (say, with a button on a webpage).

Provide *one* kernel, "NSLS-II base latest", whose target environment is
automatically moved to point to the newest environment every time a build runs.
This means that every time a notebook on the "NSLS-II base latest" has its
kernel restarted, it may be pointing at a different environment than before.
Automatically delete old "latest" environments, but not *immediately*. Notebooks
that have been running for a couple days will still be attached to the
environment that was newest *when they started* and that environment should not
be ripped out from underneath them while they are still running.

The story for deploying a fix to some library like sixtools for users then, is:
* Merge a PR with a fix.
* Tag the project (and employ CI/CD tools to automatically create a package on
  PyPI / conda).
* Click the "Build a new latest environment!" button and wait for it to run.
* Tell the user to restart their kernel, which will automatically bind to the
  updated environment with the new release of the library.
