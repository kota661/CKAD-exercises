![](https://gaforgithub.azurewebsites.net/api?repo=CKAD-exercises/pod_design&empty)
# Pod design (20%)

## Labels and annotations
kubernetes.io > Documentation > Concepts > Overview > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

### nginx1、nginx2、nginx3という名前の3つのポッドを作成します。 それらのすべてにラベルapp = v1が必要です
Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

<details><summary>show</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
```

</p>
</details>

### ポッドのすべてのラベルを表示する
Show all labels of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### ポッド「nginx2」のラベルをapp = v2に変更します
Change the labels of pod 'nginx2' to be app=v2

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
```

</p>
</details>

### ポッドの「アプリ」ラベルを取得する
Get the label 'app' for the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -L app
# or
kubectl get po --label-columns=app
```

</p>
</details>

### 「app = v2」ポッドのみを取得します
Get only the 'app=v2' pods

<details><summary>show</summary>
<p>

```bash
kubectl get po -l app=v2
# or
kubectl get po -l 'app in (v2)'
# or
kubectl get po --selector=app=v2
```

</p>
</details>

### 以前に作成したポッドから「アプリ」ラベルを削除します
Remove the 'app' label from the pods we created before

<details><summary>show</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# or
kubectl label po nginx{1..3} app-
# or
kubectl label po -lapp app-
```

</p>
</details>

### 「accelerator = nvidia-tesla-p100」というラベルが付いたノードにデプロイされるポッドを作成します
Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

<details><summary>show</summary>
<p>

We can use the 'nodeSelector' property on the Pod YAML:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # add this
    accelerator: nvidia-tesla-p100 # the selection label
```

You can easily find out where in the YAML it should be placed by:

```bash
kubectl explain po.spec
```

</p>
</details>

### ポッドnginx1、nginx2、nginx3に「description = 'my description'」の値で注釈を付ける
Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value

<details><summary>show</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

#or

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### ポッドnginx1のアノテーションを確認する
Check the annotations for pod nginx1

<details><summary>show</summary>
<p>

```bash
kubectl describe po nginx1 | grep -i 'annotations'
```

As an alternative to using `| grep` you can use jsonPath like `kubectl get po nginx -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### これら3つのポッドのアノテーションを削除します
Remove the annotations for these three pods

<details><summary>show</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description-
```

</p>
</details>

### これらのポッドを削除して、クラスターをクリーンな状態にします
Remove these pods to have a clean state in your cluster

<details><summary>show</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Controllers > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### このコンテナが公開するポートとしてポート80を定義して、2つのレプリカを持つnginxと呼ばれるイメージnginx：1.7.8のデプロイメントを作成します（このデプロイメントのサービスを作成しないでください）
Create a deployment with image nginx:1.7.8, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)

<details><summary>show</summary>
<p>

```bash
kubectl run nginx --image=nginx:1.7.8 --replicas=2 --port=80
```

**However**, `kubectl run` for Deployments is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run -o yaml > deploy.yaml
vi deploy.yaml
# change the replicas field from 1 to 2
# add this section to the container spec and save the deploy.yaml file
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

or, do something like:

```bash
kubectl create deployment nginx  --image=nginx:1.7.8  --dry-run -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.7.8/image: nginx:1.7.8\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

</p>
</details>

### このデプロイメントのYAMLを表示する
View the YAML of this deployment

<details><summary>show</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### この展開によって作成されたレプリカセットのYAMLを表示する
View the YAML of the replica set that was created by this deployment

<details><summary>show</summary>
<p>

```bash
kubectl describe deploy nginx # you'll see the name of the replica set on the Events section and in the 'NewReplicaSet' property
# OR you can find rs directly by:
kubectl get rs -l run=nginx # if you created deployment by 'run' command
kubectl get rs -l app=nginx # if you created deployment by 'create' command
# you could also just do kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

</p>
</details>

### いずれかのポッドのYAMLを取得する
Get the YAML for one of the pods

<details><summary>show</summary>
<p>

```bash
kubectl get po # get all the pods
# OR you can find pods directly by:
kubectl get po -l run=nginx # if you created deployment by 'run' command
kubectl get po -l app=nginx # if you created deployment by 'create' command
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

</p>
</details>

### deployment rollout状況を確認する
Check how the deployment rollout is going

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### nginxイメージをnginx：1.7.9に更新します
Update the nginx image to nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.7.9
# alternatively...
kubectl edit deploy nginx # change the .spec.template.spec.containers[0].image
```

The syntax of the 'kubectl set image' command is `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

</p>
</details>

### ロールアウト履歴を確認し、レプリカに問題がないことを確認します
Check the rollout history and confirm that the replicas are OK

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # check that a new replica set has been created
kubectl get po
```

</p>
</details>

### 最新のロールアウトを元に戻し、新しいポッドに古いイメージがあることを確認します（nginx：1.7.8）
Undo the latest rollout and verify that new pods have the old image (nginx:1.7.8)

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# wait a bit
kubectl get po # select one 'Running' Pod
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # should be nginx:1.7.8
```

</p>
</details>

### 間違ったイメージnginx：1.91を使用して、デプロイメントを更新します。
Do an on purpose update of the deployment with a wrong image nginx:1.91

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# or
kubectl edit deploy nginx
# change the image to nginx:1.91
# vim tip: type (without quotes) '/image' and Enter, to navigate quickly
```

</p>
</details>

### ロールアウトに問題があることを確認します
Verify that something's wrong with the rollout

<details><summary>show</summary>
<p>

```bash
kubectl rollout status deploy nginx
# or
kubectl get po # you'll see 'ErrImagePull'
```

</p>
</details>


### デプロイメントを2番目のリビジョン（番号2）に戻し、イメージがnginx：1.7.9であることを確認します
Return the deployment to the second revision (number 2) and verify the image is nginx:1.7.9

<details><summary>show</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Everything should be OK
```

</p>
</details>

### 4番目のリビジョン（番号4）の詳細を確認する
Check the details of the fourth revision (number 4)

<details><summary>show</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # You'll also see the wrong image displayed here
```

</p>
</details>

### デプロイメントを5つのレプリカにスケーリングする
Scale the deployment to 5 replicas

<details><summary>show</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

</p>
</details>

### 5〜10のポッドでデプロイメントを自動スケーリングし、CPU使用率を80％に目標とする
Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%

<details><summary>show</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
```

</p>
</details>

### デプロイメントのロールアウトを一時停止します
Pause the rollout of the deployment

<details><summary>show</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### イメージをnginx：1.9.1に更新し、ロールアウトを一時停止したため、何も起こっていないことを確認します
Update the image to nginx:1.9.1 and check that there's nothing going on, since we paused the rollout

<details><summary>show</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.9.1
# or
kubectl edit deploy nginx
# change the image to nginx:1.9.1
kubectl rollout history deploy nginx # no new revision
```

</p>
</details>

### ロールアウトを再開し、nginx：1.9.1イメージが適用されていることを確認します
Resume the rollout and check that the nginx:1.9.1 image has been applied

<details><summary>show</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # insert the number of your latest revision
```

</p>
</details>

### デプロイと作成した水平ポッドオートスケーラーを削除します
Delete the deployment and the horizontal pod autoscaler you created

<details><summary>show</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

#Or
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

## Jobs

### 引数「perl -Mbignum = bpi -wle 'print bpi（2000）'」を使用してコマンドを実行するイメージperlを使用してジョブを作成します
Create a job with image perl that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>show</summary>
<p>

```bash
kubectl run pi --image=perl --restart=OnFailure -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

**However**, `kubectl run` for Job is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create job pi  --image=perl -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### 完了するまで待って、出力を取得します
Wait till it's done, get the output

<details><summary>show</summary>
<p>

```bash
kubectl get jobs -w # wait till 'SUCCESSFUL' is 1 (will take some time, perl image might be big)
kubectl get po # get the pod name
kubectl logs pi-**** # get the pi numbers
kubectl delete job pi
```

</p>
</details>

### コマンド 'echo hello; sleep 30; echo world'を実行するイメージbusyboxを使用してジョブを作成します
Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=OnFailure -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

**However**, `kubectl run` for Job is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### ポッドのログを追跡します（30秒待機します）
Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>show</summary>
<p>

```bash
kubectl get po # find the job pod
kubectl logs busybox-ptx58 -f # follow the logs
```

</p>
</details>

### ジョブのステータスを確認し、説明し、ログを確認します
See the status of the job, describe it and see the logs

<details><summary>show</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### ジョブを削除する
Delete the job

<details><summary>show</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### ジョブを作成しますが、実行に30秒以上かかる場合、ジョブがkubernetesによって自動的に終了されるようにします
Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute

<details><summary>show</summary>
<p>
  
```bash
kubectl create job busybox --image=busybox --dry-run -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```
  
Add job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```
</p>
</details>

### 同じジョブを作成し、5回実行します。 ステータスを確認して削除します
Create the same job, make it run 5 times, one after the other. Verify its status and delete it

<details><summary>show</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Add job.spec.completions=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Verify that it has been completed:

```bash
kubectl get job busybox -w # will take two and a half minutes
kubectl delete jobs busybox
```

</p>
</details>

### 同じジョブを作成しますが、5回並行して実行します
Create the same job, but make it run 5 parallel times

<details><summary>show</summary>
<p>

```bash
vi job.yaml
```

Add job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # add this line
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

It will take some time for the parallel jobs to finish (>= 30 seconds)

```bash
kubectl delete job busybox
```

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

### "* / 1 * * * *"のスケジュールで実行され、 'date;を書き込むイメージbusyboxでcronジョブを作成します。 HelloをKubernetesクラスターから標準出力にエコーします。
Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>show</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=OnFailure --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

**However**, `kubectl run` for CronJob is Deprecated and will be removed in a future version. What you can do is:

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### ログを確認して削除する
See its logs and delete it

<details><summary>show</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # observe that the pods have a label that mentions their 'parent' job
kubectl logs busybox-1529745840-m867r
# Bear in mind that Kubernetes will run a new job/pod for each new cron job
kubectl delete cj busybox
```

</p>
</details>

### 毎分実行され、'date; echo Hello from the Kubernetes cluster'を標準出力します。 実行に17秒以上かかる場合は、cronジョブを終了する必要があります。
Create a cron job with image busybox that runs every minutes and writes 'date; echo Hello from the Kubernetes cluster' to standard output. The cron job should be terminated if it takes more than 17 seconds to execute.

<details><summary>show</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Add job.spec.activeDeadlineSeconds=17

```bash
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 17 # add this line
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

</p>
</details>
