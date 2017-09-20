# ECS

AWS ECS

---

# What is ECS?
Amazon EC2 Container Service (Amazon ECS) is a highly scalable, fast, container management service that makes it easy to run, stop, and manage Docker containers on a cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances. Amazon ECS lets you launch and stop container-based applications with simple API calls, allows you to get the state of your cluster from a centralized service, and gives you access to many familiar Amazon EC2 features.

You can use Amazon ECS to schedule the placement of containers across your cluster based on your resource needs, isolation policies, and availability requirements. Amazon ECS eliminates the need for you to operate your own cluster management and configuration management systems or worry about scaling your management infrastructure.

Amazon ECS can be used to create a consistent deployment and build experience, manage and scale batch and Extract-Transform-Load (ETL) workloads, and build sophisticated application architectures on a microservices model. For more information about Amazon ECS use cases and scenarios, see [Container Use Cases][1].

# Features of Amazon ECS
Amazon ECS is a regional service that simplifies running application containers in a highly available manner across multiple Availability Zones within a region. You can create Amazon ECS clusters within a new or existing VPC. After a cluster is up and running, you can define task definitions and services that specify which Docker container images to run across your clusters. Container images are stored in and pulled from container registries, which may exist within or outside of your AWS infrastructure.

![此处输入图片的描述][2]

# Tasks and Scheduling
A task is the instantiation of a task definition on a container instance within your cluster. After you have created a task definition for your application within Amazon ECS, you can specify the number of tasks that will run on your cluster.

The Amazon ECS task scheduler is responsible for placing tasks on container instances. There are several different scheduling options available. For example, you can define a service that runs and maintains a specified number of tasks simultaneously.

![此处输入图片的描述][3]

# Container Agent

The container agent runs on each instance within an Amazon ECS cluster. It sends information about the instance's current running tasks and resource utilization to Amazon ECS, and starts and stops tasks whenever it receives a request from Amazon ECS.

![此处输入图片的描述][4]

# Scheduling Amazon ECS Tasks
Amazon EC2 Container Service (Amazon ECS) is a shared state, optimistic concurrency system that provides flexible scheduling capabilities for your tasks and containers. The Amazon ECS schedulers leverage the same cluster state information provided by the Amazon ECS API to make appropriate placement decisions.

Amazon ECS provides a service scheduler (for long-running tasks and applications), the ability to run tasks manually (for batch jobs or single run tasks), with Amazon ECS placing tasks on your cluster for you, and the ability to run tasks on the container instance that you specify, so that you can integrate with custom or third-party schedulers or place a task manually on a specific container instance.

## Service Concepts
Amazon ECS allows you to run and maintain a specified number (the "desired count") of instances of a task definition simultaneously in an ECS cluster. This is called a service. If any of your tasks should fail or stop for any reason, the Amazon ECS service scheduler launches another instance of your task definition to replace it and maintain the desired count of tasks in the service.

In addition to maintaining the desired count of tasks in your service, you can optionally run your service behind a load balancer. The load balancer distributes traffic across the tasks that are associated with the service.

 - If a task in a service stops, the task is killed and restarted. This process continues until your service reaches the number of desired running tasks.
 - You can optionally run your service behind a load balancer.
 - You can optionally specify a deployment configuration for your service. During a deployment (which is triggered by updating the task definition or desired count of a service), the service scheduler uses the minimum healthy percent and maximum percent parameters to determine the deployment strategy. 
 - When the service scheduler launches new tasks, it attempts to balance them across the Availability Zones in your cluster with the following logic:
    - Determine which of the container instances in your cluster can support your service's task definition (for example, they have the required CPU, memory, ports, and container instance attributes).
    - Sort the valid container instances by the fewest number of running tasks for this service in the same Availability Zone as the instance. For example, if zone A has one running service task and zones B and C each have zero, valid container instances in either zone B or C are considered optimal for placement.
    - Place the new service task on a valid container instance in an optimal Availability Zone (based on the previous steps), favoring container instances with the fewest number of running tasks for this service.
 - When the service scheduler stops running tasks, it attempts to maintain balance across the Availability Zones in your cluster with the following logic:
  - Sort the container instances by the largest number of running tasks for this service in the same Availability Zone as the instance. For example, if zone A has one running service task and zones B and C each have two, container instances in either zone B or C are considered optimal for termination.
  - Stop the task on a container instance in an optimal Availability Zone (based on the previous steps), favoring container instances with the largest number of running tasks for this service.

## Service Scheduler
The service scheduler is ideally suited for **long running stateless services and applications**. The service scheduler ensures that the specified number of tasks are constantly running and reschedules tasks when a task fails (for example, if the underlying container instance fails for some reason). The service scheduler optionally also makes sure that tasks are registered against an Elastic Load Balancing load balancer. You can update your services that are maintained by the service scheduler, such as deploying a new task definition, or changing the running number of desired tasks. By default, the service scheduler spreads tasks across Availability Zones, but you can use task placement strategies and constraints to customize task placement decisions. For more information, see Services.

## Manually Running Tasks
The RunTask action is ideally suited for processes such as **batch jobs that perform work and then stop**. For example, you could have a process call RunTask when work comes into a queue. The task pulls work from the queue, performs the work, and then exits. Using RunTask, you can allow the default task placement strategy to distribute tasks randomly across your cluster, which minimizes the chances that a single instance gets a disproportionate number of tasks. Alternatively, you can use RunTask to customize how the scheduler places tasks using task placement strategies and constraints.

## Running Tasks on a *cron*-like Schedule
If you have tasks to run at set intervals in your cluster, such as a backup operation or a log scan, you can use the Amazon ECS console to create a CloudWatch Events rule that runs one or more tasks in your cluster at specified times. Your scheduled event rule can be set to either a specific interval (run every N minutes, hours, or days), or for more complicated scheduling, you can use a cron expression.
## Custom Schedulers
Amazon ECS allows you to create your own schedulers that meet the needs of your business, or to leverage third party schedulers. [Blox][5] is an open source project that gives you more control over how your containerized applications run on Amazon ECS. It enables you to build schedulers and integrate third-party schedulers with Amazon ECS while leveraging Amazon ECS to fully manage and scale your clusters. Custom schedulers use the [StartTask][6] API operation to place tasks on specific container instances within your cluster. 

# Amazon ECS Task Placement
When you launch a task into a cluster, Amazon ECS must determine where to place the task based on the requirements specified in the task definition, such as CPU and memory. Similarly, when you scale down the task count, Amazon ECS must determine which tasks to terminate. You can apply task placement strategies and constraints to customize how Amazon ECS places and terminates tasks.

A **task placement strategy** is an algorithm for selecting instances for task placement or tasks for termination. For example, Amazon ECS can select instances at random or it can select instances such that tasks are distributed evenly across a group of instances. A **task placement constraint** is a rule that is considered during task placement. For example, you can use constraints to place tasks based on Availability Zone or instance type. You can associate attributes, which are name/value pairs, with your container instances and then use a constraint to place tasks based on attribute.

You can use strategies and constraints together. For example, you can distribute tasks across Availability Zones and bin pack tasks based on memory within each Availability Zone, but only for G2 instances.

When Amazon ECS places tasks, it uses the following process to select container instances:
 1. Identify the instances that satisfy the CPU, memory, and port requirements in the task definition.
 2. Identify the instances that satisfy the task placement constraints.
 3. Identify the instances that satisfy the task placement strategies.
 4. Select the instances for task placement.

## Amazon ECS Task Placement Strategies ##
Amazon ECS supports the following task placement strategies:

 - binpack
    Place tasks based on the least available amount of CPU or memory. This minimizes the number of instances in use.
 - random
    Place tasks randomly
 - spread
Place tasks evenly based on the specified value. Accepted values are attribute key:value pairs, instanceId, or host. Service tasks are spread based on the tasks from that service.

## Amazon ECS Task Placement Constraints ##
Amazon ECS supports the following types of task placement constraints:

 - distinctInstance
    Place each task on a different container instance.
 - memberOf
    Place tasks on container instances that satisfy an expression.

  
 
  [1]: https://aws.amazon.com/containers/use-cases/
  [2]: http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview.png
  [3]: http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-service.png
  [4]: http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-clusteragent.png
  [5]: https://blox.github.io/
  [6]: http://docs.aws.amazon.com/AmazonECS/latest/APIReference//API_StartTask.html
