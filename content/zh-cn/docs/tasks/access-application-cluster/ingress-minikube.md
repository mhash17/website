---
title: 在 Minikube 环境中使用 NGINX Ingress 控制器配置 Ingress
content_type: task
weight: 100
min-kubernetes-server-version: 1.19
---
<!--
title: Set up Ingress on Minikube with the NGINX Ingress Controller
content_type: task
weight: 100
min-kubernetes-server-version: 1.19
-->

<!-- overview -->

<!--
An [Ingress](/docs/concepts/services-networking/ingress/) is an API object that defines rules
which allow external access to services in a cluster. An
[Ingress controller](/docs/concepts/services-networking/ingress-controllers/)
fulfills the rules set in the Ingress.

This page shows you how to set up a simple Ingress which routes requests to Service 'web' or
'web2' depending on the HTTP URI.
-->
[Ingress](/zh-cn/docs/concepts/services-networking/ingress/)是一种 API 对象，
其中定义了一些规则使得集群中的服务可以从集群外访问。
[Ingress 控制器](/zh-cn/docs/concepts/services-networking/ingress-controllers/)负责满足
Ingress 中所设置的规则。

本节为你展示如何配置一个简单的 Ingress，根据 HTTP URI 将服务请求路由到服务 `web` 或 `web2`。

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}
<!--
If you are using an older Kubernetes version, switch to the documentation for that version.
-->
如果你使用的是较早的 Kubernetes 版本，请切换到该版本的文档。

<!-- steps -->

<!--
### Create a Minikube cluster

Using Katacoda
: {{< kat-button >}}

Locally
: If you already [installed Minikube](/docs/tasks/tools/#minikube)
  locally, run `minikube start` to create a cluster.
-->
### 创建一个 Minikube  集群  {#create-minikube-cluster}

使用 Katacoda
: {{< kat-button >}}

本地
: 如果已经在本地[安装 Minikube](/zh-cn/docs/tasks/tools/#minikube)，
  请运行 `minikube start` 创建一个集群。

<!-- steps -->

<!--
## Enable the Ingress controller

1. To enable the NGINX Ingress controller, run the following command:
-->
## 启用 Ingress 控制器

1. 为了启用 NGINIX Ingress 控制器，可以运行下面的命令：

   ```shell
   minikube addons enable ingress
   ```

<!--
1. Verify that the NGINX Ingress controller is running
-->
2. 检查验证 NGINX Ingress 控制器处于运行状态：

   {{< tabs name="tab_with_md" >}}
   {{% tab name="minikube v1.19 或更高版本" %}}

   ```shell
   kubectl get pods -n ingress-nginx
   ```

   {{< note >}}
   <!--
   It can take up to a minute before you see these pods running OK.
   -->
   最多可能需要等待一分钟才能看到这些 Pod 运行正常。
   {{< /note >}}

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   NAME                                        READY   STATUS      RESTARTS    AGE
   ingress-nginx-admission-create-g9g49        0/1     Completed   0          11m
   ingress-nginx-admission-patch-rqp78         0/1     Completed   1          11m
   ingress-nginx-controller-59b45fb494-26npt   1/1     Running     0          11m
   ```

   {{% /tab %}}
   {{% tab name="minikube v1.18.1 或更早版本" %}}

   ```shell
   kubectl get pods -n kube-system
   ```

   {{< note >}}
   <!--
   It can take up to a minute before you see these pods running OK.
   -->
   最多可能需要等待一分钟才能看到这些 Pod 运行正常。
   {{< /note >}}

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   NAME                                        READY     STATUS    RESTARTS   AGE
   default-http-backend-59868b7dd6-xb8tq       1/1       Running   0          1m
   kube-addon-manager-minikube                 1/1       Running   0          3m
   kube-dns-6dcb57bcc8-n4xd4                   3/3       Running   0          2m
   kubernetes-dashboard-5498ccf677-b8p5h       1/1       Running   0          2m
   nginx-ingress-controller-5984b97644-rnkrg   1/1       Running   0          1m
   storage-provisioner                         1/1       Running   0          2m
   ```

   <!--
   Make sure that you see a Pod with a name that starts with `nginx-ingress-controller-`.
   -->
   请确保可以在输出中看到一个名称以 `nginx-ingress-controller-` 为前缀的 Pod。

   {{% /tab %}}
   {{< /tabs >}}

<!--
## Deploy a hello, world app

1. Create a Deployment using the following command:
-->
## 部署一个 Hello World 应用  {#deploy-hello-world}

1. 使用下面的命令创建一个 Deployment：

   ```shell
   kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
   ```

   <!--
   The output should be:
   -->
   输出：

   ```none
   deployment.apps/web created
   ```

<!--
1. Expose the Deployment:
-->
2. 将 Deployment 暴露出来：

   ```shell
   kubectl expose deployment web --type=NodePort --port=8080
   ```

   <!--
   The output should be:
   -->
   输出类似于：

   ```none
   service/web exposed
   ```

<!--
1. Verify the Service is created and is available on a node port:
-->
3. 验证 Service 已经创建，并且可以从节点端口访问：

   ```shell
   kubectl get service web
   ```

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
   web       NodePort   10.104.133.249   <none>        8080:31637/TCP   12m
   ```

<!--
1. Visit the service via NodePort:
-->
4. 使用节点端口信息访问服务：

   ```shell
   minikube service web --url
   ```

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   http://172.17.0.15:31637
   ```

   {{< note >}}
   <!--
   Katacoda environment only: at the top of the terminal panel, click the plus sign,
   and then click **Select port to view on Host 1**. Enter the NodePort value,
   in this case `31637`, and then click **Display Port**.
   -->
   如果使用的是 Katacoda 环境，在终端面板顶端，请点击加号标志。
   然后点击 **Select port to view on Host 1**。
   输入节点和端口号（这里是`31637`），之后点击 **Display Port**。
   {{< /note >}}

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

   <!--
   You can now access the sample application via the Minikube IP address and NodePort.
   The next step lets you access the application using the Ingress resource.
   -->
   你现在应该可以通过 Minikube 的 IP 地址和节点端口来访问示例应用了。
   下一步是让自己能够通过 Ingress 资源来访问应用。

<!--
## Create an Ingress

The following manifest defines an Ingress that sends traffic to your Service via
`hello-world.info`.

1. Create `example-ingress.yaml` from the following file:
-->
## 创建一个 Ingress

下面是一个定义 Ingress 的配置文件，负责通过 `hello-world.info`
将请求转发到你的服务。

1. 根据下面的 YAML 创建文件 `example-ingress.yaml`：

   {{< codenew file="service/networking/example-ingress.yaml" >}}

<!--
1. Create the Ingress object by running the following command:
-->
2. 通过运行下面的命令创建 Ingress 对象：

   ```shell
   kubectl apply -f https://k8s.io/examples/service/networking/example-ingress.yaml
   ```

   <!--
   The output should be:
   -->
   输出类似于：

   ```none
   ingress.networking.k8s.io/example-ingress created
   ```

<!--
1. Verify the IP address is set:
-->
3. 验证 IP 地址已被设置：

   ```shell
   kubectl get ingress
   ```

   {{< note >}}
   <!--
   This can take a couple of minutes.
   -->
   此操作可能需要几分钟时间。
   {{< /note >}}

   <!--
   You should see an IPv4 address in the `ADDRESS` column; for example:
   -->
   接下来你将会在 `ADDRESS` 列中看到 IPv4 地址，例如：

   ```none
   NAME              CLASS    HOSTS              ADDRESS        PORTS   AGE
   example-ingress   <none>   hello-world.info   172.17.0.15    80      38s
   ```

<!--
1. Add the following line to the bottom of the `/etc/hosts` file on
   your computer (you will need administrator access):
-->
4. 在 `/etc/hosts` 文件的末尾添加以下内容（需要管理员访问权限）：

   ```none
   172.17.0.15 hello-world.info
   ```

   {{< note >}}
   <!--
   If you are running Minikube locally, use `minikube ip` to get the external IP.
   The IP address displayed within the ingress list will be the internal IP.
   -->
   如果你在本地运行 Minikube 环境，需要使用 `minikube ip` 获得外部 IP 地址。
   Ingress 列表中显示的 IP 地址会是内部 IP 地址。
   {{< /note >}}

   <!--     
   After you make this change, your web browser sends requests for
   `hello-world.info` URLs to Minikube.
   -->
   添加完成后，在浏览器中访问 URL `hello-world.info`，请求将被发送到 Minikube。

<!--
1. Verify that the Ingress controller is directing traffic:
-->
5. 验证 Ingress 控制器能够转发请求流量：

   ```shell
   curl hello-world.info
   ```

   <!--
   You should see:
   -->
   你应该看到类似输出：

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

   {{< note >}}
   <!--
   If you are running Minikube locally, you can visit `hello-world.info` from your browser.
   -->
   如果你在使用本地 Minikube 环境，你可以从浏览器中访问 hello-world.info。
   {{< /note >}}

<!--
## Create a second Deployment

1. Create another Deployment using the following command:
-->
## 创建第二个 Deployment  {#create-second-deployment}

1. 使用下面的命令创建第二个 Deployment：

   ```shell
   kubectl create deployment web2 --image=gcr.io/google-samples/hello-app:2.0
   ```

   <!--
   The output should be:
   -->
   输出类似于：

   ```none
   deployment.apps/web2 created
   ```

<!--
1. Expose the second Deployment:
-->
2. 将第二个 Deployment 暴露出来：

   ```shell
   kubectl expose deployment web2 --port=8080 --type=NodePort
   ```

   <!--
   The output should be:
   -->
   输出类似于：

   ```none
   service/web2 exposed
   ```

<!--
## Edit the existing Ingress {#edit-ingress}

1. Edit the existing `example-ingress.yaml` manifest, and add the
   following lines at the end:
-->
## 编辑现有的 Ingress {#edit-ingress}

1. 编辑现有的 `example-ingress.yaml`，在文件最后添加以下行：

   ```yaml
   - path: /v2
     pathType: Prefix
     backend:
       service:
         name: web2
         port:
           number: 8080
   ```

<!--
1. Apply the changes:
-->
2. 应用变更：

   ```shell
   kubectl apply -f example-ingress.yaml
   ```

   <!--
   You should see:
   -->
   输出类似于：

   ```none
   ingress.networking/example-ingress configured
   ```

<!--
## Test Your Ingress

1. Access the 1st version of the Hello World app.
-->
## 测试你的 Ingress  {#test-ingress}

1. 访问 Hello World 应用的第一个版本：

   ```shell
   curl hello-world.info
   ```

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   Hello, world!
   Version: 1.0.0
   Hostname: web-55b8c6998d-8k564
   ```

<!--
1. Access the 2nd version of the Hello World app.
-->
2. 访问 Hello World 应用的第二个版本：

   ```shell
   curl hello-world.info/v2
   ```

   <!--
   The output is similar to:
   -->
   输出类似于：

   ```none
   Hello, world!
   Version: 2.0.0
   Hostname: web2-75cd47646f-t8cjk
   ```

   {{< note >}}
   <!--
   If you are running Minikube locally, you can visit `hello-world.info` and
   `hello-world.info/v2` from your browser.
   -->
   如果你在本地运行 Minikube 环境，你可以使用浏览器来访问
   `hello-world.info` 和 `hello-world.info/v2`。
   {{< /note >}}

## {{% heading "whatsnext" %}}

<!--
* Read more about [Ingress](/docs/concepts/services-networking/ingress/)
* Read more about [Ingress Controllers](/docs/concepts/services-networking/ingress-controllers/)
* Read more about [Services](/docs/concepts/services-networking/service/)
-->
* 进一步了解 [Ingress](/zh-cn/docs/concepts/services-networking/ingress/)
* 进一步了解 [Ingress 控制器](/zh-cn/docs/concepts/services-networking/ingress-controllers/)
* 进一步了解 [服务](/zh-cn/docs/concepts/services-networking/service/)
