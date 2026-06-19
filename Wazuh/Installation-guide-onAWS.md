# Steps In SettingUp Wazuh Manager On AWS


## Step-1:

### Create an AWS EC2 Instance

- Log in to [Amazon Web Services (AWS)](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Feu-north-1.console.aws.amazon.com%2Fvpcconsole%2Fhome%3Fca-oauth-flow-id%3D3857%26hashArgs%3D%2523TrafficMirrorTargets%253A%26isauthcode%3Dtrue%26oauthStart%3D1781840069323%26region%3Deu-north-1%26state%3DhashArgsFromTB_eu-north-1_773049dfbd7e0999&client_id=arn%3Aaws%3Asignin%3A%3A%3Aconsole%2Fvpcconsole&forceMobileApp=0&code_challenge=c3M1eCgXmsWVjVGqLkJRksX5Tp9QbVgNrX0UgzvhvyY&code_challenge_method=SHA-256)
- Go to EC2 → Launch Instance
- Choose size:
  - Ubuntu Server 22.04 LTS (common choice)
  - or Amazon Linux
- Choose size:
  - Lab: t3.medium (minimun recommended)
  - Production: larger depending on endpoints
- Create/download a key paire(.pem)
- Configure Security Group rules:
  
