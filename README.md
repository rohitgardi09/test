# Epay_Merchant_Service

Merchant service provides capability as listed below
   - Secret key and encryption key geneartion, used to encrypt / decrypt token.
   - Bank account configuration for merchant.
   - User creation for a portal.
   

## Prerequiste - development environment
  - Open JDK v21
  - SpringBoot v3.5
  - Gradle v8.9 
  - IntelliJ Idea 
  - Sql developer
  - Windows OS
  - Linux OS

## Clone project
```
cd <your_project_folder>
git clone https://gitlab.epay.sbi/epay/epay_merchant_service.git
```

## Branches
```

develop is the development branch.

main is the production branch.

No other permanent branches should be created in the main repository, you can create feature branches but they should get merged with the develop.

Steps to work with feature branch

To start working on a new feature, create a new branch, name should be featute/<RepositoryInitials-IssueNumber_FeatureName>. (ie. feature/<FEATURE-NAME>)
  
 Example: feature/EMS-148_documentation-update
    Repository Initials: EMS
    Issue Number: 148
    FeatureName: documentation-update
```

## Test and Deploy
- Merchant service is deployed once both CI/CD jobs state is successful.
- Each build is scanned for vulnerabilty, tools like FortifyScan and SAST (Static Application Security Testing) to check source code for vulnerabilities.
 

## License
 All content in this repository, unless otherwise stated, is Copyright © State Bank Of India. All rights reserved.
The full license text is provided in `license.txt` in `documents` folder.

