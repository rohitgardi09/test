Exception_Tracker_Service

Exception Tracker Service provides capability as listed below:

- AOP based exception tracking
- Asynchronous batch exception logging
- Buffered queue processing
- MDC logging support
- Correlation ID tracking
- Exception persistence into database

---

Prerequisite - development environment

- Open JDK v21
- SpringBoot v3.5
- Gradle v8.9
- IntelliJ IDEA
- SQL Developer
- Windows OS
- Linux OS

---

Clone project

cd <your_project_folder>
git clone <repository-url>

---

Branches

develop is the development branch.

main is the production branch.

No other permanent branches should be created in the main repository, feature branches can be created and later merged into develop branch.

Steps to work with feature branch

To start working on a new feature, create a new branch.

Branch naming format:
feature/<RepositoryInitials-IssueNumber_FeatureName>

Example:
feature/EJU-148_exception-tracker-service

---

Exception Tracking Flow

1. Exception occurs in application
2. "@TrackException" aspect intercepts exception
3. Exception details converted into "ExceptionLogDto"
4. DTO pushed into queue
5. "ExceptionQueueService" prepares MDC values
6. "ExceptionBufferedRepository" saves batch asynchronously into DB

---

Build Project

./gradlew clean build

Windows:

gradlew.bat clean build

---

Run Project

./gradlew bootRun

Windows:

gradlew.bat bootRun

---

Author

Rohit Gardi (V1024113)

State Bank of India

---

Version

1.0

---

License

All content in this repository, unless otherwise stated, is Copyright © State Bank Of India.
All rights reserved.




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

