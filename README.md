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
    %% Base colors and dark theme
    accTitle: DevSecOps GitOps Pipeline Architecture
    accDescr: A comprehensive diagram showing a multi-track GitOps pipeline with separate IaC and Application workflows leading to an AWS EKS deployment.

    %% Main Subgraphs for layout
    subgraph S_Dev ["DEVELOPER IDE (VS Code)"]
        direction TB
        L_VS[fa:fa-file-code VS Code<br/>(Developer IDE)]
    end

    subgraph S_CICD ["CI/CD & INFRASTRUCTURE"]
        direction TB

        %% Parallel workflows
        subgraph S_IaC ["INFRASTRUCTURE AS CODE (IaC) - Terraform Workflow"]
            direction TB
            
            %% Key Node Groups
            N_IaC_G[fa:fa-git-alt git]
            N_IaC_GHA[GitHub Actions <br/> Actions Runner]:::highlight
            N_IaC_Fetch[fa:fa-cloud-download-alt Fetch code]
            N_IaC_Plan[fa:fa-shield-alt Terraform <br/> Plan/Test]
            N_IaC_PR[fa:fa-code-branch Pull Request <br/> Merge]
            N_IaC_Git_Main[fa:fa-git-alt git <br/> Main branch]
            N_IaC_Apply[fa:fa-check-circle Terraform <br/> Apply]:::blue

            %% Connections
            N_IaC_G <--> N_IaC_GHA
            N_IaC_G --> N_IaC_Fetch
            N_IaC_Fetch --> N_IaC_Plan
            N_IaC_Plan --> N_IaC_PR
            N_IaC_PR --> N_IaC_Git_Main
            N_IaC_Git_Main --> N_IaC_Apply
            N_IaC_Plan --> |"Wait for Cluster"| N_IaC_Git_Main
        end

        subgraph S_App ["APPLICATION BUILD & DEPLOY - Build, Test & Deploy Workflow"]
            direction TB
            
            %% Key Node Groups
            N_App_G[fa:fa-git-alt git]
            N_App_GHA[GitHub Actions <br/> Runner]:::highlight
            N_App_Fetch[fa:fa-cloud-download-alt Fetch code]
            N_App_Maven[fa:fa-layer-group Maven Build]
            N_App_Sonar[fa:fa-sonarcloud SonarCloud]
            N_App_Docker[fa:fa-docker Docker]:::blue
            N_App_Helm[fa:fa-helm Helm Charts]

            %% Connections
            N_App_G <--> N_App_GHA
            N_App_G --> N_App_Fetch
            N_App_Fetch --> N_App_Maven
            N_App_Maven --> N_App_Docker
            N_App_Docker --> N_App_Helm
            
            %% External to Maven
            N_App_Sonar -- "Static Code Analysis" --> N_App_Maven
        end

        subgraph S_Cloud ["AWS CLOUD DEPLOYMENT"]
            direction TB
            
            %% Specific Nodes
            N_Cloud_Infra[fa:fa-aws AWS CLOUD INFRASTRUCTURE]:::cloud_base
            
            %% Subnet (a sub-group)
            subgraph S_VPC ["VPC Subnet"]
                direction TB
                N_Cloud_ECR[Amazon ECR]
                N_Cloud_EKS[Amazon EKS]
                
                N_Cloud_ECR --> |"Pull"| N_Cloud_EKS
            end
        end
    end

    %% Key Inter-Subgraph Connections
    S_Dev -- "Push Code" --> N_IaC_G
    S_Dev -- "Push Code" --> N_App_G
    
    %% Output of IaC to Cloud
    N_IaC_Apply --> N_Cloud_Infra
    
    %% Output of App to ECR
    N_App_Docker --> N_Cloud_ECR
    
    %% Helm connection
    N_App_Helm --> N_Cloud_EKS

    %% Class Definitions (Modern dark blue theme with contrasting accents)
    classDef highlight fill:#fff,stroke:#fff,color:#0f172a,stroke-width:1px,rx:5,ry:5;
    classDef blue fill:#3b82f6,stroke:#3b82f6,color:#fff,stroke-width:1px,rx:5,ry:5;
    classDef cloud_base fill:#451a03,stroke:#f97316,color:#fff,stroke-width:1px,rx:5,ry:5;
    
    %% General node styling (everything else is a default panel box)
    linkStyle default stroke:#888,stroke-width:1.5px;
    
    %% Define background and panels
    class S_Dev,S_CICD,S_IaC,S_App,S_Cloud,S_VPC panel_box;
    classDef panel_box fill:#0f172a,stroke:#4b5563,color:#fff,stroke-width:2px,rx:10,ry:10;
