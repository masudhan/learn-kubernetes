**Jobs and CronJobs**

We know that, there are

`initContainers` -> these containers must complete their task before starting the main/application container

`application/main containers` -> application container which runs your application codes

`adapter containers` -> this container run along with application/main container to support like copying files or logs

For example, out requirement is like, i want one container to be start every one hour which will go and collect the logs from adapter container and place them at some location and terminate the pod that's when jobs will be helpful

`jobs` -> one time container which will perform your task.

`cronjob` -> scheduled job like taking backup every 10minutes 


A Job creates one or more Pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created.

Jobs looks similar to init containers, but the major difference is, the main application containers don't start until init containers completes the job. Init containers can be used for initial copy of the database. Where jobs can be used for data migration and run independently with no dependency on the main container.

CronJobs are useful for creating periodic and recurring tasks, like running backups or sending emails. CronJobs can also schedule individual tasks for a specific time, such as scheduling a Job for when your cluster is likely to be idle.

`k create job test-job --image busybox --dry-run=client -o yaml` or `k apply -f job.yaml`


```
❯ k get pods
NAME             READY   STATUS              RESTARTS   AGE
test-job-fkgmm   0/1     ContainerCreating   0          2s
❯ k get jobs
NAME       COMPLETIONS   DURATION   AGE
test-job   1/1           5s         6s

After some time,

❯ k get jobs
NAME       COMPLETIONS   DURATION   AGE
test-job   1/1           5s         20s
❯ k get pods
NAME             READY   STATUS      RESTARTS   AGE
test-job-fkgmm   0/1     Completed   0          22s

❯ k logs test-job-rht76
CronJob has started and completed


```

Once the job gets completed it's not gonna delete the pod, we need to do it manually

```
❯ k delete job test-job
job.batch "test-job" deleted
```

*Notes*
When a job is running and if you kill the pod, it's gonna run again and again untill it completes and exit code

Now here job completions is 1/1, which means i has ran only once and that's it, what if we want to run for 5 iterations, that's where you need to specify job specification
```
❯ k get jobs
NAME       COMPLETIONS   DURATION   AGE
test-job   1/1           5s         46s
```
`k apply -f job-v2.yaml` - job is gonna run sequentially 

```
❯ k get jobs,pods
NAME                 COMPLETIONS   DURATION   AGE
job.batch/test-job   3/3           15s        24s

NAME                 READY   STATUS      RESTARTS   AGE
pod/test-job-2gsxd   0/1     Completed   0          14s
pod/test-job-4vz4j   0/1     Completed   0          24s
pod/test-job-gxrmc   0/1     Completed   0          19s

```

We can also say how many jobs you want to run parallelly

`k apply -f job-v3.yaml`

```
❯ k get jobs,pods
NAME                 COMPLETIONS   DURATION   AGE
job.batch/test-job   2/2           10s        13s

NAME                 READY   STATUS      RESTARTS   AGE
pod/test-job-4fkkw   0/1     Completed   0          13s
pod/test-job-sxk7m   0/1     Completed   0          13s

```

There is something called as `backoffLimit` which is, if job is fails for 2 times and that's it.. it won't create new pods, `backoffLimit` default value is 6

`k apply -f job-v4.yaml`

```
❯ k get jobs,pods
NAME                 COMPLETIONS   DURATION   AGE
job.batch/test-job   0/1           14s        14s

NAME                 READY   STATUS   RESTARTS   AGE
pod/test-job-hgfs2   0/1     Error    0          14s
pod/test-job-kdjln   0/1     Error    0          4s
pod/test-job-ppf6h   0/1     Error    0          9s
```

We can also set a threshold saying, lets say you know you job is going to take 20sec, and if it takes more than that just terminate the job. I don't want to keep the job running forever

`k apply -f job-v5.yaml`

```
❯ k get jobs,pods
NAME                 COMPLETIONS   DURATION   AGE
job.batch/test-job   0/1           11s        11s

NAME                 READY   STATUS        RESTARTS   AGE
pod/test-job-xg5rm   1/1     Terminating   0          11s

❯ k describe job.batch/test-job

  Volumes:        <none>
Events:
  Type     Reason            Age   From            Message
  ----     ------            ----  ----            -------
  Normal   SuccessfulCreate  64s   job-controller  Created pod: test-job-xg5rm
  Normal   SuccessfulDelete  54s   job-controller  Deleted pod: test-job-xg5rm
  Warning  DeadlineExceeded  54s   job-controller  Job was active longer than specified deadline


```


Now whenever you want to schedule a job on periodically then use `CronJob` - it is very similar to job specification, extra one we got it jobtemplate and [schedule](https://en.wikipedia.org/wiki/Cron#:~:text=%23%20*%20*%20*%20*%20*%20%3Ccommand%20to%20execute%3E)

`k apply -f cronjob.yaml` - job is going to run every one minute

```
❯ k get pods,cj
NAME                                   READY   STATUS      RESTARTS   AGE
pod/deamonkiller-cron-27850559-92lpf   0/1     Completed   0          9s

NAME                              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/deamonkiller-cron   */1 * * * *   False     0        9s              49s

❯ k logs pod/deamonkiller-cron-27850559-92lpf
Hello deamonkillerM!!

After 1 minute,

❯ k get pods,cj
NAME                                   READY   STATUS      RESTARTS   AGE
pod/deamonkiller-cron-27850559-92lpf   0/1     Completed   0          65s
pod/deamonkiller-cron-27850560-2f92t   0/1     Completed   0          5s

NAME                              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob.batch/deamonkiller-cron   */1 * * * *   False     1        5s              105s

```
It is goint to 3 last successful jobs and 1 failed job you can also edit them by adding `successfulJobsHistoryLimit` and `failedJobsHistoryLimit` key-value pairs

`k apply -f cronjob-v2.yaml`

Now, lets say if you want to suspend the job, it's not gonna stop the existing jobs, it'll be applicable for future jobs

There are two ways,
  1. add `suspend: true`  - `k apply -f cronjob-v2.yaml` after sometime add `k apply -f cronjob-v3.yaml`, again to resume edit the yaml file 
  2. using cli - `k patch cronjob deamonkiller-cron -p '{"spec":{"suspend":false}}'



*concurrencypolicy*
It can take 3 values:
  1.  Allow (default) - do you want a job to run when there is a job which is already in progress, you can more more than one job running at the same time
  2.  Forbid - if the job is already running, wait for that one to get completed and run a new job
  3.  Replace - if the job is already running, and there is another job which is in queue to be scheduled then replace the existing job

**Usecases**
1.  mysql backup periodically
2.  regular backup
3.  send emails
4.  send notifications
5.  checkout git repository