# Task Metadata Endpoint version 2<a name="task-metadata-endpoint-v2"></a>

Beginning with version 1\.17\.0 of the Amazon ECS container agent, various task metadata and [Docker stats](https://docs.docker.com/engine/api/v1.30/#operation/ContainerStats) are available to tasks that use the `awsvpc` network mode at an HTTP endpoint that is provided by the Amazon ECS container agent\.

All containers belonging to tasks that are launched with the `awsvpc` network mode receive a local IPv4 address within a predefined link\-local address range\. When a container queries the metadata endpoint, the Amazon ECS container agent can determine which task the container belongs to based on its unique IP address, and metadata and stats for that task are returned\.

## Enabling task metadata<a name="task-metadata-endpoint-v2-enable"></a>

The task metadata version 2 feature is enabled by default for the following:
+ Tasks using the Fargate launch type that use platform version v1\.1\.0 or later\. For more information, see [AWS Fargate platform versions](platform_versions.md)\.
+ Tasks using the EC2 launch type that also use the `awsvpc` network mode and are launched on Amazon EC2 infrastructure running at least version 1\.17\.0 of the Amazon ECS container agent\. For more information, see [Amazon ECS Container Agent Versions](ecs-agent-versions.md)\.

You can add support for this feature on older container instances by updating the agent to the latest version\. For more information, see [Updating the Amazon ECS container agent](ecs-agent-update.md)\.

## Task metadata endpoint paths<a name="task-metadata-endpoint-v2-paths"></a>

The following API endpoints are available to containers:

`169.254.170.2/v2/metadata`  
This endpoint returns metadata JSON for the task, including a list of the container IDs and names for all of the containers associated with the task\. For more information about the response for this endpoint, see [Task Metadata JSON Response](#task-metadata-endpoint-v2-response)\.

`169.254.170.2/v2/metadata/<container-id>`  
This endpoint returns metadata JSON for the specified Docker container ID\.

`169.254.170.2/v2/metadata/taskWithTags`  
This path returns the metadata for the task included in the `/task` endpoint in addition to the task and container instance tags that can be retrieved using the `ListTagsForResource` API\. 

`169.254.170.2/v2/stats`  
This endpoint returns Docker stats JSON for all of the containers associated with the task\. For more information about each of the returned stats, see [ContainerStats](https://docs.docker.com/engine/api/v1.30/#operation/ContainerStats) in the Docker API documentation\. 

`169.254.170.2/v2/stats/<container-id>`  
This endpoint returns Docker stats JSON for the specified Docker container ID\. For more information about each of the returned stats, see [ContainerStats](https://docs.docker.com/engine/api/v1.30/#operation/ContainerStats) in the Docker API documentation\. 

## Task Metadata JSON Response<a name="task-metadata-endpoint-v2-response"></a>

The following information is returned from the task metadata endpoint \(`169.254.170.2/v2/metadata`\) JSON response\.

`Cluster`  
The Amazon Resource Name \(ARN\) or short name of the Amazon ECS cluster to which the task belongs\.

`TaskARN`  
The full Amazon Resource Name \(ARN\) of the task to which the container belongs\.

`Family`  
The family of the Amazon ECS task definition for the task\.

`Revision`  
The revision of the Amazon ECS task definition for the task\.

`DesiredStatus`  
The desired status for the task from Amazon ECS\.

`KnownStatus`  
The known status for the task from Amazon ECS\.

`Limits`  
The resource limits specified at the task level \(such as CPU and memory\)\. This parameter is omitted if no resource limits are defined\.

`PullStartedAt`  
The timestamp for when the first container image pull began\.

`PullStoppedAt`  
The timestamp for when the last container image pull finished\.

`AvailabilityZone`  
The Availability Zone the task is in\.  
The Availability Zone metadata is only available for Fargate tasks using platform version 1\.4 or later\.

`Containers`  
A list of container metadata for each container associated with the task\.    
`DockerId`  
The Docker ID for the container\.  
`Name`  
The name of the container as specified in the task definition\.  
`DockerName`  
The name of the container supplied to Docker\. The Amazon ECS container agent generates a unique name for the container to avoid name collisions when multiple copies of the same task definition are run on a single instance\.  
`Image`  
The image for the container\.  
`ImageID`  
The SHA\-256 digest for the image\.  
`Ports`  
Any ports exposed for the container\. This parameter is omitted if there are no exposed ports\.  
`Labels`  
Any labels applied to the container\. This parameter is omitted if there are no labels applied\.  
`DesiredStatus`  
The desired status for the container from Amazon ECS\.  
`KnownStatus`  
The known status for the container from Amazon ECS\.  
`ExitCode`  
The exit code for the container\. This parameter is omitted if the container has not exited\.  
`Limits`  
The resource limits specified at the container level \(such as CPU and memory\)\. This parameter is omitted if no resource limits are defined\.  
`CreatedAt`  
The time stamp for when the container was created\. This parameter is omitted if the container has not been created yet\.  
`StartedAt`  
The time stamp for when the container started\. This parameter is omitted if the container has not started yet\.  
`FinishedAt`  
The time stamp for when the container stopped\. This parameter is omitted if the container has not stopped yet\.  
`Type`  
The type of the container\. Containers that are specified in your task definition are of type `NORMAL`\. You can ignore other container types, which are used for internal task resource provisioning by the Amazon ECS container agent\.  
`Networks`  
The network information for the container, such as the network mode and IP address\. This parameter is omitted if no network information is defined\.

`ExecutionStoppedAt`  
The time stamp for when the tasks `DesiredStatus` moved to `STOPPED`\. This occurs when an essential container moves to `STOPPED`\.

## Example Task Metadata Response<a name="task-metadata-endpoint-v2-example-task-metadata-response"></a>

The following JSON response is for a single\-container task\.

```
{
  "Cluster": "default",
  "TaskARN": "arn:aws:ecs:us-east-2:012345678910:task/9781c248-0edd-4cdb-9a93-f63cb662a5d3",
  "Family": "nginx",
  "Revision": "5",
  "DesiredStatus": "RUNNING",
  "KnownStatus": "RUNNING",
  "Containers": [
    {
      "DockerId": "731a0d6a3b4210e2448339bc7015aaa79bfe4fa256384f4102db86ef94cbbc4c",
      "Name": "~internal~ecs~pause",
      "DockerName": "ecs-nginx-5-internalecspause-acc699c0cbf2d6d11700",
      "Image": "amazon/amazon-ecs-pause:0.1.0",
      "ImageID": "",
      "Labels": {
        "com.amazonaws.ecs.cluster": "default",
        "com.amazonaws.ecs.container-name": "~internal~ecs~pause",
        "com.amazonaws.ecs.task-arn": "arn:aws:ecs:us-east-2:012345678910:task/9781c248-0edd-4cdb-9a93-f63cb662a5d3",
        "com.amazonaws.ecs.task-definition-family": "nginx",
        "com.amazonaws.ecs.task-definition-version": "5"
      },
      "DesiredStatus": "RESOURCES_PROVISIONED",
      "KnownStatus": "RESOURCES_PROVISIONED",
      "Limits": {
        "CPU": 0,
        "Memory": 0
      },
      "CreatedAt": "2018-02-01T20:55:08.366329616Z",
      "StartedAt": "2018-02-01T20:55:09.058354915Z",
      "Type": "CNI_PAUSE",
      "Networks": [
        {
          "NetworkMode": "awsvpc",
          "IPv4Addresses": [
            "10.0.2.106"
          ]
        }
      ]
    },
    {
      "DockerId": "43481a6ce4842eec8fe72fc28500c6b52edcc0917f105b83379f88cac1ff3946",
      "Name": "nginx-curl",
      "DockerName": "ecs-nginx-5-nginx-curl-ccccb9f49db0dfe0d901",
      "Image": "nrdlngr/nginx-curl",
      "ImageID": "sha256:2e00ae64383cfc865ba0a2ba37f61b50a120d2d9378559dcd458dc0de47bc165",
      "Labels": {
        "com.amazonaws.ecs.cluster": "default",
        "com.amazonaws.ecs.container-name": "nginx-curl",
        "com.amazonaws.ecs.task-arn": "arn:aws:ecs:us-east-2:012345678910:task/9781c248-0edd-4cdb-9a93-f63cb662a5d3",
        "com.amazonaws.ecs.task-definition-family": "nginx",
        "com.amazonaws.ecs.task-definition-version": "5"
      },
      "DesiredStatus": "RUNNING",
      "KnownStatus": "RUNNING",
      "Limits": {
        "CPU": 512,
        "Memory": 512
      },
      "CreatedAt": "2018-02-01T20:55:10.554941919Z",
      "StartedAt": "2018-02-01T20:55:11.064236631Z",
      "Type": "NORMAL",
      "Networks": [
        {
          "NetworkMode": "awsvpc",
          "IPv4Addresses": [
            "10.0.2.106"
          ]
        }
      ]
    }
  ],
  "PullStartedAt": "2018-02-01T20:55:09.372495529Z",
  "PullStoppedAt": "2018-02-01T20:55:10.552018345Z",
  "AvailabilityZone": "us-east-2b"
}
```