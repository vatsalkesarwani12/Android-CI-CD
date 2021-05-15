# Android-CI-CD
A simple github action that runs the lint check, test check and create apk and stores them to the artifacts.

### How to use this action in my project?

- Copy the below code:
```
name: AndroidCI
on:
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # This job is for lint check
  lint:
    # This job runs on ubuntu
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the code
        uses: actions/checkout@v2

      - name: Upload html test report
        uses: actions/upload-artifact@v2
        with:
          # This name line renames the file
          name: lint.htmt
          path: app/build/reports/lint-results-debug.html

  unit-test:
    # This needs command with execute this job after lint job
    needs: [ lint ]
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the code
        uses: actions/checkout@v2
        
      # This command gives gradle permission  
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Run tests
        run: ./gradlew test

      - name: Upload test report
        uses: actions/upload-artifact@v2
        with:
          name: unit_test_report
          path: app/build/reports/tests/testDebugUnitTest/

    instrumentation-test:
      needs: [unit-test]
      runs-on: macos-latest
      steps:
        - name: Checkout the code
          uses: actions/checkout@v2
  
        - name: Run espresso tests
          uses: reactivecircus/android-emulator-runner@v2
          with:
            api-level: 29
            script: ./gradlew connectedCheck
  
        - name: Upload test report
          uses: actions/upload-artifact@v2
          with:
            name: instrumentation_test_report
            path: app/build/reports/androidTests/connected/

  package:
    needs: [ instrumentation-test ]
    name: Generate APK
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the code
        uses: actions/checkout@v2

      # You can change to you project jdk version
      - name: set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Build debug APK
        run: ./gradlew assembleDebug --stacktrace

      - name: Upload APK
        uses: actions/upload-artifact@v2
        with:
          name: AppName.apk
          path: app/build/outputs/apk/debug/app-debug.apk
```
- Create a file in the `.github/workflows` folder of your project.
- Name the file `main.yml` or any other name (The extension of the file should be .yml ).
- Make any pull request to see it in Action!

**Note:**
- To run this Action, your project must have same path as written in .yml file else you have to update them.
- To get the lint output you must have to remove /build from the .gitignore file of your project(Else upload step will cause warning and result won't be uploaded to the artificat ).

