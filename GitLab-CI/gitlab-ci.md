1. Difference between "needs" and "dependencies"
   dependencies:
    - Dependencies define which artifacts should be available for a particular job, rather than "dependencies" in the usual sense (such as waiting for other jobs to be completed).
    - If dependencies are not specified in the job, all jobs from earlier stages are considered as sources of artifacts, and the job downloads all artifacts from them.
    - If you specify an empty array (dependencies: []), the job does not download any artifacts, but continues to wait for all jobs from the previous stages to be completed.
    - If you specify specific jobs in dependencies, the completion of all previous stages is still expected, but artifacts are loaded only from the specified jobs.
  
   needs:
    - Unlike dependencies, which focus on artifacts, needs defines the actual dependencies between the execution of jobs. This means that jobs with the specified needs will wait for these jobs to be completed, regardless of their stage (including the current one).
    - If you do not specify needs, the behavior will be standard and depend on other parameters.
    - The empty array needs: [] indicates that the job can be started as soon as the pipeline is created, without waiting for jobs to be completed in other stages.
    - If a specific job is specified in needs, the current job will wait for its completion, and download its artifacts by default. You can disable automatic downloading of artifacts by adding the artifacts: false option.
   
   Note: Don't use needs and dependencies together for the same job unless you have a specific reason; GitLab documents that combining them can lead to unexpected behavior.
