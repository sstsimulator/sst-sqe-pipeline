# sst-sqe-pipeline
Pipeline code for SST Jenkins Testing

# Jenkinsfiles
Each Jenkinsfile is written using scripted syntax and uses groovy. The Jenkinsfile modifies ann existing jenkins job configuration when the job is run.
## sqe
This file corresponds the to root Jenkins job that is responsible for spawning sub-jobs. This job accepts nine parameters which affect which stages are executed in each of sqe's sub-jobs. *Note that parameters in sqe are exported as bash variables while parameters passed into sub-jobs have more restricted scope.* Note that the `P_NODE_LABEL` parameter affects where all *sub-jobs* are run.
Sqe reports a summary status of all spawned as a result of running the root sqe Jenkins job. Each sub-job is responsible for reporting a verbose status of itself.
## core
This file is responsible for cloning, building, installing, and testing sst-core. Once the core job passes its tests then spawns parallel sub-jobs accordingly and passes the sst-core install path to the sub-jobs. Control is not returned to the core job until all sub-jobs have completed or timedout.
## elements
This file is responsible for cloning, building, installing, and testing elements. Once the elements job reaches a stop condition, it returns control to it parent job.

## Troubleshooting and TODOs
- Some of the node labels specified via `P_NODE_LABEL` have limited resources. For example, the elements sub-job may not be able to execute if all the node's resources are being consumed by sqe and core -- even though sqe is waiting for control from core and core is waiting for control from elements.
- Both core and elements Jenkinsfiles need to be updated to use spack for building and installing sst-core/sst-elements.
- Sqe Jenkinsfile needs to be updated with a "scenario" list that will pass scenarios to core which will affect how spack builds core/elements/others.
- The spack configuration in sst-sqe-pipeline/.spack needs to be modified such that only specified dependencies are built (we don't need to rebuild gcc, autotools, etc if they are already available). In addition, the spack config should prefer dependencies available from modules rather than building then from source. For the "spack variants" example below, zoltan should be built from source while metis, glpk, and openmpi should be available via modules (to get started, see https://spack-tutorial.readthedocs.io/en/latest/tutorial_configuration.html#advanced-compiler-configuration).

# Building sst-core with spack:release/v0.13
Install spack by running:
```bash
git clone https://github.com/spack/spack
cd spack
git checkout releases/v0.13
. share/spack/setup-env.sh
```

Building sst-core:devel via spack:
```bash
export SST_SPACK=/path/to/sst-sqe-pipeline/.spack
spack -C $SST_SPACK spec sst-core@devel
```

Build sst-core:devel with some variants via spack:
```bash
spack -C $SST_SPACK spec sst-core@devel +zoltan +metis +glpk ^zoltan@3.83 ^metis@5.1.0 ^glpk@4.54 ^openmpi@2.1.3
```