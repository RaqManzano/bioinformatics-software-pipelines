---
title: "Nextflow on HPC"
---

How to configure Nextflow to run on a HPC.

- Config file - e.g. setting SLURM options to use
- Use `tmux`/`screen` to get a persistent terminal on a remote server


## HPC configuration

To run on a HPC, we can specify a profile specific to our HPC. 
For example, to work at the Cambridge University HPC we created the following configuration file:

```conf
process {
  executor = 'slurm'
  
  // generic option
  clusterOptions = '--account LEYSER-SL2-CPU --partition cclake'

  // Settings below are for CSD3 nodes detailed at
  //   https://docs.hpc.cam.ac.uk/hpc/index.html
  // Current resources (Jun 2023):
  //   icelake: 76 CPUs; 3380 MiB per cpu; 6760 MiB per cpu (himem)
  //   cclake: 56 CPUs; 3420 MiB per cpu; 6840 MiB per cpu (himem)
  // The values used below were chosen to be multiples of these resources
  // assuming a maximum of 2 retries

  // Using himem partition to ensure enough memory for single-CPU jobs
  withLabel:process_single {
      cpus   = { check_max( 1                  , 'cpus'    ) }
      memory = { check_max( 6800MB * task.attempt, 'memory'  ) }
      time   = { check_max( 4.h  * task.attempt, 'time'    ) }
      clusterOptions = '--account LEYSER-SL2-CPU --partition cclake-himem'
  }
  // 4 CPUs + 13GB RAM
  withLabel:process_low {
      cpus   = { check_max( 4     * task.attempt, 'cpus'    ) }
      memory = { check_max( 13.GB * task.attempt, 'memory'  ) }
      time   = { check_max( 4.h   * task.attempt, 'time'    ) }
      clusterOptions = '--account LEYSER-SL2-CPU --partition cclake'
  }
  // 8 CPUs + 27GB RAM
  withLabel:process_medium {
      cpus   = { check_max( 8     * task.attempt, 'cpus'    ) }
      memory = { check_max( 27.GB * task.attempt, 'memory'  ) }
      time   = { check_max( 8.h   * task.attempt, 'time'    ) }
      clusterOptions = '--account LEYSER-SL2-CPU --partition cclake'
  }
  // 12 CPUs + 40GB RAM
  withLabel:process_high {
      cpus   = { check_max( 12    * task.attempt, 'cpus'    ) }
      memory = { check_max( 40.GB * task.attempt, 'memory')}
      time   = { check_max( 8.h  * task.attempt, 'time'    ) }
      clusterOptions = '--account LEYSER-SL2-CPU --partition cclake'
  }
  // Going by chunks of 12h (2 retries should bring it to max of 36h)
  withLabel:process_long {
      time   = { check_max( 12.h  * task.attempt, 'time'    ) }
  }
  // A multiple of 3 should bring it to max resources on cclake-himem
  withLabel:process_high_memory {
      cpus   = { check_max( 18     * task.attempt, 'cpus'    ) }
      memory = { check_max( 127.GB * task.attempt, 'memory' ) }
      clusterOptions = '--account LEYSER-SL2-CPU --partition cclake-himem'
  }
  withLabel:error_ignore {
      errorStrategy = 'ignore'
  }
  withLabel:error_retry {
      errorStrategy = 'retry'
      maxRetries    = '2'
  }
}

params {
  max_memory = '327.GB' // for cclake-himem
  max_cpus = '56'       // for cclake nodes
  max_time = '36.h'     // for SL2 service level
}

executor {
  queueSize         = '2000'
  pollInterval      = '3 min'
  queueStatInterval = '5 min'
  submitRateLimit   = '50sec'
  exitReadTimeout   = '5 min'
}

singularity {
  singularity.enabled = 'true'
  pullTimeout = '1 h'
  cacheDir = '/home/hm533/rds/hpc-work/nextflow-singularity-cache'
}
```



:::{.callout-exercise}

- Create a configuration file for our "HPC".
- Singularity cache in custom directory.
- Re-run the pipeline and see if it is submitting jobs to the scheduler as expected. 

:::