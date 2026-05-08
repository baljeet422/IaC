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
```mermaid 
graph TD
    %% Main Subgraphs for layout
    subgraph S_Dev ["DEVELOPER IDE"]
        L_VS["VS Code (Developer IDE)"]
    end

    subgraph S_CICD ["CI/CD & INFRASTRUCTURE"]
        direction TB

        subgraph S_IaC ["IaC - Terraform Workflow"]
            direction TB
            N_IaC_G["Git"]
            N_IaC_GHA["GitHub Actions Runner"]:::action_node
            N_IaC_Fetch["Fetch code"]
            N_IaC_Plan["Terraform Plan/Test"]
            N_IaC_PR["Pull Request Merge"]
            N_IaC_Git_Main["Git Main branch"]
            N_IaC_Apply["Terraform Apply"]:::blue_node

            N_IaC_G <--> N_IaC_GHA
            N_IaC_G --> N_IaC_Fetch
            N_IaC_Fetch --> N_IaC_Plan
            N_IaC_Plan --> N_IaC_PR
            N_IaC_PR --> N_IaC_Git_Main
            N_IaC_Git_Main --> N_IaC_Apply
            N_IaC_Plan --> |"Wait for Cluster"| N_IaC_Git_Main
        end

        subgraph S_App ["App Build & Deploy Workflow"]
            direction TB
            N_App_G["Git"]
            N_App_GHA["GitHub Actions Runner"]:::action_node
            N_App_Fetch["Fetch code"]
            N_App_Maven["Maven Build"]
            N_App_Sonar(("SonarCloud"))
            N_App_Docker["Docker Build"]:::blue_node
            N_App_Helm["Helm Charts"]

            N_App_G <--> N_App_GHA
            N_App_G --> N_App_Fetch
            N_App_Fetch --> N_App_Maven
            N_App_Maven --> N_App_Docker
            N_App_Docker --> N_App_Helm
            N_App_Sonar -- "Static Analysis" --> N_App_Maven
        end

        subgraph S_Cloud ["AWS CLOUD DEPLOYMENT"]
            direction TB
            N_Cloud_Infra["AWS Infrastructure"]:::orange_node
            
            subgraph S_VPC ["VPC Subnet"]
                direction TB
                N_Cloud_ECR[("Amazon ECR")]
                N_Cloud_EKS{"Amazon EKS"}
                N_Cloud_ECR --> |"Pull"| N_Cloud_EKS
            end
        end
    end

    %% Inter-subgraph connections
    L_VS -- "Push Code" --> N_IaC_G
    L_VS -- "Push Code" --> N_App_G
    N_IaC_Apply --> N_Cloud_Infra
    N_App_Docker --> N_Cloud_ECR
    N_App_Helm --> N_Cloud_EKS

    %% --- STYLING BLOCK ---
    
    %% 1. Style the containers (Subgraphs) - Light fill for visibility
    style S_Dev fill:#e2e8f0,stroke:#64748b,stroke-width:2px,color:#0f172a
    style S_CICD fill:#1e293b,stroke:#334155,stroke-width:2px,color:#f8fafc
    style S_IaC fill:#f1f5f9,stroke:#94a3b8,color:#0f172a
    style S_App fill:#f1f5f9,stroke:#94a3b8,color:#0f172a
    style S_Cloud fill:#ffedd5,stroke:#fb923c,color:#7c2d12
    style S_VPC fill:#fff7ed,stroke:#fdba74,color:#7c2d12

    %% 2. Style individual nodes for contrast
    classDef action_node fill:#3b82f6,stroke:#1e3a8a,color:#fff,font-weight:bold
    classDef blue_node fill:#1e40af,stroke:#1e3a8a,color:#fff
    classDef orange_node fill:#ea580c,stroke:#9a3412,color:#fff
    
    %% Default node style
    style L_VS fill:#ffffff,stroke:#334155,color:#0f172a
    style N_IaC_G fill:#ffffff,stroke:#334155,color:#0f172a
    style N_App_G fill:#ffffff,stroke:#334155,color:#0f172a