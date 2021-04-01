# capacitor-biometric-auth-demo

This [Ionic](https://ionicframework.com) application provides a demo of all of the capacibilities of the [capacitor-biometric-auth](https://github.com/aparajita/capacitor-biometric-auth) Capacitor plugin.

## Installation

```shell
git clone https://github.com/aparajita/capacitor-biometric-auth-demo.git
cd capacitor-biometric-auth-demo
pnpm install  # npm install, yarn
pnpm build
cap sync
```

## Usage

### Web

To launch the demo in a browser:

```shell
pnpm serve
```

Once the demo is open, select a biometry type from the menu and click Authenticate. A browser confirm will appear. Clicking OK simulates successful authentication, clicking Cancel simulates user cancellation.

### Native

To launch the demo on iOS or Android:

```shell
pnpm ios
pnpm android
```

Once Xcode/Android Studio opens, select the device or simulator you wish to run the demo on. When the demo app opens, the supported biometry type and status is displayed at the top. On iOS, if biometry is supported but not available in a simulator, you have to manually enroll in biometry:

- Select via the Features > Touch/Face ID > Enrolled.
- Suspend and resume the demo app. You should see that biometry is now available.

You may set all of the available [AuthenticateOptions](https://github.com/aparajita/capacitor-biometric-auth/blob/main/src/definitions.ts#L41) for the current platform via inputs.
