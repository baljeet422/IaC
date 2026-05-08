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
    %% Global Theme Settings
    %% This sets the background and line colors for dark mode
    
    subgraph S_Dev ["Developer IDE"]
        L_VS["VS Code"]
    end

    subgraph S_CICD ["CI/CD & Infrastructure"]
        direction TB

        subgraph S_IaC ["IaC - Terraform Workflow"]
            direction TB
            N_IaC_G["Git"]
            N_IaC_GHA["GitHub Actions Runner"]:::highlight
            N_IaC_Fetch["Fetch code"]
            N_IaC_Plan["Terraform Plan/Test"]
            N_IaC_PR["Pull Request Merge"]
            N_IaC_Git_Main["Git Main branch"]
            N_IaC_Apply["Terraform Apply"]:::blue

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
            N_App_GHA["GitHub Actions Runner"]:::highlight
            N_App_Fetch["Fetch code"]
            N_App_Maven["Maven Build"]
            N_App_Sonar(("SonarCloud"))
            N_App_Docker["Docker Build"]:::blue
            N_App_Helm["Helm Charts"]

            N_App_G <--> N_App_GHA
            N_App_G --> N_App_Fetch
            N_App_Fetch --> N_App_Maven
            N_App_Maven --> N_App_Docker
            N_App_Docker --> N_App_Helm
            N_App_Sonar -- "Static Analysis" --> N_App_Maven
        end

        subgraph S_Cloud ["AWS Cloud Deployment"]
            direction TB
            N_Cloud_Infra["AWS Infrastructure"]:::orange
            
            subgraph S_VPC ["VPC Subnet"]
                direction TB
                N_Cloud_ECR[("Amazon ECR")]
                N_Cloud_EKS{"Amazon EKS"}
                N_Cloud_ECR --> |"Pull"| N_Cloud_EKS
            end
        end
    end

    %% Connections
    L_VS -- "Push Code" --> N_IaC_G
    L_VS -- "Push Code" --> N_App_G
    N_IaC_Apply --> N_Cloud_Infra
    N_App_Docker --> N_Cloud_ECR
    N_App_Helm --> N_Cloud_EKS

    %% Style Definitions for Dark Theme
    classDef default fill:#111827,stroke:#374151,color:#d1d5db,stroke-width:1px;
    classDef highlight fill:#1f2937,stroke:#60a5fa,color:#fff,stroke-width:2px;
    classDef blue fill:#2563eb,stroke:#60a5fa,color:#fff,font-weight:bold;
    classDef orange fill:#7c2d12,stroke:#fb923c,color:#fff;
    
    %% Subgraph Box Styling
    style S_Dev fill:#0f172a,stroke:#1e293b,color:#94a3b8
    style S_CICD fill:#030712,stroke:#1f2937,color:#94a3b8
    style S_IaC fill:#111827,stroke:#334155,color:#cbd5e1
    style S_App fill:#111827,stroke:#334155,color:#cbd5e1
    style S_Cloud fill:#1c1917,stroke:#44403c,color:#fde047
    style S_VPC fill:#292524,stroke:#57534e,color:#fff
