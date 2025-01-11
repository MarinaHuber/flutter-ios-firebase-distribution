# Installing Codemagic
### basic helper instructions

## <sup>Task:<sup> Distribute Flutter iOS app via Firebase Distribution App with codemagic.yaml
This is an example project that shows how to build a iOS app with Codemagic CI/CD using manual iOS code signing. Please refer to the codemagic.yaml file in the root of the project. You will need a Codemagic.io account to use this example project. You can get started for free here:

https://codemagic.io/signup

## Prerequisites for publishing
_Codemagic.io account_ (free)  
_Apple developer account_ (not free)  
_App Store Connect App creation_ (not free)  
_Firebase account_ (free)

Precise steps in order:

- [x] Download Flutter sample project (Runner) from [Codemagic](https://github.com/codemagic-ci-cd/codemagic-sample-projects/tree/main/integrations/firebase-app-distribution)

- [x] Install in Flutter firebase_core Cocoapod

- [x] With app Bundle ID create iOS app in Firebase Console and download the [GoogleService.plist](https://medium.com/flutter-community/how-to-load-firebase-config-in-codemagic-with-environment-variables-e36e0378b7e6) file + c/p App ID number

- [x] Get the SERVICE ACCOUNT setup with (Firebase Distribution App) ADMIN role -> download .json (TOKEN setup is deprecated)

- [x] Set up an App in AppStore Connect to get the number APP_STORE_ID (optional) & AD HOC iOS Distribution Certificate in Developer Account 🍏

- [x] Set up MANUAL distribution method via Environment variables (i have total of 8) & set up groups for those vars (appstore, manual_code_sign, firebase)

- [x] Start filling codemagic.yaml scripts (build IPA, code sign, firebase setup...)

- [x] Trouboleshoot if so ⤴️   

 - bash manual code signing script with ad hoc provisioning

``` exml bash for manual code signing with ad hoc provisioning

          PROFILES_HOME="$HOME/Library/MobileDevice/Provisioning Profiles"
          mkdir -p "$PROFILES_HOME"
          PROFILE_PATH="$(mktemp "$PROFILES_HOME"/$(uuidgen).mobileprovision)"
          echo ${CM_PROVISIONING_PROFILE} | base64 --decode > "$PROFILE_PATH"
          echo "Saved provisioning profile $PROFILE_PATH"

```
- script Flutter build .ipa and automatic versioning *-agvtool* new-version -all $(($BUILD_NUMBER)) responsible for increasing the version number
takes current TestFlight number to increment

```
      - name: Flutter build ipa and automatic versioning
        script: |
          # Get the next build number directly
          NEXT_BUILD_NUMBER=$(($(app-store-connect get-latest-testflight-build-number "$APP_STORE_ID") + 1))
          echo "Next build number will be: $NEXT_BUILD_NUMBER"
          
          # Set the build number in the iOS project
          cd ios
          agvtool new-version -all $NEXT_BUILD_NUMBER
          cd ..

          # Build IPA with the incremented build number
          flutter build ipa --release \
            --build-name=1.0.43 \
            --build-number=$NEXT_BUILD_NUMBER \
            --export-options-plist=/Users/builder/export_options.plist
```

- script Flutter build without App store connection incrementing from .ipa

``` 
    CURRENT_BUILD_NUMBER=$(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" Runner/Info.plist)
    NEW_BUILD_NUMBER=$((CURRENT_BUILD_NUMBER + 1))
    agvtool new-version -all $NEW_BUILD_NUMBER
    cd ..
    
    flutter build ipa --release \
      --build-number=$NEW_BUILD_NUMBER \
      --export-options-plist=/Users/builder/export_options.plist
```  

Happy building! ✨
