# Bazel Web Testing Rules

## Configure your Bazel project

Add the following to your WORKSPACE file:

```bzl
local_repository(
    name = "web_test_rules",
    path = "/usr/local/google/home/fisherii/git/web_test_launcher",
)

load("@web_test_rules//web:repositories.bzl", "web_test_repositories")

web_test_repositories(
    go = True,
    java = True,
)

load("@web_test_rules//web:bindings.bzl", "web_test_bindings")

web_test_bindings()
```

This will configure the following repositories required to get web_test_suite
working:

*   [io_bazel_rules_go](https://github.com/bazelbuild/rules_go)
*   [com_github_gorilla_mux](https://github.com/gorilla/mux)
*   [org_seleniumhq_server](http://www.seleniumhq.org/download/) -- Selenium
    Standalone Server
*   [org_seleniumhq_java](http://www.seleniumhq.org/download/) -- Java Client
    Binding (only if java = True)
*   [org_json_json](https://mvnrepository.com/artifact/org.json/json) (only if
    java = True)
*   [com_google_code_findbugs_jsr305](https://mvnrepository.com/artifact/com.google.code.findbugs/jsr305)
    (only if java = True)
*   [com_google_guava_guava](https://mvnrepository.com/artifact/com.google.guava/guava)
    (only if java = True)
*   [com_github_tebeka_selenium](https://github.com/tebeka/selenium) (only if
    go = True)

## Write your tests

Write your test in the language of your choice, but use our provided Browser API
to get an instance of WebDriver.

Example Test (Java):

```java
import com.google.testing.web.Browser;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.JUnit4;
import org.openqa.selenium.WebDriver;

@RunWith(JUnit4.class)
public class BrowserTest {
  private WebDriver driver;

  @Before public void createDriver() {
    driver = new Browser().newSession();
  }

  @After public void quitDriver() {
    try {
      driver.quit();
     } finally {
      driver = null;
     }
   }

  // your tests here
}
```

Example Test (Go):

```go
import (
    "testing"

    "github.com/tebeka/selenium/selenium"
    "github.com/web_test_launcher/go/browser"
)

func TestWebApp(t *testing.T) {
    wd, err := NewSession(nil)
    if err != nil {
        t.Fatal(err)
    }

    // your test here

    if err := wd.Quit(); err != nil {
        t.Logf("Error quitting webdriver: %v", err)
    }
}
```

In your BUILD files, create your test target as normal, but tag it "manual".
Then create a web_test_suite that depends on your test target:

```bzl
load("@io_bazel_rules_go//go:def.bzl", "go_test")
load("@web_test_rules//web:web.bzl", "web_test_suite")

go_test(
    name = "browser_test_wrapped",
    srcs = ["browser_test.go"],
    tags = ["manual"],
    deps = [
        "@com_github_tebeka_selenium//:selenium",
        "@web_test_rules//go:browser",
    ],
)

web_test_suite(
    name = "browser_test",
    browsers = [
        "@web_test_rules//browsers:chrome-native",
    ],
    local = 1,
    test = ":browser_test_wrapped",
)
```
