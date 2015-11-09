# Tumba way of building and distributing iOS apps

### Prerequisites
 - Github repo
 - CircleCI account 
 - Fastlane automation bundle
 - Apple developper account: 
   - Code signing identity/Signing certificate
   - App ID
   - Team ID
 
 ## Continuous Integration
 - `fastlane init` - creates the fastlane directory structure in the project repo.
 - add team_name, team_id and app_identifier in the Appfile
 - Configure your pipelines in the Fastfile-aka lanes. For Developmeny we have:
   - prerequisites - under `before_all` - i.e. 'cocoapods', increment_build_number etc.
   - test lane - runs xctest 
 - circle.yml configures the build environment and calls the build/test commands:
   - specify Xcode version on the CircleCI side
   - install fastlane tool chain
   - call the appropriate lane - in this case `fastlane test`
 
 ## Continuous delivery
  - ### Test phase - Crashlitics
    - Fastlane alpha:
      - sigh block - downloads the adhoc provisioning profile or, if missing, creates it
      - gym block - creates the .ipa file and signs it 
      - crashlitics block - submits the app to Fabric.io for testing
    - circle.yml 
      - imports the Distribution code signing identity into the CirleCI environment
      - calls `fastlane alpha`
  - ### UAT and Release
    - Fastlane beta;
      - sigh block - downloads the distribution provisioning profile or, if missing, creates it
      - snapshot block - creates snapshots of the app to upload to TestFlight
      - gym block - builds and signs the app and .ipa file 
      - deliver - upload app meta data and the app to TestFlight. 
    
  
