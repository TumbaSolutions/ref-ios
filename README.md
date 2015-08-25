# iOS build and distribution automation 

## This is a how to automate iOS CI-CD. 

### How to build, test and sign in the Development stage
At this stage archiving the app is not necessary, that is way we can the standard Xcode tool xctool.

- 1. Create the below variables in circleci.com:
     GYM_SCHEME
     GYM_WORKSPACE

- 2. Create circle.yml with the below standard configuration:
- 
```
test:
  override:
    - xctool
      -reporter pretty
      -reporter junit:$CIRCLE_TEST_REPORTS/xcode/results.xml
      -reporter plain:$CIRCLE_ARTIFACTS/xctool.log
      CODE_SIGNING_REQUIRED=NO
      CODE_SIGN_IDENTITY=
      PROVISIONING_PROFILE=
      -destination 'platform=iOS Simulator,OS=8.1,name=iPhone 6'
      -sdk iphonesimulator
      -workspace $GYM_WORKSPACE
      -scheme $GYM_SCHEME
      build test
```

- 3. This alone should be enough to build and test your code. The default Xcode installation provided by circleci.com is 6.1
 
### How to build, archive and distribute in the Distribution stage 
This stage requires archiving the code, signing it and uploading it to iTunes connect. Since Fastlane`s 'gym' is doing this job much fadter and cleaner it is suggested to use it.
'gym' inherits all the projects settings directly from the Xcode project. This means that the code signing identity, provisioning profile and bundle identifier are set properly beforehand. The version number of the app should be incremented before git push, else iTunes connect will reject it. 

- 1. Export a Distribution code signing identity through Xcode. 
   !!!Absolutely mendatory is to password protect it!!!

- 2. Include the code signing identity in the githubrepo(not very safe) or upload it to a cloud storage(additional configuration required)

- 3. Set code signing resuorce rules path to $(SDKROOT)/ResourceRules.plist in Xcode. Fixes a Xcode bug.

- 4. Create a Gemfile with the below content. It is a library required by the 'cupertino' gem:
```
source 'https://rubygems.org'
gem 'highline', '>=1.7.1'
```

- 5. Create the below variables: 
     - APPLE_USER - developer portal usr 
     - APPLE_PASS - developer portal pass
     - CSI_PASS - cosw signing identity export pass
     - AD_HOC_PROVISIONING_PROFILE - the name of the profile as is in developer portal
     - GYM_CODE_SIGNING_IDENTITY - the name of the code signing identity. ID is preferred.
     - GYM_OUTPUT_NAME - name of the archive. No need to end it with .ipa 
     - GYM_OUTPUT_DIRECTORY - name of the directory to hold the archive
     - GYM_WORKSPACE - the workspace 
     - GYM_SCHEME - the scheme 
     - DELIVER_USER - iTunes connect user for 'pilot' to use 
     - DELIVER_PASSWORD - iTunes Connect pass

- 6. Create the circle.yml file with the below content. What ech command does is explained inline: 
```
dependencies:
  pre:
    # Creates the direcory for the .ipa files.
    - mkdir -p `pwd`/build
    # Circle`s ruby version error installing custom tools
    - curl -sSL https://get.rvm.io | bash -s stable --ruby
    - sudo gem install bundler
    - bundle install
    # In order - install a tool to manage the provisioning profiles, a tool to build and archive the code and a tool to upload to iTunes Connect:
    - sudo gem install cupertino
    - sudo gem install gym
    - sudo gem install pilot
    # Imports the code signing identity. The export password is set as a variable 
    - security import ad_hoc.p12 -k ~/Library/Keychains/login.keychain -P $CSI_PASS -T /usr/bin/codesign -A
    # Downloads the ad hoc provisioning profile:
    - sudo ios profiles:download "$AD_HOC_PROVISIONING_PROFILE" --type distribution -u "$APPLE_USER" -p "$APPLE_PASS"
    # Verifies the code signing identity was installed properly
    - security find-identity -v -p codesigning
    # Installs the provisioning profile
    - rm -Rf ~/Library/MobileDevice/Provisioning\ Profiles/*
    - cp *.mobileprovision  ~/Library/MobileDevice/Provisioning\ Profiles/ 
test:
  override:
    # Builds and archives the code. Configuration is retrieved from the Xcode project settings and the GYM_* variables 
    - gym
deployment:
  testflight:
    branch: submission
    commands:
      # Uploads the .ipa to iTunes Connect
      - pilot upload -u "$APPLE_USER" -i $GYM_OUTPUT_DIRECTORY/RefIOS.ipa -a 1031215666 -s
```
   
## Resources

Cupertino - https://github.com/nomad/Cupertino
Fastlane - https://github.com/nomad/Cupertino
CircleCi - https://circleci.com/docs/ios 
