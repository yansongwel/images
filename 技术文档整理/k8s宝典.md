# MySQL

### 清理之前的mysql

```
#yum remove mysql* mariadb* -y           

#rm /etc/my.cnf                          

#rm -rf /var/lib/mysql                   

#rm -rf /usr/share/mysql                 

#rm -rf /usr/lib/mysql                   

查询mysql服务                            

#systemctl list-unit-files | grep mysql  

#systemctl disable mysqld.service        

#systemctl disable mysql.service         

#rm -rf /var/run/mysql/                  

#rm -rf /etc/mecabrc                         

#rm -rf /usr/lib/systemd/system/mysqld.service

#rm -rf /etc/systemd/system/mysqld.service   

#rm -rf /etc/systemd/system/mysql.service    
```



## 1，mysql 安装

官方文档安装地址：	

https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html

1. 运行以下命令更新YUM源。

   ```
   rpm -Uvh  https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
   ```

2. 运行以下命令安装MySQL。

   ```
   yum -y install mysql-community-server
   ```

3. 运行以下命令查看MySQL版本号。

   ```
   mysql -V
   ```

## 2，mysql配置

1. 运行以下命令启动MySQL服务。

   ```
   systemctl start mysqld
   ```

2. 运行以下命令设置MySQL服务开机自启动。

   ```
   systemctl enable mysqld
   ```

3. 运行以下命令查看/var/log/mysqld.log文件，获取并记录root用户的初始密码。

   ```
   grep 'temporary password' /var/log/mysqld.log
   ```

   执行命令结果示例如下。

   ```
   2021-11-19T05:38:55.769079Z 1 [Note] A temporary password is generated for root@localhost: &8dyNIejohuD
   ```

通过使用生成的临时密码登录并为超级用户帐户设置自定义密码，尽快更改 root 密码：

```terminal
$> mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```

笔记

[`validate_password`](https://dev.mysql.com/doc/refman/5.7/en/validate-password.html) 默认安装。执行的默认密码策略`validate_password`要求密码至少包含1个大写字母、1个小写字母、1个数字和1个特殊字符，并且密码总长度至少为8个字符。  



  4.运行下列命令对MySQL进行安全性配置。

```
mysql_secure_installation
```

1. 重置root用户的密码。

   **说明** 在输入密码时，系统为了最大限度的保证数据安全，命令行将不做任何回显。您只需要输入正确的密码信息，然后按Enter键即可。

   ```
   Enter password for user root: #输入上一步获取的root用户初始密码
   The 'validate_password' plugin is installed on the server.
   The subsequent steps will run with the existing configuration of the plugin.
   Using existing password for root.
   Estimated strength of the password: 100 
   Change the password for root ? ((Press y|Y for Yes, any other key for No) : Y #是否更改root用户密码，输入Y
   New password: #输入新密码，长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号可以是()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/
   Re-enter new password: #再次输入新密码
   Estimated strength of the password: 100 
   Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y #是否继续操作，输入Y
   ```

2. 删除匿名用户账号。

   ```
   By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.
   Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y  #是否删除匿名用户，输入Y
   Success.
   ```

3. 禁止root账号远程登录。

   ```
   Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y #禁止root远程登录，输入Y
   Success.
   ```

4. 删除test库以及对test库的访问权限。

   ```
   Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y #是否删除test库和对它的访问权限，输入Y
   - Dropping test database...
   Success.
   ```

5. 重新加载授权表。

   ```
   Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y #是否重新加载授权表，输入Y
   Success.
   All done!
   ```

安全性配置的更多信息，请参见[MySQL官方文档](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)。

## 远程访问MySQL数据库

您可以使用数据库客户端或阿里云提供的数据管理服务DMS（Data Management Service）来远程访问MySQL数据库。本节以DMS为例，介绍远程访问MySQL数据库的操作步骤。

1. 在ECS实例上，创建远程登录MySQL的账号。

   1. 运行以下命令后，输入root用户的密码登录MySQL。

      ```
       mysql -uroot -p
      ```

   2. 依次运行以下命令创建远程登录MySQL的账号。

      建议您使用非root账号远程登录MySQL数据库，本示例账号为`dms`、密码为`123456`。

      **注意** 实际创建账号时，需将示例密码`123456`更换为符合要求的密码，并妥善保存。密码要求：长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。可以使用以下特殊符号：

      `()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/`

      ```
      mysql> grant all on *.* to 'dms'@'%' IDENTIFIED BY '123456'; #使用root替换dms，可设置为允许root账号远程登录。
      mysql> flush privileges;
      ```

2. 登录[数据管理控制台](https://dms.console.aliyun.com/)。

3. 在左侧导航栏中，选择**自建库（ECS、公网）**。

4. 单击**新增数据库**。

5. 配置自建数据库信息。 具体操作，请参见[管理ECS实例自建数据库](https://help.aliyun.com/document_detail/111100.htm#task-wyv-z3g-chb)。

6. 单击**登录**。

   成功登录后，您可以使用DMS提供的菜单栏功能，创建数据库、表、函数等。





# docker



dockerfile



docker-compose



# k8s

k8s介绍





k8s安装







### k8s命令介绍：

#### 1.kubectl explain命令详解：

```
~]# kubectl api-resources  #列出可以查看的资源
~]# kubectl explain pods.spec.containers.env.valueFrom
KIND:     Pod
VERSION:  v1

RESOURCE: valueFrom <Object>

DESCRIPTION:
     Source for the environment variable's value. Cannot be used if value is not
     empty.

     EnvVarSource represents a source for the value of an EnvVar.

FIELDS:
   configMapKeyRef	<Object>
     Selects a key of a ConfigMap.

   fieldRef	<Object>
     Selects a field of the pod: supports metadata.name, metadata.namespace,
     `metadata.labels['<KEY>']`, `metadata.annotations['<KEY>']`, spec.nodeName,
     spec.serviceAccountName, status.hostIP, status.podIP, status.podIPs.

   resourceFieldRef	<Object>
     Selects a resource of the container: only resources limits and requests
     (limits.cpu, limits.memory, limits.ephemeral-storage, requests.cpu,
     requests.memory and requests.ephemeral-storage) are currently supported.

   secretKeyRef	<Object>
     Selects a key of a secret in the pod's namespace

```







#### k8s端口

Kubernetes - Port，Targetport和NodePort 1
使用Kubernetes Service时，会遇到以下一些术语：

- port 端口：端口是使服务对同一K8群集中运行的其他服务可见的端口号。换句话说，如果服务想要调用在同一Kubernetes集群中运行的另一个服务，它将能够使用服务规范文件中针对“port”指定的端口来执行此操作。
- targetPort：目标端口是POD上运行服务的端口。
- nodePort：节点端口是使用Kube-Proxy从外部用户访问服务的端口。

看一下定义示例服务的以下规范：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - port: 8080         #service端口 这个端口可以自定义
    targetPort: 8070   #pod端口及容器端口，不可自定义
    nodePort: 8888     #节点端口  可自定义
    protocol: TCP 
  selector:
    component: my-service-app
   
```

请注意上述规范中的以下部分内容：

- **port 8080**: 表示该服务可由群集中的其他服务的在端口8080访问。
- **targetPort 8070**: 代表了my-service 实际上是在pod 的 8070 端口上运行的
- **nodePort 8888**: 表示为了服务可经由kube-proxy上端口8888 进行访问。

这是 Kubernetes 的ipTables 配置的结果。它维护 nodePort 与 targetPort 的映射。K8s Kube-Proxy使用ipTables来解析特定nodePort上的请求，并将它们重定向到适当的pod。





原始服务不变，新增service

```
#原始service
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: arms-prometheus-ack-arms-prometheus
    chart: ack-arms-prometheus-0.1.5
    heritage: Helm
    kubernetes.io/service-name: prometheus-admin
    release: arms-prometheus
  name: arms-prom-admin
  namespace: arms-prom
spec:
  clusterIP: 172.16.225.64
  clusterIPs:
    - 172.16.225.64
  ports:
    - name: operator
      port: 9335
      protocol: TCP
      targetPort: 9335
  selector:
    app: arms-prometheus-ack-arms-prometheus
  sessionAffinity: None
  type: ClusterIP



#我新增的
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: arms-prometheus-ack-arms-prometheus
    chart: ack-arms-prometheus-0.1.5
    kubernetes.io/service-name: prometheus-admin
    release: arms-prometheus
  name: arms-prom-45
  namespace: arms-prom
spec:
  ports:
    - name: operator
      nodePort: 39335
      port: 9335
      protocol: TCP
      targetPort: 9335
  selector:
    app: arms-prometheus-ack-arms-prometheus
  sessionAffinity: None
  type: NodePort
```

![image-20211119183235924](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111191832974.png)

![image-20211119183510305](https://betagitlab.oss-cn-shanghai.aliyuncs.com/Beta_gitlab%E8%BF%81%E7%A7%BB%E5%8F%8A%E5%8D%87%E7%BA%A7202111191835925.png)



### k8s存储介绍

#### 1.k8s 卷







##### hostPath

**警告：**

HostPath 卷存在许多安全风险，最佳做法是尽可能避免使用 HostPath。 当必须使用 HostPath 卷时，它的范围应仅限于所需的文件或目录，并以只读方式挂载。

如果通过 AdmissionPolicy 限制 HostPath 对特定目录的访问， 则必须要求 `volumeMounts` 使用 `readOnly` 挂载以使策略生效。

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

例如，`hostPath` 的一些用法有：

- 运行一个需要访问 Docker 内部机制的容器；可使用 `hostPath` 挂载 `/var/lib/docker` 路径。
- 在容器中运行 cAdvisor 时，以 `hostPath` 方式挂载 `/sys`。
- 允许 Pod 指定给定的 `hostPath` 在运行 Pod 之前是否应该存在，是否应该创建以及应该以什么方式存在。

除了必需的 `path` 属性之外，用户可以选择性地为 `hostPath` 卷指定 `type`。

支持的 `type` 值如下：

| 取值                | 行为                                                         |
| :------------------ | :----------------------------------------------------------- |
|                     | 空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。 |
| `DirectoryOrCreate` | 如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。 |
| `Directory`         | 在给定路径上必须存在的目录。                                 |
| `FileOrCreate`      | 如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。 |
| `File`              | 在给定路径上必须存在的文件。                                 |
| `Socket`            | 在给定路径上必须存在的 UNIX 套接字。                         |
| `CharDevice`        | 在给定路径上必须存在的字符设备。                             |
| `BlockDevice`       | 在给定路径上必须存在的块设备。                               |

当使用这种类型的卷时要小心，因为：

- HostPath 卷可能会暴露特权系统凭据（例如 Kubelet）或特权 API（例如容器运行时套接字）， 可用于容器逃逸或攻击集群的其他部分。
- 具有相同配置（例如基于同一 PodTemplate 创建）的多个 Pod 会由于节点上文件的不同 而在不同节点上有不同的行为。
- 下层主机上创建的文件或目录只能由 root 用户写入。你需要在 [特权容器](https://kubernetes.io/zh/docs/tasks/configure-pod-container/security-context/) 中以 root 身份运行进程，或者修改主机上的文件权限以便容器能够写入 `hostPath` 卷。

##### hostPath 配置示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # 宿主上目录位置
      path: /data
      # 此字段为可选
      type: Directory
```

**注意：** `FileOrCreate` 模式不会负责创建文件的父目录。 如果欲挂载的文件的父目录不存在，Pod 启动会失败。 为了确保这种模式能够工作，可以尝试把文件和它对应的目录分开挂载，如 [`FileOrCreate` 配置](https://kubernetes.io/zh/docs/concepts/storage/volumes/#hostpath-fileorcreate-example) 所示。

##### hostPath FileOrCreate 配置示例 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: k8s.gcr.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # 确保文件所在目录成功创建。
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```



##### 使用 subPath 

有时，在单个 Pod 中共享卷以供多方使用是很有用的。 `volumeMounts.subPath` 属性可用于指定所引用的卷内的子路径，而不是其根路径。

下面例子展示了如何配置某包含 LAMP 堆栈（Linux Apache MySQL PHP）的 Pod 使用同一共享卷。 此示例中的 `subPath` 配置不建议在生产环境中使用。 PHP 应用的代码和相关数据映射到卷的 `html` 文件夹，MySQL 数据库存储在卷的 `mysql` 文件夹中：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```

##### 使用带有扩展环境变量的 subPath 

**FEATURE STATE:** `Kubernetes v1.17 [stable]`

使用 `subPathExpr` 字段可以基于 Downward API 环境变量来构造 `subPath` 目录名。 `subPath` 和 `subPathExpr` 属性是互斥的。

在这个示例中，Pod 使用 `subPathExpr` 来 hostPath 卷 `/var/log/pods` 中创建目录 `pod1`。 `hostPath` 卷采用来自 `downwardAPI` 的 Pod 名称生成目录名。 宿主目录 `/var/log/pods/pod1` 被挂载到容器的 `/logs` 中。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```