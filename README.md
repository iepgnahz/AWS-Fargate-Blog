# AWS-Fargate-Blog

## 背景

随着容器化的流行和普遍，我们对APP的部署和构建方式都发生了很大的改变。
为了适应这一潮流，很多云服务供应商（比如AWS）发布了各种新的service，帮助简化APP所需容器在云上的编排，部署和管理。

比如在AWS上，最常见两个容器编排service：

- ECS
- EKS

以ECS为例， 虽然ECS可以帮助我们编排APP中不同task所需的不同容器，以及容器之间的联系。
但是我们不能仅仅只关注在容器上，我们还需考虑和配置容器运行在什么样的云服务上。
为了保证APP服务的可用性和可扩展性，我们需要管理和配置容器所运行在的云服务集群（比如所有的容器都运行在EC2上，我们就需要管理我们的EC2集群）
由此，及时我们使用了ECS，我们仍然需要配置和管理容器运行在的实例。

由于容器化的普及，我们希望关注点能够更集中在打包了我们所有服务的容器上，而并非底层实例，此时fargate应运而生。


接下来，我们就要开始学习一种新的service： Fargate

一个专门处理容器的serverless计算平台， 简而言之， 
就是一个专门用来运行容器的serverless的云计算服务。

## Need For AWS Fargate

当docker容器化还没有出现的时候，人们通常将自己的APP部署到VM上。
所以当时的云服务商AWS发布了自己的云计算服务AWS EC2, EC2就是一个虚拟机， 用户可以直接将自己服务部署到EC2实例上。
为了让APP更快的被部署，我们通常将自己的App和操作系统一起打包成一个AMI。
然后直接作为EC2的AMI被运行即可。

然后docker 容器化开始盛行，用户开始将自己的APP部署到docker容器中，将自己的APP和所需的所有工具打包成一个docker镜像， 在需要的时候在任何有docker engine的实例上运行起来即可。

那么容器和虚拟机的最大区别其实就是，在同一个实例上运行的所有容器都共享主机的操作系统， 而不同的虚拟机中都有自己的操作系统，其实就是一个独立的主机， 一个EC2实例其实就是一个云上虚拟机。

<img width="608" alt="image" src="https://user-images.githubusercontent.com/19647596/170998431-9c990e1b-c90b-4247-99d0-32d7d409809b.png">

随着容器化的普及，人们更倾向于将自己的APP部署到容器中而不是虚拟机中，因此人们从以前操作虚拟机转成了现在操作容器。通常一个APP能够工作，需要多个容器配合工作， 为了方便APP的部署，AWS 发布了ECS

ECS就是一个拥有高扩展性和高性能的容器编排工具，ECS帮助用户减少了很多容器管理的开销，但是用户仍然需要管理容器运行在的实例集群，也就是容器运行的宿主虚拟机集群。

对于用户来说虽然ECS能够帮助编排和管理容器， 但是用户依然需
部署和管理容器运行在的底层宿主虚拟机集群，那么如果还有一个工具，能够负责部署和管理容器运行在宿主集群，这样用户就可以只关注于APP。因此AWS Fargate出现

## What is AWS Fargate
AWS Fargate 是 `serverless`  计算服务， 专门用于运行`容器` 。Fargate负责部署和配置你所有容器运行所需要的实例集群（VM） ， 并且根据流量对集群进行扩缩容。

AWS Fargate 也是 `ECS` 的compute engine，ECS作为容器编排工具，负责编排容器，但是不负责容器所运行在的底层实例（虚拟机）集群。Fargate和EC2是ECS现有的两种不同的compute engine，也就是两种不同的`launch type`

- Fargate是serverless的compute engine (Fargate Launch Type)
- EC2 是非serverless的compute engine(EC2 Launch Type)



## AWS Fargate vs Lambda

AWS Fargate 是 `serverless` compute service， 这个定义想必你一定不会觉得陌生，因为我们最常用的lambda就是一个`serverless compute service`.

AWS Fargate 和 lambda 虽然都是serverless的服务。区别在于：

- Fargate主要用于运行container（容器）

`在本地` 通常为了让你的APP能够正常运行，我们可能会使用`docker-compose`（docker容器编排工具）同时setup多个容器（APP + DB + nginx容器...），并让他们能够正常的运行在我的机器上。

`在AWS上` Fargate只是一种serverless的服务，类似于在云上`按需`部署并setup起来多个实例，用于运行多个容器。由此观之，想要Fargate单独工作似乎还不行，还需要一个在AWS上的容器编排工具， 此时你应该已经猜到了，就是ECS（PS: EKS也可以作为容器编排工具和Fargate一起工作，本文就不提了）。
因此Fargate又被看作为ECS的compute engine。

- Lambda其实是一个按需启动的实例，使用AWS lambda你不需要考虑实例的配置和部署工作， 只要选择一种运行环境，并把需要运行的代码上传到lambda上，lambda就会根据流量进行自动的扩缩容工作，你完全不需要担心。
