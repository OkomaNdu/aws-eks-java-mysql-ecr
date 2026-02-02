#### This project is for the Devops Bootcamp Exercise for:
#### "Kubernetes on AWS - EKS" 
# aws-eks-java-mysql-ecr

**aws-eks-java-mysql-ecr** shows how you deployed a Java app on **AWS EKS** using **Fargate** (app namespace) and ran **MySQL + phpMyAdmin** on **EC2 nodes**, with images stored in **Amazon ECR** and deployments automated with **Jenkins**.

## Architecture

flowchart TB
  Dev[Developer] --> SCM[Repo]
  SCM --> Jenkins[Jenkins pipeline]
  Jenkins --> Build[Build Java app]
  Build --> DockerBuild[Docker build]
  DockerBuild --> Image["Image tag: 1.0-${BUILD_NUMBER}"]
  Image --> Push["Push to ECR"]
  Push --> ECR["ECR: 099597654282.dkr.ecr.ca-central-1.amazonaws.com/java-app"]
  ECR --> EKSCluster

  subgraph EKSCluster["Amazon EKS: my-cluster"]
    subgraph EC2Nodes["EC2 worker nodes (3)"]
      MYSQL["MySQL (Helm: my-release)"]
      PMA["phpMyAdmin (svc: phpmyadmin-service)"]
    end

    subgraph FargateProfile["Fargate: my-fargate-profile"]
      NS["Namespace: my-app"]
      JAVA["Java app (replicas: 3)"]
    end

    JAVA --> MYSQL
    PMA --> MYSQL
  end

  You["Browser"] --> PF["kubectl port-forward 8081:8081"]
  PF --> PMA
