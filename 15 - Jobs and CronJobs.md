# 15 · Jobs and CronJobs

Work that is supposed to **finish**, not run forever. A Deployment restarts a failed pod; a Job retries a pod until it succeeds once.

| Serial | Command |
|--------|---------|
| K8S-040 | `kubectl get jobs` |

## How a Job differs from a Deployment

| | Deployment | Job |
|---|---|---|
| Goal | Keep N pods running forever | Run until one exits `0` |
| On app exit | Restart it, that is a failure | Success, job is done |
| Pod after success | Kept running | `Completed`, kept for logs |
| `restartPolicy` | Always | `Never` or `OnFailure` (required) |
| Retry budget | n/a | `backoffLimit`, default 6 |

Typical uses: database migrations, one-off cleanups, nightly reports, seeding data.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: synergychat-stats
spec:
  backoffLimit: 4
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: synergychat-stats
          image: curlimages/curl:latest
          args:
            - -sf
            - http://crawler-service.crawler.svc.cluster.local/stats
```

`restartPolicy: Never` starts a **new pod** for each retry. `OnFailure` restarts the **container in the same pod** instead. Neither is optional on a Job.

## K8S-040 · `kubectl get jobs`

- **First seen:** [[Ch11 Jobs]] → Jobs (assignment)
- **Reused:** Ch11 · CronJobs (one Job per schedule tick)
- **Purpose:** See whether a Job finished and how many completions it has.

```sh
kubectl get jobs
```

Typical columns: `NAME  STATUS  COMPLETIONS  AGE` (older clusters include `DURATION`). Wait for `COMPLETIONS 1/1` and `STATUS Complete`.

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-o yaml` | Full status, including `succeeded`, `failed`, conditions |
| `-w` | Watch it finish |

**Notes**

- `job` is the short name. To pair with the pods it created: `kubectl get pods -l job-name=synergychat-stats`.
- Job pods go to `Completed`, not `Running`. That is success, not an error state.
- `STATUS Failed` means `backoffLimit` was exhausted. Check `kubectl logs` and `describe pod` for the actual error.
- Pods from a Job are kept after completion so their logs survive. Deleting the Job deletes them too, which is why "read the output before you clean up".
- For CronJobs, `kubectl get cronjobs` lists the schedules and `kubectl get jobs` shows the runs they spawned, named `<cronjob>-<number>`.

## CronJobs

A CronJob is a Job plus a schedule. Its `jobTemplate` is literally the Job spec.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: synergychat-stats-report
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      backoffLimit: 4
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: synergychat-stats
              image: curlimages/curl:latest
              args:
                - -sf
                - http://crawler-service.crawler.svc.cluster.local/stats
```

**Schedule syntax** — five fields, then `*` means "every":

```text
┌──────── minute      (0-59)
│ ┌────── hour        (0-23)
│ │ ┌──── day of month (1-31)
│ │ │ ┌── month        (1-12)
│ │ │ │ ┌ day of week  (0-6, Sunday = 0)
* * * * *
```

| Schedule | Runs |
|----------|------|
| `* * * * *` | Every minute |
| `0 * * * *` | Top of every hour |
| `0 2 * * *` | Daily at 02:00 |
| `30 9 * * 1` | Mondays at 09:30 |

**Notes**

- History is capped: the last 3 successful and 1 failed Job by default. Change with `successfulJobsHistoryLimit` and `failedJobsHistoryLimit`.
- `suspend: true` pauses the schedule without deleting the CronJob. Existing Jobs are unaffected.
- Kubernetes uses UTC for schedules unless the CronJob sets `timeZone`.
- Overlapping runs are governed by `concurrencyPolicy` (`Allow` by default, also `Forbid` and `Replace`).
- Jobs are created with a numeric suffix (`synergychat-stats-report-29123456`), so a single failing run is easy to isolate.
