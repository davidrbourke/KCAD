# Jobs

Running a Pod so it shuts down after the required task is complete.

One way around this is to set the **restartPolicy: never**

A Job is used to run a set of Pods for a given task to completion. The apiVersion is **apiVersion: batch/v1**.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: match-add-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '4']
      restartPolicy: Never
```

`kubectl create -f <job-definition>.yaml`  

The job would usually have an output, e.g. generate an image to a persistent volume, email a report, etc.  

**completions** sets the number of Pods, running in sequence, the second only starts after the first completes.
The completions means the number of successful completions, so if you have 3 completions and 3 failures, 6 pods would have run in total to get 3 successful completions.

To create Pods in parallel, set **parallelism: 3**:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: match-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '4']
      restartPolicy: Never
```

## Cron Jobs

A job that can be scheduled. The apiVersion is **batch/v1**. There are 3 **spec** sections:
1. The Cron section
2. The Job section
3. The Pod section

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: reporting-tool
              image: reporting-tool
          restartPolicy: Never
```

To see the cronjobs:  
`kubectl get cronjob`  

### Job retrying
By default there are 6 retries (default backoffLimit), you can configure this with the backoff limit. The restartPolicy is set to **Never*8, so the Pod will not try to restart on Failure, but the Job will try again.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-job
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 10
  template:
    spec:
      containers:
        -  image: kodekloud/throw-dice
           name: throw-dice
      restartPolicy: Never                  
```