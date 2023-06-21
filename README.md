parameters:
- name: environment
  displayName: 'Environment'
  type: string
  default: 'npe'
  values:
    - 'npe'
    - 'pre-prod'

variables:
- name: npeVariableGroup
  value: 'variableGroupNPE'
- name: preProdVariableGroup
  value: 'variableGroupPreProd'

stages:
- stage: DeployNPE
  displayName: 'Deploy to NPE'
  condition: eq(parameters.environment, 'npe')
  variables:
    - group: $(npeVariableGroup)
  jobs:
  - job: ExampleJob
    displayName: 'Example Job'
    steps:
    - script: |
        echo "Deploying to NPE"
        # Add your deployment steps for NPE

- stage: DeployPreProd
  displayName: 'Deploy to Pre-Prod'
  condition: eq(parameters.environment, 'pre-prod')
  variables:
    - group: $(preProdVariableGroup)
  jobs:
  - job: ExampleJob
    displayName: 'Example Job'
    steps:
    - script: |
        echo "Deploying to Pre-Prod"
        # Add your deployment steps for Pre-Prod
