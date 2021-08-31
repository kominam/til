### Một vài khái niệm
- ECS là 1 dịch vụ giúp đơn giản hóa việc run, stop, và quản lý containers trong 1 cụm cluster
- Container sẽ được định nghĩa trong 1 task definition - là nơi bạn run các task cá nhân or 1 service. Run theo kiểu serverless và được quản lý bởi aws fargate hoặc trên EC2
- Có thể launch/stop container bằng API
- Phân bổ được các container dựa vào tài nguyên bạn cần, isolation policies và availability requirement
- Không cần quản lý cluster hay scale hạ tầng

### Tính năng
- Regional service - đơn giản là running container với 1 HA across multi AZ trong 1 region
- Có thể tạo 1 cluster cùng với tạo mới VPC or chọn VPC đã tồn tại.
- Sau khi Cluster dc tạo, bạn có thể tạo task definition - là định nghĩa container image dùng để run trên cluster của bạn
- Task definition dùng để run task hoặc tạo service
- Container image lưu trữ và đc kéo về từ container registries (ECR)

![ECS Overview](https://jayendrapatil.com/wp-content/uploads/2017/08/overview-ecs.png)
### Container và Image
- Để deploy app trên ESC, ứng dụng của bạn cần phải run kiểu container. Một container là đơn vị chuẩn hóa của software, nó chứa mọi thứ mà app bạn cần để có thể chạy được: Code, runtime, system tools, system libraries ...
- Các container được tạo từ 1 read-only template (hay còn gọi template đó là **image**)
- Images thường được build từ một Docker file, là 1 file text chỉ định các thành phần chứa trong container. Các images sau khi build và được chứa trong 1 **Registry** - là nơi có thể download và run trên cluster.
- ECS có thể config để truy cập vào private Docker image registry trong cùng một VPC, Docker hub hoặc được tích hợp với Elastic Container Registry (ECR)

![Diagram showing Docker image creation and registration within an Amazon ECS
environment.](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-containers.png)
### Task Definition
- Để chuẩn bị run app trên ECS, bạn cần tạo 1 task definition.
- Task definition là 1 file text - chuẩn json, mô tả 1 or nhiều container (tối đa là 10). Có thể coi đây là bản thiết kế cho app nhé
- Task definition chỉ định các parameter cho app của bạn. Ví dụ TD chứa các param mà container sử dụng, port mà bạn muốn open, data volume sử dụng với container

``` JSON
{
  "containerDefinitions": [
    {
      "name": "server-api",
      "image": "rails-6",
      "cpu": 0,
      "memoryReservation": 800,
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "entryPoint": [
        "sh",
        "-c"
      ],
      "command": ["RAILS_ENV=staging bundle exec rails db:migrate assets:precompile; RAILS_ENV=staging bundle exec rails s -p 3000 -b 0.0.0.0"],
      "environment": [
        {
          "name": "RAILS_LOG_TO_STDOUT",
          "value": "true"
        },
        {
          "name": "RAILS_ENV",
          "value": "staging"
        },
        {
          "name": "RAILS_SERVE_STATIC_FILES",
          "value": "true"
        }
    ],
    "mountPoints": [],
    "volumesFrom": [],
    "workingDirectory": "/app",
    "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/server-api",
          "awslogs-region": "ap-northeast-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ],
  "family": "server-api",
  "cpu": "512",
  "memory": "1024",
  "taskRoleArn": "arn:aws:iam::123123:role/ecsTaskExecutionRole",
  "executionRoleArn": "arn:aws:iam::123123:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "volumes": [],
  "placementConstraints": [],
  "requiresCompatibilities": [
    "FARGATE"
  ]
}
```
### Task và scheduling
- Một task là việc khởi tạo 1 task definition với 1 cluster. Sau khi tạo 1 TD, bạn cần chỉ định số lượng task sẽ run trên cluster.
- Task sheduler phụ trách việc placing task trong your cluster.
![](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-service.png)

### Cluster
- Cluster là 1 logical group của các task hoặc service. Bạn có thể run trên 1 hoặc nhiều EC2/Fargate. Khi run trên Fargate thì cluster sẽ do Fargate quản lý
- Default ECS tạo cluster cho bạn, bạn có thể tạo nhiều cluster để riêng biệt các resource với nhau
### Container agent
Container agent run trên mội container instance trong cluster. Nó gửi thông tin về resource của các task đang runnning ... tới ECS, và thực hiện start/stop task khi có request từ ECS
![](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/images/overview-clusteragent.png)
### ECS vs Elastic Beanstalk
- ECS cho phép bạn kiểm soát chi tiết/sát sao hơn với application architecture
- Elastic Beanstalk là ý tưởng để tận dụng tính năng của container nhưng chỉ ở mức đơn giản hóa việc deploy từ development và production thông qua việc upload container image.
- Elastic Beanstalk thiên hướng là application management platform giúp người vận hành có thể deploy và scale dễ dàng.
- Elastic Beanstalk tối giản các chi tiết và tự động handle tất cả như  provisioning ECS cluster, balancing load, auto-scaling, monitoring, và placing the containers across the cluster.

### Notes
- Có thể share volume giữa các container trong cùng 1 task sử dụng command VOLUME
- dùng awslog để put log lên cloudwatch, ở trong /dev/stdout và /dev/stderror. Chú ý link tới 2 file này khi build image trong Docker file
- Nên tách task role và task execute role