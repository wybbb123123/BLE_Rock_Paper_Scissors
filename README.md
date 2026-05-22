# BLE Rock Paper Scissors

Android-to-Android rock paper scissors prototype built from the BLE_BBS transport path.

The app keeps the first version intentionally small:

- scans, advertises, connects, writes, and receives through the existing BLE_BBS BLE flow
- lets each device choose rock, paper, or scissors
- shows the peer as chosen before revealing the concrete move
- reveals and verifies both moves after both sides have chosen
- computes win, lose, or draw locally
- supports starting a new round

This repository is separated from BLE_BBS so the prototype can validate reuse of the BLE communication layer without replacing the BBS app.

## Android Build

```bash
cd android
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home \
ANDROID_HOME=/Users/wanghaha/bounty-work-all/android-sdk \
GRADLE_USER_HOME=/Users/wanghaha/bounty-work-all/.gradle-cache \
bash ./gradlew --no-daemon --stacktrace :app:assembleDebug
```

## Notes

The first implementation does not include countdown or random timeout moves. That keeps the prototype focused on the BLE message flow and avoids adding timer synchronization, background pause, and late-reveal edge cases before the communication layer is validated.
