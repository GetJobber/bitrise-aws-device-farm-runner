# Amazon Device Farm Runner
Deploys app to device farm and starts a test run with a preconfigured test package and device pool.

## Setup instructions
:warning: This step requires a fair amount of configuration in order to work properly.
[Please read the wiki for setup instructions](https://github.com/peartherapeutics/bitrise-aws-device-farm-runner/wiki).

## Inputs

### Required Inputs
- `access_key_id` - Access key for your AWS/IAM user. Define this as a secret environment variable in your workflow.
- `secret_access_key` - Secret key for your AWS/IAM user. Define this as a secret environment variable in your workflow.
- `device_farm_project` - Project ARNs can be obtained using the AWS CLI `devicefarm list-projects` command.
- `test_package_name` - Filename of test package to run. This should be a filename (not a path) that matches the name of the test bundle previously uploaded with the [aws-device-farm-file-deploy](https://github.com/peartherapeutics/bitrise-aws-device-farm-file-deploy) step. The most recently uploaded package with this name will be used.
- `test_type` - [See the Test.type documentation](http://docs.aws.amazon.com/devicefarm/latest/APIReference/API_Test.html#devicefarm-Type-Test-type).
- `platform` - Platform to run tests on. Options: `ios`, `android`, `ios+android`
- `build_version` - Build number

### Optional Inputs
- `billing_method` - Billing method for your test run. Use `METERED` for free tier and pay-as-you-go billing, and `UNMETERED` to use [pre-paid device slots](https://docs.aws.amazon.com/devicefarm/latest/developerguide/how-to-purchase-device-slots.html). Default: `METERED`
- `locale` - The locale as ISO language and country code to be used by the test devices. Default: `en_US`
- `filter` - Test filter. For example com.android.abc.test1. [See the ScheduleRunTest.filter documentation](https://docs.aws.amazon.com/devicefarm/latest/APIReference/API_ScheduleRunTest.html#devicefarm-Type-ScheduleRunTest-filter).
- `test_spec` - Device Farm Custom TestSpec ARN. Environment ARNs can be obtained using the AWS CLI `devicefarm list-uploads` command.
- `run_name_prefix` - If you want to specify a name, this prefix will be used followed by platform and bitrise build number.
- `aws_region` - If you want to specify a different AWS region. Leave empty to use the default config/env setting.
- `run_wait_for_results` - Whether or not to wait for the test results from device farm. If set to true, the script waits for the test run to complete on Device farm before returning success/failure. This will slow down your Bitrise runs, however allows you to make decisions in subsequent steps based on success/failure of the tests. Default: `true`
- `run_fail_on_warning` - Fail if the device farm results return result of WARNED. Depending on your tests, you may or may not wish to fail the step if Device farm returns a WARNED result. Only takes effect if `run_wait_for_results` is also enabled. Default: `true`

### iOS-specific Inputs
- `ipa_path` - IPA file path. Required for iOS runs when `ipa_package_name` is not provided. Default: `$BITRISE_IPA_PATH`
- `ipa_package_name` - Filename of IPA package to use for testing. This should be a filename (not a path) that matches the name of the IPA bundle previously uploaded with the [aws-device-farm-file-deploy](https://github.com/peartherapeutics/bitrise-aws-device-farm-file-deploy) step. The most recently uploaded package with this name will be used. If provided, this takes precedence over `ipa_path`.
- `ios_pool` - Device Farm iOS Device Pool ARN. Required for iOS runs. ARNs can be obtained using the AWS CLI `devicefarm list-device-pools` command.

### Android-specific Inputs
- `apk_path` - APK file path. Required for Android runs when `apk_package_name` is not provided. Default: `$BITRISE_SIGNED_APK_PATH`
- `apk_package_name` - Filename of APK package to use for testing. This should be a filename (not a path) that matches the name of the APK bundle previously uploaded with the [aws-device-farm-file-deploy](https://github.com/peartherapeutics/bitrise-aws-device-farm-file-deploy) step. The most recently uploaded package with this name will be used. If provided, this takes precedence over `apk_path`.
- `android_pool` - Device Farm Android Device Pool ARN. Required for Android runs. ARNs can be obtained using the AWS CLI `devicefarm list-device-pools` command.

## Outputs
- `BITRISE_DEVICEFARM_RESULTS_RAW` - The full output from the device farm run in JSON from AWS Device Farm
- `BITRISE_DEVICEFARM_RESULTS_SUMMARY` - A human-readable summary of the results suitable for feeding into something like a slack message

## Package Upload Options

This step supports two approaches for providing app packages (IPA/APK files):

### File Path Approach (Traditional)
- Use `ipa_path` for iOS apps
- Use `apk_path` for Android apps
- The step will upload the file to Device Farm before running tests
- Suitable when you have local files or files generated during the build

### Package Name Approach (New)
- Use `ipa_package_name` for iOS apps
- Use `apk_package_name` for Android apps
- The step will look up the most recently uploaded package with the specified name
- No upload is performed, saving time and bandwidth
- Requires packages to be previously uploaded using the [aws-device-farm-file-deploy](https://github.com/peartherapeutics/bitrise-aws-device-farm-file-deploy) step
- Package name parameters take precedence over file path parameters when both are provided

### Test Packages
- Always use `test_package_name` (package name approach only)
- Test packages must be previously uploaded using the aws-device-farm-file-deploy step

## How to use this Step

Can be run directly with the [bitrise CLI](https://github.com/bitrise-io/bitrise),
just `git clone` this repository, `cd` into it's folder in your Terminal/Command Line
and call `bitrise run test`.

*Check the `bitrise.yml` file for required inputs which have to be
added to your `.bitrise.secrets.yml` file!*

Step by step:

1. Open up your Terminal / Command Line
2. `git clone` the repository
3. `cd` into the directory of the step (the one you just `git clone`d)
5. Create a `.bitrise.secrets.yml` file in the same directory of `bitrise.yml` - the `.bitrise.secrets.yml` is a git ignored file, you can store your secrets in
6. Check the `bitrise.yml` file for any secret you should set in `.bitrise.secrets.yml`
  * Best practice is to mark these options with something like `# define these in your .bitrise.secrets.yml`, in the `app:envs` section.
7. Once you have all the required secret parameters in your `.bitrise.secrets.yml` you can just run this step with the [bitrise CLI](https://github.com/bitrise-io/bitrise): `bitrise run test`

An example `.bitrise.secrets.yml` file:

```
---
# These environments should NOT be checked into source control, they are used
# to populate your tests when running this step locally.
envs:
 - AWS_ACCESS_KEY: ""
 - AWS_SECRET_KEY: ""
 - DEVICE_FARM_PROJECT: ""
 - TEST_PACKAGE_NAME: "test_bundle.zip"
 - TEST_TYPE: "APPIUM_PYTHON"
 - PLATFORM: "ios+android"
 - IOS_POOL: ""
 - ANDROID_POOL: ""
 - RUN_NAME_PREFIX: "testscript"
 - AWS_REGION: "us-west-2"
 - BITRISE_IPA_PATH: ""
 - BITRISE_SIGNED_APK_PATH: ""
 - BITRISE_BUILD_NUMBER: 0
```

## Testing
- `bitrise run test`
 - Note: This test requires additional configuration to pass:
     1. `AWS_ACCESS_KEY` and `AWS_SECRET_KEY` must be set in `.bitrise.secrets.yml`
     1. An Amazon device farm project must be set up in the target region, and its ARN must be specified in the `device_farm_project` input
     1. If `platform` input is...
       1. ... set to `ios`, then `ios_pool` must be set to the ARN of an iOS device pool and either `ipa_path` or `ipa_package_name` must be set
       1. ... set to `android`, then `android_pool` must be set to the ARN of an Android device pool and either `apk_path` or `apk_package_name` must be set
       1. ... set to `ios+android`, then all of the above inputs must be set
  - see `step.yml` for more info on obtaining ARNs

## How to create your own step

1. Create a new git repository for your step (**don't fork** the *step template*, create a *new* repository)
2. Copy the [step template](https://github.com/bitrise-steplib/step-template) files into your repository
3. Fill the `step.sh` with your functionality
4. Wire out your inputs to `step.yml` (`inputs` section)
5. Fill out the other parts of the `step.yml` too
6. Provide test values for the inputs in the `bitrise.yml`
7. Run your step with `bitrise run test` - if it works, you're ready

__For Step development guidelines & best practices__ check this documentation: [https://github.com/bitrise-io/bitrise/blob/master/_docs/step-development-guideline.md](https://github.com/bitrise-io/bitrise/blob/master/_docs/step-development-guideline.md).

**NOTE:**

If you want to use your step in your project's `bitrise.yml`:

1. git push the step into it's repository
2. reference it in your `bitrise.yml` with the `git::PUBLIC-GIT-CLONE-URL@BRANCH` step reference style:

```
- git::https://github.com/user/my-step.git@branch:
   title: My step
   inputs:
   - my_input_1: "my value 1"
   - my_input_2: "my value 2"
```

You can find more examples of step reference styles
in the [bitrise CLI repository](https://github.com/bitrise-io/bitrise/blob/master/_examples/tutorials/steps-and-workflows/bitrise.yml#L65).

## How to contribute to this Step

1. Fork this repository
2. `git clone` it
3. Create a branch you'll work on
4. To use/test the step just follow the **How to use this Step** section
5. Do the changes you want to
6. Run/test the step before sending your contribution
  * You can also test the step in your `bitrise` project, either on your Mac or on [bitrise.io](https://www.bitrise.io)
  * You just have to replace the step ID in your project's `bitrise.yml` with either a relative path, or with a git URL format
  * (relative) path format: instead of `- original-step-id:` use `- path::./relative/path/of/script/on/your/Mac:`
  * direct git URL format: instead of `- original-step-id:` use `- git::https://github.com/user/step.git@branch:`
  * You can find more example of alternative step referencing at: https://github.com/bitrise-io/bitrise/blob/master/_examples/tutorials/steps-and-workflows/bitrise.yml
7. Once you're done just commit your changes & create a Pull Request


## Share your own Step

You can share your Step or step version with the [bitrise CLI](https://github.com/bitrise-io/bitrise). Just run `bitrise share` and follow the guide it prints.
