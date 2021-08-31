### Install 
``` bash
# install aws cli v2
curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip"](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip%22 "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip%22") -o "awscliv2.zip"
sudo apt install unzip python-is-python3 python3-venv
unzip awscli-bundle.zip
sudo ./aws/install

# install SSM
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/ubuntu_64bit/session-manager-plugin.deb" -o "session-manager-plugin.deb"
sudo dpkg -i session-manager-plugin.deb
session-manager-plugin
```
### Query ECS task
``` bash
export CLUSTER=<cluster-name>
export REGION=ap-northeast-1
export SERVICE=<service-name>

TASK_ID=$(aws ecs list-tasks --cluster $CLUSTER --service $SERVICE --region $REGION --output text --query 'taskArns[0]')

aws ecs execute-command --cluster $CLUSTER --region $REGION --task $TASK_ID --command "sh" --interactive --container $CONTAINER_NAME

# enable execute command with option
--enable-execute-command
```
### ECS
``` bash
# login ecr
aws ecr get-login-password --region $REGION  | docker login --username AWS --password-stdin $ECR_ENDPOINT
# build image with multi-stage (optional)
docker build --target=$STAGE -t $ECR_IMAGE $SOURCE_DIR
# create image tag
docker tag $ECR_IMAGE:latest $ECR_ENDPOINT/$ECR_IMAGE:latest
# push image to remote repository
docker push $ECR_ENDPOINT/$ECR_IMAGE:latest
# update service
aws ecs update-service --cluster $CLUSTER --service $SERVICE --task-definition $SERVICE --force-new-deployment --region $REGION
```
### Query active session SSM
``` bash
aws ssm describe-sessions --state Active
aws ssm describe-sessions --state Active --output json --query "Sessions[0].SessionId"
aws ssm terminate-session --session-id <Session-ID>
```
### Copy file to S3
``` bash
aws s3 cp application.yml s3://secret/application.yml
```
### Codedeploy
**1. Revision**
``` bash
cd codedeploy/
aws deploy push --region ap-northeast-1 --profile <project-name> --application-name <project-ec2-app-name> --s3-location s3://codedeploy/codedeploy-prod.zip --source .
```
**2. Deploy app**
``` bash
aws deploy create-deployment --region ap-northeast-1 --profile <project-name> --application-name <project-ec2-app-name> --deployment-group-name <project-deployment-group-admin> --s3-location bucket=codedeploy-prod,key=codedeploy-prod.zip,bundleType=zip
```
### Lambda
1. Tạo Dockerfile để test khi implement với Ruby
``` Dockerfile
FROM lambci/lambda:build-ruby2.7
RUN yum -y install mysql-devel
RUN gem update bundler

# copy purpose (skip it if you run in step 4)
COPY Gemfile Gemfile.lock ./
RUN bundle install --path vendor/bundle --job 4
CMD "/bin/bash"
```
2. Build image
``` bash
docker build -t lambda-ruby2.7 .
```
3. Mount code and run
``` bash
docker run --rm -it -v $PWD:/var/task -w /var/task lambda-ruby2.7
```
4. Execute to container and install bundle
``` bash
mkdir lib
cp /usr/lib64/mysql/. lib/
bundle config --local build.mysql --with-mysql-config=/usr/local/mysql/bin/mysql_config
RUN bundle install --path vendor/bundle --job 4
```
**Note:** Other way, copy vendor and mysql's so file from container to host
``` bash
docker run --rm lambda-ruby2.7
docker cp <container-name>:/var/task/vendor/ ./
docker cp <container-name>:/var/task/usr/lib64/mysql/. ./lib
```
5. Run and test 
``` bash
docker run -e DB_HOST=db -e DB_USERNAME=root -e DB_PASSWORD=Aa@123456 -e DB_NAME=app_development --network=default_app --rm -v $PWD:/var/task lambci/lambda:ruby2.7 main.handler
```