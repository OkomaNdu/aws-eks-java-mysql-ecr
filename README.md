#### This project is for the Devops Bootcamp Exercise for:
#### "Kubernetes on AWS - EKS" 
# aws-eks-java-mysql-ecr

**aws-eks-java-mysql-ecr** shows how you deployed a Java app on **AWS EKS** using **Fargate** (app namespace) and ran **MySQL + phpMyAdmin** on **EC2 nodes**, with images stored in **Amazon ECR** and deployments automated with **Jenkins**.

## Architecture

```mermaid
flowchart TB
  Dev[Developer] --> SCM[Repo]
  SCM --> Jenkins[Jenkins pipeline]
  Jenkins --> Img[Build image]
  Img --> ECR[Amazon ECR (TBD repo)]
  ECR --> EKS[(EKS: my-cluster)]

  subgraph EKS[(EKS: my-cluster)]
    subgraph EC2[EC2 worker nodes (3)]
      MYSQL[MySQL (Helm: my-release)]
      PMA[phpMyAdmin (svc: phpmyadmin-service)]
    end
    subgraph Fargate[Fargate: my-fargate-profile]
      NS[Namespace: my-app]
      JAVA[Java app (replicas: 3)]
    end
    JAVA --> MYSQL
    PMA --> MYSQL
  end

  You[Browser] -->|kubectl port-forward 8081| PMA
