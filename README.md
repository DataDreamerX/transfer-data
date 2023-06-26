steps:
- task: Docker@2
  displayName: 'Login to Azure Container Registry'
  inputs:
    containerRegistry: '<registry>'
    command: 'login'
    arguments: '--username <username> --password <password>'
