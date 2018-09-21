<p align="center">
  <a href="https://wasiqb.github.io/coteafs/appium/">
    <img src="assets/coteafs-appium-logo.png" width=300 padding=10 />
  </a>
</p>

<h1 align="center">Wrapper Appium Framework which supports Automation of Mobile and Tablet apps for Android and iOS Real Devices / Emulators.</h1>

<div align="center">

[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)][home]
[![CircleCI](https://circleci.com/gh/WasiqB/coteafs-appium.svg?style=svg)][circleci]
[![Bugs](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=bugs)](https://sonarcloud.io/project/issues?id=com.github.wasiqb.coteafs%3Aappium&resolved=false)
[![Test Coverage](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=coverage)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=Coverage)
[![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=alert_status)](https://sonarcloud.io/dashboard?id=com.github.wasiqb.coteafs%3Aappium)
[![Maintainability](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=sqale_rating)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=Maintainability)
[![Reliability](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=reliability_rating)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=Reliability)
[![Security](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=security_rating)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=Security)
[![Vulnurability](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=vulnerabilities)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=new_vulnerabilities)
[![Duplicate Code](https://sonarcloud.io/api/project_badges/measure?project=com.github.wasiqb.coteafs%3Aappium&metric=duplicated_lines_density)](https://sonarcloud.io/component_measures?id=com.github.wasiqb.coteafs%3Aappium&metric=Duplications)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.wasiqb.coteafs/appium.svg)][maven]
[![Github Releases](https://img.shields.io/github/downloads/WasiqB/coteafs-appium/total.svg)](https://github.com/WasiqB/coteafs-appium/releases)

</div>

## Getting Started
* The [wiki][] of coteafs-appium includes all the information you need to get started including setup, usage, advantages, sample test.
* To know what changes are Released, check out the [change log][] for complete list of changes.
* Want to know when our next feature or fix release is going to happen? Watch out our planned [milestones][].

## [How to configure the tests?][config-doc]

First step in writing tests using coteafs-appium framework is defining a Yaml config file in the `src/test/resources` folder.

**(_For more details, check the link above._)**

Sample file is shown below.

```yaml
servers:
  android:
    ip: 127.0.0.1
    port: 4723
    external: true
    arguments:
      log_level: DEBUG
      log_time_stamp: true
      local_time_zone: true
      session_override: true
      android:
        suppress_adb_kill_server: true
devices:
  test:
    platform_type: ANDROID
    device_name: MI Redmi Note 4
    device_version: 7.0
    app_type: HYBRID
    device_type: REAL
    automation_name: APPIUM
    app_location: apps/android/VodQA.apk
    session_timeout: 120000
    playback:
      delay_before_swipe: 2
      delay_after_swipe: 1
      delay_before_tap: 2
      delay_after_tap: 1
```

## [How simple it is to write the tests?][test-doc]

By using Action classes for each Activity, the flow specific for that activity can be modularized and tests looks much clean and readable.
See the sample test below.

```java
import org.testng.annotations.AfterClass;
import org.testng.annotations.AfterMethod;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.BeforeMethod;

import com.github.wasiqb.coteafs.appium.android.vodqa.actions.LoginActivityAction;
import com.github.wasiqb.coteafs.appium.service.AppiumServer;

public class SampleTest {
  protected AndroidDevice androidDevice;
  private AppiumServer androidServer;

  @BeforeClass
  public void setupTestSuite () {
    this.androidServer = new AppiumServer ("android");  // Here the parameter refers to the key in server block in config file.
    this.androidServer.start ();

    this.androidDevice = new AndroidDevice (this.androidServer, "test");  // Here the param refers to the key in devices block in config file.
    this.androidDevice.start ();
  }

  @AfterClass (alwaysRun = true)
  public void tearDownTestSuite () {
    this.androidDevice.stop ();
    this.androidServer.stop ();
  }

  @Test
  public void login () {
    final LoginActivityAction login = new LoginActivityAction (this.androidDevice);
    login.addInputValue ("UserName", "admin")
      .addInputValue ("Password", "admin")
      .perform ();
  }
}
```

## [How to create Activity class?][activity-doc]

New class needs to be created for each Activity. There's an abstract activity class for each type of device.
Here `prepare` method needs to be implemented with all the elements available on that Activity.

See the below sample for an Android activity with comments for better understanding.

```java
import org.openqa.selenium.By;

import com.github.wasiqb.coteafs.appium.android.AndroidDevice;
import com.github.wasiqb.coteafs.appium.device.DeviceElement;

public class LoginActivity extends AndroidActivity {
  public LoginActivity (final AndroidDevice device) {
    super (device);
  }

  @Override
  protected DeviceElement prepare () {
    final DeviceElement main = DeviceElement.create ("Main")
      .using (By.id ("android:id/content"));
    DeviceElement.create ("UserName")
      .using (By.id ("username"))
      .parent (main);
    DeviceElement.create ("Password")
      .using (By.id ("password"))
      .parent (main);
    DeviceElement.create ("Login")
      .index (1)    // Index of element when multiple elements for same locator exists.
      .waitStrategy (WaitStrategy.VISIBLE)  // Wait strategy to be used while finding the element.
      .using (By.id ("login"))     // Locator used to find the element.
      .parent (main);    // Parent of current element.
    return main;
  }
}
```

## [How to create your Activity action class?][action-doc]

There is abstract action class provided by framework where Activity specific flow is implemented in perform method.
See the sample Activity action class below.

```java
import com.github.wasiqb.coteafs.appium.android.AndroidActivityActions;
import com.github.wasiqb.coteafs.appium.android.AndroidDevice;
import com.github.wasiqb.coteafs.appium.android.vodqa.activities.LoginActivity;

public class LoginActivityAction extends AndroidActivityActions {
  public LoginActivityAction (final AndroidDevice device) {
    super (device);
  }

  @Override
  public void perform () {
    final LoginActivity login = new LoginActivity (getDevice ());
    login.onElement ("UserName")
      .enterText (value ("UserName"));
    login.onElement ("Password")
      .enterText (value ("Password"));
    login.onDevice ()
      .hideKeyboard ();
    login.onElement ("Login")
      .tap ();
  }
}
```

## Liked the framework until now?
### Add dependency to your projects POM.

```xml
<dependency>
    <groupId>com.github.wasiqb.coteafs</groupId>
    <artifactId>appium</artifactId>
    <version>2.2.0</version>
</dependency>
```

## [Want to know Key Features?][intro-doc]
In order to use a framework, it's important to know it's advantages. Let's see what are the key features of this framework:
* Supports Android and iOS Real Devices and Emulators.
* Able to start server on run-time and also can connect to already running server.
* Enforces Page object model style of coding.
* Allows parallel and sequential execution of tests.
* All capabilities, playback and delay settings are configurable through config file.
* Supports logging of all events occurred during test execution.
* Provides pre-defined errors which wraps the Appium exceptions in a meaningful way.
* Provides inbuilt assertions to verify the device elements.
* Supports any Testing frameworks like TestNG, JUnit or Cucumber.

## What to do when you need help?
* Directly chat with me on my [site][] and I'll revert to you as soon as possible.
* Discuss your queries by writing to us on our [mailing list][]
* If you find any issue which is bottleneck for you, [search the issue tracker][] to see if it is already raised.
* If not raised, then you can create a [new issue][] with required details as mentioned in the issue template.

## What you do if you like the project?
* Spread the word with your network.
* **Star** the project to make the project popular.
* Stay updated with the project progress by **Watching** it.
* Contribute to fix open issues, documentations or add new features. To know more, see our [contributing][] page.

## Contributors

<div>
  <ul>
    <li>
      <a href="https://github.com/WasiqB">
        <img alt="Wasiq Bhamla: Framework developer and maintainer." src="https://github.com/WasiqB.png" width=100 height=100 />
      </a>
    </li>
    <li>
      <a href="https://github.com/mfaisalkhatri">
        <img alt="Mohammad Faisal Khatri: Framework Tester." src="https://github.com/mfaisalkhatri.png" width=100 height=100 />
      </a>
    </li>
  </ul>
</div>

## Thanks for the support.

<p align="left">
  <a href="http://browserstack.com">
    <img src="assets/browserstack-logo-600x315.png" width=300 />
  </a>
</p>

For allowing us to run our unit tests on different platforms.

## Versioning ideology

<p align="left">
  <a href="http://semver.org/">
    <img src="assets/semver.png" width=300 />
  </a>
</p>

## License

<p align="left">
  <a href="http://www.apache.org/licenses/LICENSE-2.0">
    <img src="http://www.apache.org/img/asf_logo.png" width=300 />
  </a>
</p>

[home]: https://github.com/wasiqb/coteafs-appium
[circleci]: https://circleci.com/gh/WasiqB/coteafs-appium
[wiki]: https://wasiqb.github.io/coteafs/appium/intro/
[site]: https://wasiqb.github.io
[mailing list]: https://groups.google.com/forum/#!forum/coteafs-appium-users
[search the issue tracker]: https://github.com/WasiqB/coteafs-appium/issues?q=something
[new issue]: https://github.com/WasiqB/coteafs-appium/issues/new
[coteafs-logger]: https://github.com/WasiqB/coteafs-logger
[coteafs-config]: https://github.com/WasiqB/coteafs-config
[coteafs-error]: https://github.com/WasiqB/coteafs-error
[coteafs-parent]: https://github.com/WasiqB/coteafs-parent
[Appium]: https://github.com/appium/java-client/releases
[change log]: CHANGELOG.md
[milestones]: https://github.com/WasiqB/coteafs-appium/milestones
[maven]: https://maven-badges.herokuapp.com/maven-central/com.github.wasiqb.coteafs/appium
[contributing]: .github/CONTRIBUTING.md
[config-doc]: https://wasiqb.github.io/coteafs/appium/config-basics/
[test-doc]: https://wasiqb.github.io/coteafs/appium/sample-test-class/
[action-doc]: https://wasiqb.github.io/coteafs/appium/action-class/
[activity-doc]: https://wasiqb.github.io/coteafs/appium/activity-class/
[intro-doc]: https://wasiqb.github.io/coteafs/appium/intro/
