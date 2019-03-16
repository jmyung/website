---
title: kubectl 치트 시트
content_template: templates/concept
card:
  name: reference
  weight: 30
---

{{% capture overview %}}

참고 항목: [Kubectl 개요](/docs/reference/kubectl/overview/)와 [JsonPath 가이드](/docs/reference/kubectl/jsonpath).

이 페이지는 `kubectl` 명령어의 개요이다.

{{% /capture %}}

{{% capture body %}}

# kubectl - 치트 시트

## Kubectl 자동 완성

### BASH

```bash
source <(kubectl completion bash) # bash-completion 패키지를 먼저 설치한 후, bash의 자동 완성을 현재 쉘에 설정한다
echo "source <(kubectl completion bash)" >> ~/.bashrc # 자동 완성을 bash 쉘에 영구적으로 추가한다
```

또한, `kubectl`의 의미로 사용되는 약칭을 사용할 수 있다.

```bash
alias k=kubectl
complete -F __start_kubectl k
```

### ZSH

```bash
source <(kubectl completion zsh)  # 현재 쉘에 zsh의 자동 완성 설정
echo "if [ $commands[kubectl] ]; then source <(kubectl completion zsh); fi" >> ~/.zshrc # 자동 완성을 zsh 쉘에 영구적으로 추가한다.
```

## Kubectl 컨텍스트와 설정

`kubectl`이 통신하고 설정 정보를 수정하는 쿠버네티스 클러스터를 지정한다.

설정 파일에 대한 자세한 정보는 [kubeconfig를 이용한 클러스터 간 인증](/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) 문서를 참고하라.

```bash
kubectl config view # 병합된 kubeconfig 설정을 표시한다.

# 동시에 여러 kubeconfig 파일을 사용하고 병합된 구성을 확인한다
KUBECONFIG=~/.kube/config:~/.kube/kubconfig2 kubectl config view

# e2e 사용자의 암호를 확인한다
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'

kubectl config current-context              # 현재 컨텍스트 확인
kubectl config use-context my-cluster-name  # my-cluster-name를 기본 컨텍스트로 설정

# 기본 인증을 지원하는 새로운 클러스터를 kubeconf에 추가한다
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# 특정 사용자와 네임스페이스를 사용하는 컨텍스트 설정
kubectl config set-context gce --user=cluster-admin --namespace=foo \
  && kubectl config use-context gce
```

## 오브젝트 생성

쿠버네티스 매니페스트는 json이나 yaml로 정의된다. 파일 확장자는 `.yaml`, `.yml`, `.json` 이 사용된다.

```bash
kubectl create -f ./my-manifest.yaml           # 리소스(들) 생성
kubectl create -f ./my1.yaml -f ./my2.yaml     # 여러 파일로 부터 생성
kubectl create -f ./dir                        # dir 내 모든 매니페스트 파일에서 리소스(들) 생성
kubectl create -f https://git.io/vPieo         # url로부터 리소스(들) 생성
kubectl create deployment nginx --image=nginx  # nginx 단일 인스턴스를 시작
kubectl explain pods,svc                       # 파드와 서비스 매니페스트 문서를 조회

# stdin으로 다수의 YAML 오브젝트 생성
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# 여러 개의 키로 시크릿 생성
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF

```

## 조회, 리소스 찾기

```bash
# 기본 출력을 위한 Get 명령어
kubectl get services                          # 네임스페이스 내 모든 서비스의 목록 조회
kubectl get pods --all-namespaces             # 모든 네임스페이스 내 모든 파드의 목록 조회
kubectl get pods -o wide                      # 네임스페이스 내 모든 파드의 상세 목록 조회
kubectl get deployment my-dep                 # 특정 디플로이먼트의 목록 조회
kubectl get pods --include-uninitialized      # 초기화되지 않은 것을 포함하여 네임스페이스 내 모든 파드의 목록 조회

# 상세 출력을 위한 Describe 명령어
kubectl describe nodes my-node
kubectl describe pods my-pod

kubectl get services --sort-by=.metadata.name # Name으로 정렬된 서비스의 목록 조회

# 재시작 회수로 정렬된 파드의 목록 조회
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# app=cassandra 레이블을 가진 모든 파드의 레이블 버전 조회
kubectl get pods --selector=app=cassandra rc -o \
  jsonpath='{.items[*].metadata.labels.version}'

# 모든 워커 노드 조회 (셀렉터를 사용하여 'node-role.kubernetes.io/master'
# 으로 명명된 라벨의 결과를 제외)
kubectl get node --selector='!node-role.kubernetes.io/master'

# 네임스페이스의 모든 실행중인 파드를 조회
kubectl get pods --field-selector=status.phase=Running

# 모든 노드의 외부IP를 조회
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# 특정 RC에 속해있는 파드 이름의 목록 조회
# "jq" 명령어는 jsonpath를 사용하는 매우 복잡한 변환에 유용하다. https://stedolan.github.io/jq/ 에서 확인할 수 있다.
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# 모든 파드(또는 레이블을 지원하는 다른 쿠버네티스 오브젝트)의 레이블 조회
# 마찬가지로 "jq"를 사용
for item in $( kubectl get pod --output=name); do printf "Labels for %s\n" "$item" | grep --color -E '[^/]+$' && kubectl get "$item" --output=json | jq -r -S '.metadata.labels | to_entries | .[] | " \(.key)=\(.value)"' 2>/dev/null; printf "\n"; done

# 어떤 노드가 준비됐는지 확인
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# 파드에 의해 현재 사용되고 있는 모든 시크릿 목록 조회
kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

# 타임스템프로 정렬된 이벤트 목록 조회
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 리소스 업데이트

1.11 버전에서 `rolling-update`는 더이상 사용되지 않는다. ([CHANGELOG-1.11.md](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.11.md) 참고) 대신 `rollout`를 사용한다.

```bash
kubectl set image deployment/frontend www=image:v2               # "frontend" 디플로이먼트의 "www" 컨테이너 이미지를 업데이트하는 롤링 업데이트
kubectl rollout undo deployment/frontend                         # 이전 디플로이먼트로 롤백
kubectl rollout status -w deployment/frontend                    # 완료될 때까지 "frontend" 디플로이먼트의 롤링 업데이트 상태를 감시

# 버전 1.11 부터 사용중단
kubectl rolling-update frontend-v1 -f frontend-v2.json           # (사용중단) frontend-v1 파드의 롤링 업데이트
kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2  # (사용중단) 리소스 이름 변경과 이미지 업데이트
kubectl rolling-update frontend --image=image:v2                 # (사용중단) 프론트엔드의 파드 이미지 업데이트
kubectl rolling-update frontend-v1 frontend-v2 --rollback        # (사용중단) 진행중인 기존 롤아웃 중단

cat pod.json | kubectl replace -f -                              # std로 전달된 JSON을 기반으로 파드 교체

# 리소스를 강제 교체, 삭제후 재생성함. 이것은 서비스를 중단시킴.
kubectl replace --force -f ./pod.json

# 복제된 nginx를 위한 서비스를 생성한다. 80 포트로 서비스하고, 컨테이너는 8000 포트로 연결한다.
kubectl expose rc nginx --port=80 --target-port=8000

# 단일-컨테이너 파드의 이미지 버전 (태그)를 v4로 업데이트
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

kubectl label pods my-pod new-label=awesome                      # 레이블 추가
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # 어노테이션 추가
kubectl autoscale deployment foo --min=2 --max=10                # 디플로이먼트 "foo" 오토스케일
```

## Patching Resources

```bash
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}' # Partially update a node

# Update a container's image; spec.containers[*].name is required because it's a merge key
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# Update a container's image using a json patch with positional arrays
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# Disable a deployment livenessProbe using a json patch with positional arrays
kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'

# Add a new element to a positional array
kubectl patch sa default --type='json' -p='[{"op": "add", "path": "/secrets/1", "value": {"name": "whatever" } }]'
```

## Editing Resources
The edit any API resource in an editor.

```bash
kubectl edit svc/docker-registry                      # Edit the service named docker-registry
KUBE_EDITOR="nano" kubectl edit svc/docker-registry   # Use an alternative editor
```

## Scaling Resources

```bash
kubectl scale --replicas=3 rs/foo                                 # Scale a replicaset named 'foo' to 3
kubectl scale --replicas=3 -f foo.yaml                            # Scale a resource specified in "foo.yaml" to 3
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # If the deployment named mysql's current size is 2, scale mysql to 3
kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # Scale multiple replication controllers
```

## Deleting Resources

```bash
kubectl delete -f ./pod.json                                              # Delete a pod using the type and name specified in pod.json
kubectl delete pod,service baz foo                                        # Delete pods and services with same names "baz" and "foo"
kubectl delete pods,services -l name=myLabel                              # Delete pods and services with label name=myLabel
kubectl delete pods,services -l name=myLabel --include-uninitialized      # Delete pods and services, including uninitialized ones, with label name=myLabel
kubectl -n my-ns delete po,svc --all                                      # Delete all pods and services, including uninitialized ones, in namespace my-ns,
```

## Interacting with running Pods

```bash
kubectl logs my-pod                                 # dump pod logs (stdout)
kubectl logs my-pod --previous                      # dump pod logs (stdout) for a previous instantiation of a container
kubectl logs my-pod -c my-container                 # dump pod container logs (stdout, multi-container case)
kubectl logs my-pod -c my-container --previous      # dump pod container logs (stdout, multi-container case) for a previous instantiation of a container
kubectl logs -f my-pod                              # stream pod logs (stdout)
kubectl logs -f my-pod -c my-container              # stream pod container logs (stdout, multi-container case)
kubectl run -i --tty busybox --image=busybox -- sh  # Run pod as interactive shell
kubectl attach my-pod -i                            # Attach to Running Container
kubectl port-forward my-pod 5000:6000               # Listen on port 5000 on the local machine and forward to port 6000 on my-pod
kubectl exec my-pod -- ls /                         # Run command in existing pod (1 container case)
kubectl exec my-pod -c my-container -- ls /         # Run command in existing pod (multi-container case)
kubectl top pod POD_NAME --containers               # Show metrics for a given pod and its containers
```

## Interacting with Nodes and Cluster

```bash
kubectl cordon my-node                                                # Mark my-node as unschedulable
kubectl drain my-node                                                 # Drain my-node in preparation for maintenance
kubectl uncordon my-node                                              # Mark my-node as schedulable
kubectl top node my-node                                              # Show metrics for a given node
kubectl cluster-info                                                  # Display addresses of the master and services
kubectl cluster-info dump                                             # Dump current cluster state to stdout
kubectl cluster-info dump --output-directory=/path/to/cluster-state   # Dump current cluster state to /path/to/cluster-state

# If a taint with that key and effect already exists, its value is replaced as specified.
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

### Resource types

List all supported resource types along with their shortnames, [API group](/docs/concepts/overview/kubernetes-api/#api-groups), whether they are [namespaced](/docs/concepts/overview/working-with-objects/namespaces), and [Kind](/docs/concepts/overview/working-with-objects/kubernetes-objects):

```bash
kubectl api-resources
```

Other operations for exploring API resources:

```bash
kubectl api-resources --namespaced=true      # All namespaced resources
kubectl api-resources --namespaced=false     # All non-namespaced resources
kubectl api-resources -o name                # All resources with simple output (just the resource name)
kubectl api-resources -o wide                # All resources with expanded (aka "wide") output
kubectl api-resources --verbs=list,get       # All resources that support the "list" and "get" request verbs
kubectl api-resources --api-group=extensions # All resources in the "extensions" API group
```

### Formatting output

To output details to your terminal window in a specific format, you can add either the `-o` or `--output` flags to a supported `kubectl` command.

Output format | Description
--------------| -----------
`-o=custom-columns=<spec>` | Print a table using a comma separated list of custom columns
`-o=custom-columns-file=<filename>` | Print a table using the custom columns template in the `<filename>` file
`-o=json`     | Output a JSON formatted API object
`-o=jsonpath=<template>` | Print the fields defined in a [jsonpath](/docs/reference/kubectl/jsonpath) expression
`-o=jsonpath-file=<filename>` | Print the fields defined by the [jsonpath](/docs/reference/kubectl/jsonpath) expression in the `<filename>` file
`-o=name`     | Print only the resource name and nothing else
`-o=wide`     | Output in the plain-text format with any additional information, and for pods, the node name is included
`-o=yaml`     | Output a YAML formatted API object

### Kubectl output verbosity and debugging

Kubectl verbosity is controlled with the `-v` or `--v` flags followed by an integer representing the log level. General Kubernetes logging conventions and the associated log levels are described [here](https://github.com/kubernetes/community/blob/master/contributors/devel/logging.md).

Verbosity | Description
--------------| -----------
`--v=0` | Generally useful for this to ALWAYS be visible to an operator.
`--v=1` | A reasonable default log level if you don't want verbosity.
`--v=2` | Useful steady state information about the service and important log messages that may correlate to significant changes in the system. This is the recommended default log level for most systems.
`--v=3` | Extended information about changes.
`--v=4` | Debug level verbosity.
`--v=6` | Display requested resources.
`--v=7` | Display HTTP request headers.
`--v=8` | Display HTTP request contents.
`--v=9` | Display HTTP request contents without truncation of contents.

{{% /capture %}}

{{% capture whatsnext %}}

* Learn more about [Overview of kubectl](/docs/reference/kubectl/overview/).

* See [kubectl](/docs/reference/kubectl/kubectl/) options.

* Also [kubectl Usage Conventions](/docs/reference/kubectl/conventions/) to understand how to use it in reusable scripts.

* See more community [kubectl cheatsheets](https://github.com/dennyzhang/cheatsheet-kubernetes-A4).

{{% /capture %}}
