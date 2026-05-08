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

        subgraph S_Cloud ["AWS CLOUD DEPLOYMENT"]
            direction TB
            N_Cloud_Infra["AWS Infrastructure"]:::cloud_base
            
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

    %% Styling
    classDef highlight fill:#fff,stroke:#3b82f6,color:#000,font-weight:bold
    classDef blue fill:#3b82f6,stroke:#1e3a8a,color:#fff
    classDef cloud_base fill:#f97316,stroke:#c2410c,color:#fff
