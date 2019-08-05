---
title: 서비스
feature:
  title: Service discovery and load balancing
  description: >
    No need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

content_template: templates/concept
weight: 10
---




{{% capture overview %}}

{{< glossary_definition term_id="service" length="short" >}}

쿠버네티스를 사용하면 익숙하지 않은 서비스 검색 메커니즘을 사용하기 위해 애플리케이션을 수정할 필요가 없다.
쿠버네티스는 파드에게 고유한 IP 주소와 파드 집합에 대한 단일 DNS 명을 부여하고,
그것들간에 로드-밸런스를 수행할 수 있다.

{{% /capture %}}

{{% capture body %}}

## 동기

쿠버네티스 {{< glossary_tooltip term_id="pod" text="파드" >}}는 수명이 있다.
파드는 생성되고, 소멸된 후 부활하지 않는다.
만약 앱을 실행하기 위해 {{< glossary_tooltip term_id="deployment" text="디플로이먼트" >}}를 사용한다면,
동적으로 파드를 생성하고 제거할 수 있다.

각 파드는 고유한 IP 주소를 갖지만, 디플로이먼트에서는
한 시점에 실행되는 파드 집합이
잠시후 실행되는 해당 파드 집합과 다를 수 있다.

이는 다음과 같은 문제를 야기한다. (“백엔드”라 불리는) 일부 파드 집합이
클러스터의 (“프론트엔드”라 불리는) 다른 파드 에 기능을 제공하는 경우,
프론트엔드가 워크로드의 백엔드를 사용하기 위해,
프론트엔드가 어떻게 연결할 IP 주소를 찾아서 추적할 수 있는가?

_서비스_ 로 들어가보자.

## 서비스 리소스 {#서비스-리소스}

쿠버네티스에서 서비스는 파드의 논리적 집합과 그것들을 액세스할 수 있는
정책을 정의하는 추상화이다. (때로는 이 패턴을
마이크로-서비스라고 한다.) 서비스가 대상으로 하는 파드 집합은 일반적으로
{{< glossary_tooltip text="셀렉터" term_id="selector" >}}가 결정한다.
(셀렉터가 _없는_ 서비스가 필요한 이유는 [아래](#셀렉터가-없는-서비스)를
참조한다).

예를 들어, 3개의 레플리카로 실행되는 스테이트리스 이미지-처리 백엔드를
생각해보자. 이러한 레플리카는 대체 가능하고&mdash;프론트 엔드는 그것들이 사용하는 백엔드를
신경 쓰지 않는다. 백엔드 세트를 구성하는 실제 파드는 변경될 수 있지만,
프론트엔드 클라이언트는 이를 인식할 필요가 없으며, 백엔드 세트 자체를 추적해야 할 필요도
없다.

서비스 추상화는 이러한 디커플링을 가능하게 한다.

### 클라우드-네이티브 서비스 검색

애플리케이션에서 서비스 검색을 위해 쿠버네티스 API를 사용할 수 있는 경우,
서비스 내 파드 세트가 변경 될 때마다 업데이트되는 엔드포인트를 {{< glossary_tooltip text="API 서버" term_id="kube-apiserver" >}}에
질의할 수 ​​있다.

네이티브 애플리케이션이 아닌 (non-native applications) 경우, 쿠버네티스는 애플리케이션과 백엔드 파드 사이에 네트워크 포트 또는 로드
밸런서를 배치 할 수 있는 방법을 제공한다.

## 서비스 정의

쿠버네티스의 서비스는 파드와 비슷한 REST 객체이다. 모든 REST 객체와
마찬가지로, 서비스 정의를 API 서버에 `POST`하여
새 인스턴스를 생성할 수 있다.

예를 들어, 각각 TCP 포트 9376에서 수신하고
`app=MyApp` 레이블을 가지고 있는 파드 세트가 있다고 가정해 보자.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

이 명세는 “my-service”라는 새로운 서비스 오브젝트를 생성하고,
`app=MyApp` 레이블을 가진 파드의 TCP 9376 포트를 대상으로 한다.

쿠버네티스는 이 서비스에 서비스 프록시가 사용하는 IP 주소 ("cluster IP"라고도 함)
를 할당한다.
(이하 [가상 IP와 서비스 프록시](#가상-IP와-서비스-프록시) 참고)

서비스 셀렉터의 컨트롤러는 셀렉터와 일치하는 파드를 지속적으로 검색하고,
“my-service”라는 엔드포인트 오브젝트에 대한
모든 업데이트를 POST한다.

{{< note >}}
서비스는 _모든_ 수신 `port`를 `targetPort`에 매핑할 수 있다. 기본적으로 그리고
편의상, `targetPort`는 `port`
필드와 같은 값으로 설정된다.
{{< /note >}}

파드의 포트 정의에는 이름이 있고, 서비스의 `targetPort` 속성에서 이 이름을
참조할 수 있다. 이것은 다른 포트 번호를 통한 가용한 동일 네트워크 프로토콜이 있고,
단일 구성 이름을 사용하는 서비스 내에
혼합된 파드가 존재해도 가능하다.
이를 통해 서비스를 배포하고 진전시키는데 많은 유연성을 제공한다.
예를 들어, 클라이언트를 망가뜨리지 않고, 백엔드 소프트웨어의 다음
버전에서 파드가 노출시키는 포트 번호를 변경할 수 있다.

서비스의 기본 프로토콜은 TCP이다. 다른
[지원 프로토콜](#지원-프로토콜)을 사용할 수도 있다.

많은 서비스가 하나 이상의 포트를 노출해야 하기 때문에, 쿠버네티스는 서비스 오브젝트에서 다중
포트 정의를 지원한다.
각 포트는 동일한 `프로토콜` 또는 다른 프로토콜로 정의될 수 있다.

### 셀렉터가 없는 서비스

Services most commonly abstract access to Kubernetes Pods, but they can also
abstract other kinds of backends.
For example:

  * You want to have an external database cluster in production, but in your
    test environment you use your own databases.
  * You want to point your Service to a Service in a different
    {{< glossary_tooltip term_id="namespace" >}} or on another cluster.
  * You are migrating a workload to Kubernetes. Whilst evaluating the approach,
    you run only a proportion of your backends in Kubernetes.

In any of these scenarios you can define a Service _without_ a Pod selector.
For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

Because this Service has no selector, the corresponding Endpoint object is *not*
created automatically. You can manually map the Service to the network address and port
where it's running, by adding an Endpoint object manually:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 192.0.2.42
    ports:
      - port: 9376
```

{{< note >}}
The endpoint IPs _must not_ be: loopback (127.0.0.0/8 for IPv4, ::1/128 for IPv6), or
link-local (169.254.0.0/16 and 224.0.0.0/24 for IPv4, fe80::/64 for IPv6).

Endpoint IP addresses cannot be the cluster IPs of other Kubernetes Services,
because {{< glossary_tooltip term_id="kube-proxy" >}} doesn't support virtual IPs
as a destination.
{{< /note >}}

Accessing a Service without a selector works the same as if it had a selector.
In the example above, traffic is routed to the single endpoint defined in
the YAML: `192.0.2.42:9376` (TCP).

An ExternalName Service is a special case of Service that does not have
selectors and uses DNS names instead. For more information, see the
[ExternalName](#externalname) section later in this document.

## 가상 IP와 서비스 프록시

Every node in a Kubernetes cluster runs a `kube-proxy`. `kube-proxy` is
responsible for implementing a form of virtual IP for `Services` of type other
than [`ExternalName`](#externalname).

### Why not use round-robin DNS?

A question that pops up every now and then is why Kubernetes relies on
proxying to forward inbound traffic to backends. What about other
approaches? For example, would it be possible to configure DNS records that
have multiple A values (or AAAA for IPv6), and rely on round-robin name
resolution?

There are a few reasons for using proxying for Services:

 * There is a long history of DNS implementations not respecting record TTLs,
   and caching the results of name lookups after they should have expired.
 * Some apps do DNS lookups only once and cache the results indefinitely.
 * Even if apps and libraries did proper re-resolution, the low or zero TTLs
   on the DNS records could impose a high load on DNS that then becomes
   difficult to manage.

### Version compatibility

Since Kubernetes v1.0 you have been able to use the
[userspace proxy mode](#proxy-mode-userspace).
Kubernetes v1.1 added iptables mode proxying, and in Kubernetes v1.2 the
iptables mode for kube-proxy became the default.
Kubernetes v1.8 added ipvs proxy mode.

### User space proxy mode {#proxy-mode-userspace}

In this mode, kube-proxy watches the Kubernetes master for the addition and
removal of Service and Endpoint objects. For each Service it opens a
port (randomly chosen) on the local node.  Any connections to this "proxy port"
is proxied to one of the Service's backend Pods (as reported via
Endpoints). kube-proxy takes the `SessionAffinity` setting of the Service into
account when deciding which backend Pod to use.

Lastly, the user-space proxy installs iptables rules which capture traffic to
the Service's `clusterIP` (which is virtual) and `port`. The rules
redirect that traffic to the proxy port which proxies the backend Pod.

By default, kube-proxy in userspace mode chooses a backend via a round-robin algorithm.

![Services overview diagram for userspace proxy](/images/docs/services-userspace-overview.svg)

### `iptables` proxy mode {#proxy-mode-iptables}

In this mode, kube-proxy watches the Kubernetes control plane for the addition and
removal of Service and Endpoint objects. For each Service, it installs
iptables rules, which capture traffic to the Service's `clusterIP` and `port`,
and redirect that traffic to one of the Service's
backend sets.  For each Endpoint object, it installs iptables rules which
select a backend Pod.

By default, kube-proxy in iptables mode chooses a backend at random.

Using iptables to handle traffic has a lower system overhead, because traffic
is handled by Linux netfilter without the need to switch between userspace and the
kernel space. This approach is also likely to be more reliable.

If kube-proxy is running in iptables mode and the first Pod that's selected
does not respond, the connection fails. This is different from userspace
mode: in that scenario, kube-proxy would detect that the connection to the first
Pod had failed and would automatically retry with a different backend Pod.

You can use Pod [readiness probes](/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
to verify that backend Pods are working OK, so that kube-proxy in iptables mode
only sees backends that test out as healthy. Doing this means you avoid
having traffic sent via kube-proxy to a Pod that's known to have failed.

![Services overview diagram for iptables proxy](/images/docs/services-iptables-overview.svg)

### IPVS proxy mode {#proxy-mode-ipvs}

{{< feature-state for_k8s_version="v1.11" state="stable" >}}

In `ipvs` mode, kube-proxy watches Kubernetes Services and Endpoints,
calls `netlink` interface to create IPVS rules accordingly and synchronizes
IPVS rules with Kubernetes Services and Endpoints periodically.
This control loop ensures that IPVS status matches the desired
state.
When accessing a Service, IPVS directs traffic to one of the backend Pods.

The IPVS proxy mode is based on netfilter hook function that is similar to
iptables mode, but uses hash table as the underlying data structure and works
in the kernel space.
That means kube-proxy in IPVS mode redirects traffic with a lower latency than
kube-proxy in iptables mode, with much better performance when synchronising
proxy rules. Compared to the other proxy modes, IPVS mode also supports a
higher throughput of network traffic.

IPVS provides more options for balancing traffic to backend Pods;
these are:

- `rr`: round-robin
- `lc`: least connection (smallest number of open connections)
- `dh`: destination hashing
- `sh`: source hashing
- `sed`: shortest expected delay
- `nq`: never queue

{{< note >}}
To run kube-proxy in IPVS mode, you must make the IPVS Linux available on
the node before you starting kube-proxy.

When kube-proxy starts in IPVS proxy mode, it verifies whether IPVS
kernel modules are available. If the IPVS kernel modules are not detected, then kube-proxy
falls back to running in iptables proxy mode.
{{< /note >}}

![Services overview diagram for IPVS proxy](/images/docs/services-ipvs-overview.svg)

In these proxy models, the traffic bound for the Service’s IP:Port is
proxied to an appropriate backend without the clients knowing anything
about Kubernetes or Services or Pods.

If you want to make sure that connections from a particular client
are passed to the same Pod each time, you can select the session affinity based
the on client's IP addresses by setting `service.spec.sessionAffinity` to "ClientIP"
(the default is "None").
You can also set the maximum session sticky time by setting
`service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` appropriately.
(the default value is 10800, which works out to be 3 hours).

## Multi-Port Services

For some Services, you need to expose more than one port.
Kubernetes lets you configure multiple port definitions on a Service object.
When using multiple ports for a Service, you must give all of your ports names
so that these are unambiguous.
For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  - name: https
    protocol: TCP
    port: 443
    targetPort: 9377
```

{{< note >}}
As with Kubernetes {{< glossary_tooltip term_id="name" text="names">}} in general, names for ports
must only contain lowercase alphanumeric characters and `-`. Port names must
also start and end with an alphanumeric character.

For example, the names `123-abc` and `web` are valid, but `123_abc` and `-web` are not.
{{< /note >}}

## Choosing your own IP address

You can specify your own cluster IP address as part of a `Service` creation
request.  To do this, set the `.spec.clusterIP` field. For example, if you
already have an existing DNS entry that you wish to reuse, or legacy systems
that are configured for a specific IP address and difficult to re-configure.

The IP address that you choose must be a valid IPv4 or IPv6 address from within the
`service-cluster-ip-range` CIDR range that is configured for the API server.
If you try to create a Service with an invalid clusterIP address value, the API
server will returns a 422 HTTP status code to indicate that there's a problem.

## Discovering services

Kubernetes supports 2 primary modes of finding a Service - environment
variables and DNS.

### Environment variables

When a Pod is run on a Node, the kubelet adds a set of environment variables
for each active Service.  It supports both [Docker links
compatible](https://docs.docker.com/userguide/dockerlinks/) variables (see
[makeLinkVariables](http://releases.k8s.io/{{< param "githubbranch" >}}/pkg/kubelet/envvars/envvars.go#L49))
and simpler `{SVCNAME}_SERVICE_HOST` and `{SVCNAME}_SERVICE_PORT` variables,
where the Service name is upper-cased and dashes are converted to underscores.

For example, the Service `"redis-master"` which exposes TCP port 6379 and has been
allocated cluster IP address 10.0.0.11, produces the following environment
variables:

```shell
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

{{< note >}}
When you have a Pod that needs to access a Service, and you are using
the environment variable method to publish the port and cluster IP to the client
Pods, you must create the Service *before* the client Pods come into existence.
Otherwise, those client Pods won't have their environment variables populated.

If you only use DNS to discover the cluster IP for a Service, you don't need to
worry about this ordering issue.
{{< /note >}}

### DNS

You can (and almost always should) set up a DNS service for your Kubernetes
cluster using an [add-on](/docs/concepts/cluster-administration/addons/).

A cluster-aware DNS server, such as CoreDNS, watches the Kubernetes API for new
Services and creates a set of DNS records for each one.  If DNS has been enabled
throughout your cluster then all Pods should automatically be able to resolve
Services by their DNS name.

For example, if you have a Service called `"my-service"` in a Kubernetes
Namespace `"my-ns"`, the control plane and the DNS Service acting together
create a DNS record for `"my-service.my-ns"`. Pods in the `"my-ns"` Namespace
should be able to find it by simply doing a name lookup for `my-service`
(`"my-service.my-ns"` would also work).

Pods in other Namespaces must qualify the name as `my-service.my-ns`. These names
will resolve to the cluster IP assigned for the Service.

Kubernetes also supports DNS SRV (Service) records for named ports.  If the
`"my-service.my-ns"` Service has a port named `"http"` with protocol set to
`TCP`, you can do a DNS SRV query for `_http._tcp.my-service.my-ns` to discover
the port number for `"http"`, as well as the IP address.

The Kubernetes DNS server is the only way to access `ExternalName` Services.
You can find more information about `ExternalName` resolution in
[DNS Pods and Services](/docs/concepts/services-networking/dns-pod-service/).

## Headless Services

Sometimes you don't need load-balancing and a single Service IP.  In
this case, you can create what are termed “headless” Services, by explicitly
specifying `"None"` for the cluster IP (`.spec.clusterIP`).

You can use a headless Service to interface with other service discovery mechanisms,
without being tied to Kubernetes' implementation. For example, you could implement
a custom {{< glossary_tooltip term_id="operator-pattern" text="Operator" >}} upon
this API.

For such `Services`, a cluster IP is not allocated, kube-proxy does not handle
these Services, and there is no load balancing or proxying done by the platform
for them. How DNS is automatically configured depends on whether the Service has
selectors defined.

### With selectors

For headless Services that define selectors, the endpoints controller creates
`Endpoints` records in the API, and modifies the DNS configuration to return
records (addresses) that point directly to the `Pods` backing the `Service`.

### Without selectors

For headless Services that do not define selectors, the endpoints controller does
not create `Endpoints` records. However, the DNS system looks for and configures
either:

  * CNAME records for [`ExternalName`](#externalname)-type Services.
  * A records for any `Endpoints` that share a name with the Service, for all
    other types.

## Publishing Services (ServiceTypes) {#publishing-services-service-types}

For some parts of your application (for example, frontends) you may want to expose a
Service onto an external IP address, that's outside of your cluster.

Kubernetes `ServiceTypes` allow you to specify what kind of Service you want.
The default is `ClusterIP`.

`Type` values and their behaviors are:

   * `ClusterIP`: Exposes the Service on a cluster-internal IP. Choosing this value
     makes the Service only reachable from within the cluster. This is the
     default `ServiceType`.
   * [`NodePort`](#nodeport): Exposes the Service on each Node's IP at a static port
     (the `NodePort`). A `ClusterIP` Service, to which the `NodePort` Service
     routes, is automatically created.  You'll be able to contact the `NodePort` Service,
     from outside the cluster,
     by requesting `<NodeIP>:<NodePort>`.
   * [`LoadBalancer`](#loadbalancer): Exposes the Service externally using a cloud
     provider's load balancer. `NodePort` and `ClusterIP` Services, to which the external
     load balancer routes, are automatically created.
   * [`ExternalName`](#externalname): Maps the Service to the contents of the
     `externalName` field (e.g. `foo.bar.example.com`), by returning a `CNAME` record

     with its value. No proxying of any kind is set up.
     {{< note >}}
     You need CoreDNS version 1.7 or higher to use the `ExternalName` type.
     {{< /note >}}

You can also use [Ingress](/docs/concepts/services-networking/ingress/) to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.

### Type NodePort {#nodeport}

If you set the `type` field to `NodePort`, the Kubernetes control plane
allocates a port from a range specified by `--service-node-port-range` flag (default: 30000-32767).
Each node proxies that port (the same port number on every Node) into your Service.
Your Service reports the allocated port in its `.spec.ports[*].nodePort` field.


If you want to specify particular IP(s) to proxy the port, you can set the `--nodeport-addresses` flag in kube-proxy to particular IP block(s); this is supported since Kubernetes v1.10.
This flag takes a comma-delimited list of IP blocks (e.g. 10.0.0.0/8, 192.0.2.0/25) to specify IP address ranges that kube-proxy should consider as local to this node.

For example, if you start kube-proxy with the `--nodeport-addresses=127.0.0.0/8` flag, kube-proxy only selects the loopback interface for NodePort Services. The default for `--nodeport-addresses` is an empty list. This means that kube-proxy should consider all available network interfaces for NodePort. (That's also compatible with earlier Kubernetes releases).

If you want a specific port number, you can specify a value in the `nodePort`
field. The control plane will either allocate you that port or report that
the API transaction failed.
This means that you need to take care about possible port collisions yourself.
You also have to use a valid port number, one that's inside the range configured
for NodePort use.

Using a NodePort gives you the freedom to set up your own load balancing solution,
to configure environments that are not fully supported by Kubernetes, or even
to just expose one or more nodes' IPs directly.

Note that this Service is visible as `<NodeIP>:spec.ports[*].nodePort`
and `.spec.clusterIP:spec.ports[*].port`. (If the `--nodeport-addresses` flag in kube-proxy is set, <NodeIP> would be filtered NodeIP(s).)

### Type LoadBalancer {#loadbalancer}

On cloud providers which support external load balancers, setting the `type`
field to `LoadBalancer` provisions a load balancer for your Service.
The actual creation of the load balancer happens asynchronously, and
information about the provisioned balancer is published in the Service's
`.status.loadBalancer` field.
For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
    - ip: 146.148.47.155
```

Traffic from the external load balancer is directed at the backend Pods. The cloud provider decides how it is load balanced.


Some cloud providers allow you to specify the `loadBalancerIP`. In those cases, the load-balancer is created
with the user-specified `loadBalancerIP`. If the `loadBalancerIP` field is not specified,
the loadBalancer is set up with an ephemeral IP address. If you specify a `loadBalancerIP`
but your cloud provider does not support the feature, the `loadbalancerIP` field that you
set is ignored.

{{< note >}}
If you're using SCTP, see the [caveat](#caveat-sctp-loadbalancer-service-type) below about the
`LoadBalancer` Service type.
{{< /note >}}

{{< note >}}

On **Azure**, if you want to use a user-specified public type `loadBalancerIP`, you first need
to create a static type public IP address resource. This public IP address resource should
be in the same resource group of the other automatically created resources of the cluster.
For example, `MC_myResourceGroup_myAKSCluster_eastus`.

Specify the assigned IP address as loadBalancerIP. Ensure that you have updated the securityGroupName in the cloud provider configuration file. For information about troubleshooting `CreatingLoadBalancerFailed` permission issues see, [Use a static IP address with the Azure Kubernetes Service (AKS) load balancer](https://docs.microsoft.com/en-us/azure/aks/static-ip) or [CreatingLoadBalancerFailed on AKS cluster with advanced networking](https://github.com/Azure/AKS/issues/357).

{{< /note >}}

#### Internal load balancer
In a mixed environment it is sometimes necessary to route traffic from Services inside the same
(virtual) network address block.

In a split-horizon DNS environment you would need two Services to be able to route both external and internal traffic to your endpoints.

You can achieve this by adding one the following annotations to a Service.
The annotation to add depends on the cloud Service provider you're using.

{{< tabs name="service_tabs" >}}
{{% tab name="Default" %}}
Select one of the tabs.
{{% /tab %}}
{{% tab name="GCP" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        cloud.google.com/load-balancer-type: "Internal"
[...]
```
Use `cloud.google.com/load-balancer-type: "internal"` for masters with version 1.7.0 to 1.7.3.
For more information, see the [docs](https://cloud.google.com/kubernetes-engine/docs/internal-load-balancing).
{{% /tab %}}
{{% tab name="AWS" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: 0.0.0.0/0
[...]
```
{{% /tab %}}
{{% tab name="Azure" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
[...]
```
{{% /tab %}}
{{% tab name="OpenStack" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/openstack-internal-load-balancer: "true"
[...]
```
{{% /tab %}}
{{% tab name="Baidu Cloud" %}}
```yaml
[...]
metadata:
    name: my-service
    annotations:
        service.beta.kubernetes.io/cce-load-balancer-internal-vpc: "true"
[...]
```
{{% /tab %}}
{{< /tabs >}}


#### TLS support on AWS {#ssl-support-on-aws}

For partial TLS / SSL support on clusters running on AWS, you can add three
annotations to a `LoadBalancer` service:

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012
```

The first specifies the ARN of the certificate to use. It can be either a
certificate from a third party issuer that was uploaded to IAM or one created
within AWS Certificate Manager.

```yaml
metadata:
  name: my-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: (https|http|ssl|tcp)
```

The second annotation specifies which protocol a Pod speaks. For HTTPS and
SSL, the ELB expects the Pod to authenticate itself over the encrypted
connection, using a certificate.

HTTP and HTTPS selects layer 7 proxying: the ELB terminates
the connection with the user, parse headers and inject the `X-Forwarded-For`
header with the user's IP address (Pods only see the IP address of the
ELB at the other end of its connection) when forwarding requests.

TCP and SSL selects layer 4 proxying: the ELB forwards traffic without
modifying the headers.

In a mixed-use environment where some ports are secured and others are left unencrypted,
you can use the following annotations:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
        service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443,8443"
```

In the above example, if the Service contained three ports, `80`, `443`, and
`8443`, then `443` and `8443` would use the SSL certificate, but `80` would just
be proxied HTTP.

From Kubernetes v1.9 onwrds you can use [predefined AWS SSL policies](http://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-security-policy-table.html) with HTTPS or SSL listeners for your Services.
To see which policies are available for use, you can the `aws` command line tool:

```bash
aws elb describe-load-balancer-policies --query 'PolicyDescriptions[].PolicyName'
```

You can then specify any one of those policies using the
"`service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy`"
annotation; for example:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy: "ELBSecurityPolicy-TLS-1-2-2017-01"
```

#### PROXY protocol support on AWS

To enable [PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt)
support for clusters running on AWS, you can use the following service
annotation:

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-proxy-protocol: "*"
```

Since version 1.3.0, the use of this annotation applies to all ports proxied by the ELB
and cannot be configured otherwise.

#### ELB Access Logs on AWS

There are several annotations to manage access logs for ELB Services on AWS.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-enabled`
controls whether access logs are enabled.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval`
controls the interval in minutes for publishing the access logs. You can specify
an interval of either 5 or 60 minutes.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name`
controls the name of the Amazon S3 bucket where load balancer access logs are
stored.

The annotation `service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix`
specifies the logical hierarchy you created for your Amazon S3 bucket.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
        # Specifies whether access logs are enabled for the load balancer
        service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
        # The interval for publishing the access logs. You can specify an interval of either 5 or 60 (minutes).
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
        # The name of the Amazon S3 bucket where the access logs are stored
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
        # The logical hierarchy you created for your Amazon S3 bucket, for example `my-bucket-prefix/prod`
```

#### Connection Draining on AWS

Connection draining for Classic ELBs can be managed with the annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled` set
to the value of `"true"`. The annotation
`service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout` can
also be used to set maximum time, in seconds, to keep the existing connections open before deregistering the instances.


```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-enabled: "true"
        service.beta.kubernetes.io/aws-load-balancer-connection-draining-timeout: "60"
```

#### Other ELB annotations

There are other annotations to manage Classic Elastic Load Balancers that are described below.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"
        # The time, in seconds, that the connection is allowed to be idle (no data has been sent over the connection) before it is closed by the load balancer

        service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
        # Specifies whether cross-zone load balancing is enabled for the load balancer

        service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags: "environment=prod,owner=devops"
        # A comma-separated list of key-value pairs which will be recorded as
        # additional tags in the ELB.

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: ""
        # The number of successive successful health checks required for a backend to
        # be considered healthy for traffic. Defaults to 2, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
        # The number of unsuccessful health checks required for a backend to be
        # considered unhealthy for traffic. Defaults to 6, must be between 2 and 10

        service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "20"
        # The approximate interval, in seconds, between health checks of an
        # individual instance. Defaults to 10, must be between 5 and 300
        service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
        # The amount of time, in seconds, during which no response means a failed
        # health check. This value must be less than the service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval
        # value. Defaults to 5, must be between 2 and 60

        service.beta.kubernetes.io/aws-load-balancer-extra-security-groups: "sg-53fae93f,sg-42efd82e"
        # A list of additional security groups to be added to the ELB
```

#### Network Load Balancer support on AWS [alpha] {#aws-nlb-support}

{{< warning >}}
This is an alpha feature and is not yet recommended for production clusters.
{{< /warning >}}

Starting from Kubernetes v1.9.0, you can use AWS Network Load Balancer (NLB) with Services. To
use a Network Load Balancer on AWS, use the annotation `service.beta.kubernetes.io/aws-load-balancer-type`
with the value set to `nlb`.

```yaml
    metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

{{< note >}}
NLB only works with certain instance classes; see the [AWS documentation](http://docs.aws.amazon.com/elasticloadbalancing/latest/network/target-group-register-targets.html#register-deregister-targets)
on Elastic Load Balancing for a list of supported instance types.
{{< /note >}}

Unlike Classic Elastic Load Balancers, Network Load Balancers (NLBs) forward the
client's IP address through to the node. If a Service's `.spec.externalTrafficPolicy`
is set to `Cluster`, the client's IP address is not propagated to the end
Pods.

By setting `.spec.externalTrafficPolicy` to `Local`, the client IP addresses is
propagated to the end Pods, but this could result in uneven distribution of
traffic. Nodes without any Pods for a particular LoadBalancer Service will fail
the NLB Target Group's health check on the auto-assigned
`.spec.healthCheckNodePort` and not receive any traffic.

In order to achieve even traffic, either use a DaemonSet, or specify a
[pod anti-affinity](/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)
to not locate on the same node.

You can also use NLB Services with the [internal load balancer](/docs/concepts/services-networking/service/#internal-load-balancer)
annotation.

In order for client traffic to reach instances behind an NLB, the Node security
groups are modified with the following IP rules:

| Rule | Protocol | Port(s) | IpRange(s) | IpRange Description |
|------|----------|---------|------------|---------------------|
| Health Check | TCP | NodePort(s) (`.spec.healthCheckNodePort` for `.spec.externalTrafficPolicy = Local`) | VPC CIDR | kubernetes.io/rule/nlb/health=\<loadBalancerName\> |
| Client Traffic | TCP | NodePort(s) | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/client=\<loadBalancerName\> |
| MTU Discovery | ICMP | 3,4 | `.spec.loadBalancerSourceRanges` (defaults to `0.0.0.0/0`) | kubernetes.io/rule/nlb/mtu=\<loadBalancerName\> |

In order to limit which client IP's can access the Network Load Balancer,
specify `loadBalancerSourceRanges`.

```yaml
spec:
  loadBalancerSourceRanges:
  - "143.231.0.0/16"
```

{{< note >}}
If `.spec.loadBalancerSourceRanges` is not set, Kubernetes
allows traffic from `0.0.0.0/0` to the Node Security Group(s). If nodes have
public IP addresses, be aware that non-NLB traffic can also reach all instances
in those modified security groups.

{{< /note >}}

### Type ExternalName {#externalname}

Services of type ExternalName map a Service to a DNS name, not to a typical selector such as
`my-service` or `cassandra`. You specify these Services with the `spec.externalName` parameter.

This Service definition, for example, maps
the `my-service` Service in the `prod` namespace to `my.database.example.com`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
{{< note >}}
ExternalName accepts an IPv4 address string, but as a DNS names comprised of digits, not as an IP address. ExternalNames that resemble IPv4 addresses are not resolved by CoreDNS or ingress-nginx because ExternalName
is intended to specify a canonical DNS name. To hardcode an IP address, consider using
[headless Services](#headless-services).
{{< /note >}}

When looking up the host `my-service.prod.svc.cluster.local`, the cluster DNS Service
returns a `CNAME` record with the value `my.database.example.com`. Accessing
`my-service` works in the same way as other Services but with the crucial
difference that redirection happens at the DNS level rather than via proxying or
forwarding. Should you later decide to move your database into your cluster, you
can start its Pods, add appropriate selectors or endpoints, and change the
Service's `type`.


{{< note >}}
This section is indebted to the [Kubernetes Tips - Part
1](https://akomljen.com/kubernetes-tips-part-1/) blog post from [Alen Komljen](https://akomljen.com/).
{{< /note >}}

### External IPs

If there are external IPs that route to one or more cluster nodes, Kubernetes Services can be exposed on those
`externalIPs`. Traffic that ingresses into the cluster with the external IP (as destination IP), on the Service port,
will be routed to one of the Service endpoints. `externalIPs` are not managed by Kubernetes and are the responsibility
of the cluster administrator.

In the Service spec, `externalIPs` can be specified along with any of the `ServiceTypes`.
In the example below, "`my-service`" can be accessed by clients on "`80.11.12.10:80`" (`externalIP:port`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 9376
  externalIPs:
  - 80.11.12.10
```

## Shortcomings

Using the userspace proxy for VIPs, work at small to medium scale, but will
not scale to very large clusters with thousands of Services.  The [original
design proposal for portals](http://issue.k8s.io/1107) has more details on
this.

Using the userspace proxy obscures the source IP address of a packet accessing
a Service.
This makes some kinds of network filtering (firewalling) impossible.  The iptables
proxy mode does not
obscure in-cluster source IPs, but it does still impact clients coming through
a load balancer or node-port.

The `Type` field is designed as nested functionality - each level adds to the
previous.  This is not strictly required on all cloud providers (e.g. Google Compute Engine does
not need to allocate a `NodePort` to make `LoadBalancer` work, but AWS does)
but the current API requires it.

## Virtual IP implementation {#the-gory-details-of-virtual-ips}

The previous information should be sufficient for many people who just want to
use Services.  However, there is a lot going on behind the scenes that may be
worth understanding.

### Avoiding collisions

One of the primary philosophies of Kubernetes is that you should not be
exposed to situations that could cause your actions to fail through no fault
of your own. For the design of the Service resource, this means not making
you choose your own port number for a if that choice might collide with
someone else's choice.  That is an isolation failure.

In order to allow you to choose a port number for your Services, we must
ensure that no two Services can collide. Kubernetes does that by allocating each
Service its own IP address.

To ensure each Service receives a unique IP, an internal allocator atomically
updates a global allocation map in {{< glossary_tooltip term_id="etcd" >}}
prior to creating each Service. The map object must exist in the registry for
Services to get IP address assignments, otherwise creations will
fail with a message indicating an IP address could not be allocated.

In the control plane, a background controller is responsible for creating that
map (needed to support migrating from older versions of Kubernetes that used
in-memory locking). Kubernetes also uses controllers to checking for invalid
assignments (eg due to administrator intervention) and for cleaning up allocated
IP addresses that are no longer used by any Services.

### Service IP addresses {#ips-and-vips}

Unlike Pod IP addresses, which actually route to a fixed destination,
Service IPs are not actually answered by a single host.  Instead, kube-proxy
uses iptables (packet processing logic in Linux) to define _virtual_ IP addresses
which are transparently redirected as needed.  When clients connect to the
VIP, their traffic is automatically transported to an appropriate endpoint.
The environment variables and DNS for Services are actually populated in
terms of the Service's virtual IP address (and port).

kube-proxy supports three proxy modes&mdash;userspace, iptables and IPVS&mdash;which
each operate slightly differently.

#### Userspace

As an example, consider the image processing application described above.
When the backend Service is created, the Kubernetes master assigns a virtual
IP address, for example 10.0.0.1.  Assuming the Service port is 1234, the
Service is observed by all of the kube-proxy instances in the cluster.
When a proxy sees a new Service, it opens a new random port, establishes an
iptables redirect from the virtual IP address to this new port, and starts accepting
connections on it.

When a client connects to the Service's virtual IP address, the iptables
rule kicks in, and redirects the packets to the proxy's own port.
The “Service proxy” chooses a backend, and starts proxying traffic from the client to the backend.

This means that Service owners can choose any port they want without risk of
collision.  Clients can simply connect to an IP and port, without being aware
of which Pods they are actually accessing.

#### iptables

Again, consider the image processing application described above.
When the backend Service is created, the Kubernetes control plane assigns a virtual
IP address, for example 10.0.0.1.  Assuming the Service port is 1234, the
Service is observed by all of the kube-proxy instances in the cluster.
When a proxy sees a new Service, it installs a series of iptables rules which
redirect from the virtual IP address  to per-Service rules.  The per-Service
rules link to per-Endpoint rules which redirect traffic (using destination NAT)
to the backends.

When a client connects to the Service's virtual IP address the iptables rule kicks in.
A backend is chosen (either based on session affinity or randomly) and packets are
redirected to the backend.  Unlike the userspace proxy, packets are never
copied to userspace, the kube-proxy does not have to be running for the virtual
IP address to work, and Nodes see traffic arriving from the unaltered client IP
address.

This same basic flow executes when traffic comes in through a node-port or
through a load-balancer, though in those cases the client IP does get altered.

#### IPVS

iptables operations slow down dramatically in large scale cluster e.g 10,000 Services.
IPVS is designed for load balancing and based on in-kernel hash tables. So you can achieve performance consistency in large number of Services from IPVS-based kube-proxy. Meanwhile, IPVS-based kube-proxy has more sophisticated load balancing algorithms (least conns, locality, weighted, persistence).

## API Object

Service is a top-level resource in the Kubernetes REST API. You can find more details
about the API object at: [Service API object](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#service-v1-core).

## 지원 프로토콜 {#지원-프로토콜}

### TCP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

You can use TCP for any kind of Service, and it's the default network protocol.

### UDP

{{< feature-state for_k8s_version="v1.0" state="stable" >}}

You can use UDP for most Services. For type=LoadBalancer Services, UDP support
depends on the cloud provider offering this facility.

### HTTP

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

If your cloud provider supports it, you can use a Service in LoadBalancer mode
to set up external HTTP / HTTPS reverse proxying, forwarded to the Endpoints
of the Service.

{{< note >}}
You can also use {{< glossary_tooltip term_id="ingress" >}} in place of Service
to expose HTTP / HTTPS Services.
{{< /note >}}

### PROXY protocol

{{< feature-state for_k8s_version="v1.1" state="stable" >}}

If your cloud provider supports it (eg, [AWS](/docs/concepts/cluster-administration/cloud-providers/#aws)),
you can use a Service in LoadBalancer mode to configure a load balancer outside
of Kubernetes itself, that will forward connections prefixed with
[PROXY protocol](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt).

The load balancer will send an initial series of octets describing the
incoming connection, similar to this example

```
PROXY TCP4 192.0.2.202 10.0.42.7 12345 7\r\n
```
followed by the data from the client.

### SCTP

{{< feature-state for_k8s_version="v1.12" state="alpha" >}}

Kubernetes supports SCTP as a `protocol` value in Service, Endpoint, NetworkPolicy and Pod definitions as an alpha feature. To enable this feature, the cluster administrator needs to enable the `SCTPSupport` feature gate on the apiserver, for example, `--feature-gates=SCTPSupport=true,…`.

When the feature gate is enabled, you can set the `protocol` field of a Service, Endpoint, NetworkPolicy or Pod to `SCTP`. Kubernetes sets up the network accordingly for the SCTP associations, just like it does for TCP connections.

#### Warnings {#caveat-sctp-overview}

##### Support for multihomed SCTP associations {#caveat-sctp-multihomed}

{{< warning >}}
The support of multihomed SCTP associations requires that the CNI plugin can support the assignment of multiple interfaces and IP addresses to a Pod.

NAT for multihomed SCTP associations requires special logic in the corresponding kernel modules.
{{< /warning >}}

##### Service with type=LoadBalancer {#caveat-sctp-loadbalancer-service-type}

{{< warning >}}
You can only create a Service with `type` LoadBalancer plus `protocol` SCTP if the cloud provider's load balancer implementation supports SCTP as a protocol. Otherwise, the Service creation request is rejected. The current set of cloud load balancer providers (Azure, AWS, CloudStack, GCE, OpenStack) all lack support for SCTP.
{{< /warning >}}

##### Windows {#caveat-sctp-windows-os}

{{< warning >}}
SCTP is not supported on Windows based nodes.
{{< /warning >}}

##### Userspace kube-proxy {#caveat-sctp-kube-proxy-userspace}

{{< warning >}}
The kube-proxy does not support the management of SCTP associations when it is in userspace mode.
{{< /warning >}}

## Future work

In the future, the proxy policy for Services can become more nuanced than
simple round-robin balancing, for example master-elected or sharded.  We also
envision that some Services will have "real" load balancers, in which case the
virtual IP address will simply transport the packets there.

The Kubernetes project intends to improve support for L7 (HTTP) Services.

The Kubernetes project intends to have more flexible ingress modes for Services
which encompass the current ClusterIP, NodePort, and LoadBalancer modes and more.


{{% /capture %}}

{{% capture whatsnext %}}

* Read [Connecting Applications with Services](/docs/concepts/services-networking/connect-applications-service/)
* Read about [Ingress](/docs/concepts/services-networking/ingress/)

{{% /capture %}}
