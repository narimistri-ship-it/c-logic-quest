
name: Build C Logic Quest APK

on:
  workflow_dispatch:
  push:
    paths:
      - 'CLogicQuestAndroid_Starter.zip'
      - '.github/workflows/build-apk.yml'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install Cordova
        run: npm install -g cordova

      - name: Extract game files
        run: |
          mkdir -p extracted www
          unzip -q CLogicQuestAndroid_Starter.zip -d extracted
          INDEX=$(find extracted -type f -name index.html -print -quit)
          if [ -z "$INDEX" ]; then
            echo "ERROR: No index.html found in ZIP"
            exit 1
          fi
          cp -a "$(dirname "$INDEX")"/. www/
          test -f www/index.html

      - name: Create Android app
        run: |
          cordova create app com.clogicquest.game "C Logic Quest"
          cp -a www/. app/www/
          cd app
          cordova platform add android
          cordova build android --debug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: C-Logic-Quest-APK
          path: app/platforms/android/app/build/outputs/apk/debug/app-debug.apk
          if-no-files-found: error
          
