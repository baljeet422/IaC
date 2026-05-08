# Terraform code 

## Maintain vpc & eks with terraform for GitOps Project

## Tools required
Terraform version 1.6.3

### Steps
* terraform init
* terraform fmt -check
* terraform validate
* terraform plan -out planfile
* terraform apply -auto-approve -input=false -parallelism=1 planfile

# Architecture Diagram
```mermaid graph TD
    %% Left Side: Developer
    subgraph Dev_IDE ["Developer IDE"]
        VS[VS Code]
    end

    VS -- "Push Code" --> IaC_Flow
    VS -- "Push Code" --> App_Flow

    %% Infrastructure as Code Section
    subgraph IaC_Flow ["Infrastructure as Code (IaC)"]
        direction TB
        G1[Git] <--> GHA1[GitHub Actions]
        G1 --> Fetch1[Fetch Code]
        Fetch1 --> TF_Plan[Terraform Plan/Test]
        TF_Plan --> PR[Pull Request Merge]
        PR --> G_Main[Git Main Branch]
        G_Main --> TF_Apply[Terraform Apply]
    end

    %% Application Build Section
    subgraph App_Flow ["Application Build & Deploy"]
        direction TB
        G2[Git] <--> GHA2[GitHub Actions]
        G2 --> Fetch2[Fetch Code]
        Fetch2 --> Maven[Maven Build]
        Sonar((SonarCloud)) -.-> Maven
        Maven --> Docker[Docker Build]
        Docker --> Helm[Helm Charts]
    end

    %% Cloud Deployment Section
    subgraph AWS_Cloud ["AWS Cloud Deployment"]
        direction TB
        subgraph VPC ["VPC Subnet"]
            ECR[(Amazon ECR)]
            EKS{Amazon EKS}
        end
        Infra[AWS Infrastructure]
    end

    %% Cross-subgraph connections
    TF_Apply --> Infra
    Docker --> ECR
    ECR -- "Pull" --> EKS
    Helm --> EKS
    TF_Plan -- "Wait for Cluster" --> G_Main

    %% Styling
    style Dev_IDE fill:#1e293b,stroke:#3b82f6,color:#fff
    style IaC_Flow fill:#0f172a,stroke:#4b5563,color:#fff
    style App_Flow fill:#0f172a,stroke:#4b5563,color:#fff
    style AWS_Cloud fill:#451a03,stroke:#f97316,color:#fff
    style VPC fill:#1c1917,stroke:#9ca3af,color:#fff
