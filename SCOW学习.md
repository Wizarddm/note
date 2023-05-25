/libs/slurm/src/getRunningJobs.ts

获取进行中的作业方法

```js
export async function querySacct(ssh: NodeSSH, username: string, filterOptions: RunningJobFilter, logger: Logger) {
  const { userId, accountNames, jobIdList } = filterOptions;
  const result = await executeAsUser(ssh, username, logger, true,
    "sacct",
    [
      "--noheader",
      "--state=RUNNING,PENDING",
      "-P",
      `--delimiter=${SEPARATOR}`,
      "-o",
      "JobID,Partition,JobName,User,State,Elapsed,AllocNodes,NodeList,"
      + "Account,AllocCPUs,QOS,Submit,NodeList,Timelimit,WorkDir",
      // replace %Y with NodeList temporarily
      ...userId ? ["-u", userId] : [],
      ...accountNames ? (accountNames.length > 0 ? ["-A", accountNames.join(",")] : []) : [],
      ...jobIdList ? (jobIdList.length > 0 ? ["-j", jobIdList.join(",")] : []) : [],
    ],
  );

  const jobs = result.stdout.split("\n").filter((x) => x).map((x) => {
    const [
      jobId,
      partition, name, user, state, runningTime,
      nodes, nodesOrReason, account, cores,
      qos, submissionTime, nodesToBeUsed, timeLimit, workingDir,
    ] = x.split(SEPARATOR);

    return {
      jobId,
      partition, name, user, state, runningTime,
      nodes, nodesOrReason, account, cores,
      qos, submissionTime, nodesToBeUsed, timeLimit,
      workingDir,
    } as RunningJob;
  });

  return jobs;
  
}
```

