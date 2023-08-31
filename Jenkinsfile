pipeline {

  agent any

  options {
    timestamps()
  }
  environment {
    APP_ID = "com.example.test123"
  }

  stages {
    stage('Local unit test') {
      agent {
        dockerfile {
          filename 'SDK_Dockerfile'
          dir 'cicd/androidsdk'
          reuseNode true
          args "-v $HOME/.gradle:/root/.gradle -v $HOME/cicd/android_01:/application"
        }
      }
      steps {
          sh './gradlew test'
      }
    }

    stage('Integration test') {
      environment {
        ANDROID_ADB_SERVER_ADDRESS = "host.docker.internal"
      }
      agent {
        dockerfile {
          filename 'SDK_Dockerfile'
          dir 'cicd/androidsdk'
          reuseNode true
          args "-v $HOME/.gradle:/root/.gradle -v $HOME/cicd/android_01:/application"
        }
      }
      steps {
          sh "adb devices"
          sh "./gradlew assembleDebug"
          sh "adb install -r -d app/build/outputs/apk/debug/app-debug.apk"
          sh "adb shell pm clear $APP_ID"
          sh "./gradlew assembleAndroidTest"
          sh "adb install -r -d app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk"
          sh "adb shell am instrument -w \"${APP_ID}.test/androidx.test.runner.AndroidJUnitRunner\""
      }
    }

    stage("Appium Test") {
      environment {
        ANDROID_ADB_SERVER_ADDRESS = "host.docker.internal"
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
      }
      agent {
        dockerfile {
          filename 'Appium_Dockerfile'
          dir 'cicd/androidsdk'
          reuseNode true
          args "-v $HOME/.gradle:/root/.gradle -v $HOME/cicd/android_01:/application"
        }
      }
      steps {
        sh "./gradlew assembleDebug"
        sh "adb install -r app/build/outputs/apk/debug/app-debug.apk"
        sh "cd appiumTest && npm install && npm run test-ci-cd"
      }
    }

    stage('Build') {
      agent {
        dockerfile {
          filename 'SDK_Dockerfile'
          dir 'cicd/androidsdk'
          reuseNode true
          args "-v $HOME/.gradle:/root/.gradle -v $HOME/cicd/android_01:/application"
        }
      }
      steps {
          sh './gradlew assembleRelease'
          sh 'ls -R app/build/outputs/apk'
          stash name: "apk", includes: 'app/build/outputs/apk/release/*.apk', allowEmpty: true
          archiveArtifacts artifacts: 'app/build/outputs/apk/release/*.apk', fingerprint: true
      }
    }
  }
}
