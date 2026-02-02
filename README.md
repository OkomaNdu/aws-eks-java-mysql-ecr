#### This project is for the Devops Bootcamp Exercise for:
#### "Kubernetes on AWS - EKS" 
# aws-eks-java-mysql-ecr

**aws-eks-java-mysql-ecr** shows how you deployed a Java app on **AWS EKS** using **Fargate** (app namespace) and ran **MySQL + phpMyAdmin** on **EC2 nodes**, with images stored in **Amazon ECR** and deployments automated with **Jenkins**.

## Architecture

## Architecture

```mermaid
flowchart TB
  %% CI/CD + Image Registry
  Dev[Developer] --> Repo[Repo]
  Repo --> Jenkins[Jenkins pipeline]
  Jenkins --> Build[Build Java app]
  Build --> DockerBuild[Docker build]
  DockerBuild --> Tag["Image tag: 1.0-${BUILD_NUMBER}"]
  Tag --> Push[Push image]
  Push --> ECR["Amazon ECR: 099597654282.dkr.ecr.ca-central-1.amazonaws.com/java-app"]

  %% EKS Cluster
  ECR --> EKSCluster

  subgraph EKSCluster["Amazon EKS: my-cluster"]
    %% Control plane + autoscaling components
    subgraph KubeSystem["kube-system"]
      CA["Cluster Autoscaler\n(Deployment)"]
      SA["ServiceAccount: cluster-autoscaler\n(IRSA role + policy)"]
    end

    %% NodeGroup / ASG
    subgraph ASG["EC2 Node Group / Auto Scaling Group\n(min: 1, max: 3)"]
      EC2N["EC2 worker nodes"]
      MYSQL["MySQL (Helm: my-release)"]
      PMA["phpMyAdmin (svc: phpmyadmin-service)"]
    end

    %% Fargate (App Namespace)
    subgraph Fargate["Fargate: my-fargate-profile"]
      NS["Namespace: my-app"]
      JAVA["Java app (replicas: 3)"]
    end

    %% Runtime connections
    JAVA --> MYSQL
    PMA --> MYSQL

    %% Autoscaling control loop
    CA -->|watches pending pods + utilization| EC2N
    CA -->|updates desired capacity| ASG
  end

  %% Access path
  Browser["Browser"] --> PF["kubectl port-forward 8081:8081"] --> PMA


