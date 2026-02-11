# Selenium WebDriver Automation Guide
*By Sadath EM - QA Test Lead*

## Introduction

Test automation has been a game-changer in my QA career. At Tieto EVRY, Deutsche Bank, and British Telecom, I've built automation frameworks that improved test coverage and reduced regression testing time significantly. This guide shares practical Selenium WebDriver knowledge I've gained automating complex enterprise applications.

## 1. Why Selenium?

### Advantages:
- ✅ **Open Source**: Free to use
- ✅ **Multi-Browser Support**: Chrome, Firefox, Safari, Edge
- ✅ **Multi-Language Support**: Java, Python, C#, JavaScript
- ✅ **Large Community**: Extensive support and resources
- ✅ **CI/CD Integration**: Works with Jenkins, Azure DevOps, GitHub Actions
- ✅ **Cross-Platform**: Windows, Mac, Linux

### When to Use Selenium:
- Web application testing
- Regression test automation
- Cross-browser testing
- Data-driven testing
- Integration with CI/CD pipelines

### When NOT to Use Selenium:
- Desktop applications (use WinAppDriver)
- Mobile apps (use Appium)
- API testing (use REST Assured, Postman)
- Performance testing (use JMeter)

## 2. Getting Started with Selenium

### Installation - Java

```bash
# 1. Install Java JDK
# Download from: https://www.oracle.com/java/technologies/downloads/

# 2. Set JAVA_HOME environment variable
# Windows: Set in System Environment Variables
# Mac/Linux: Add to .bash_profile or .zshrc
export JAVA_HOME=/path/to/jdk
export PATH=$JAVA_HOME/bin:$PATH

# 3. Create Maven project
mvn archetype:generate -DgroupId=com.qa.automation -DartifactId=selenium-tests

# 4. Add Selenium dependency to pom.xml
```

**pom.xml**
```xml
<dependencies>
    <!-- Selenium Java -->
    <dependency>
        <groupId>org.seleniumhq.selenium</groupId>
        <artifactId>selenium-java</artifactId>
        <version>4.15.0</version>
    </dependency>
    
    <!-- TestNG -->
    <dependency>
        <groupId>org.testng</groupId>
        <artifactId>testng</artifactId>
        <version>7.8.0</version>
    </dependency>
    
    <!-- WebDriverManager -->
    <dependency>
        <groupId>io.github.bonigarcia</groupId>
        <artifactId>webdrivermanager</artifactId>
        <version>5.6.2</version>
    </dependency>
</dependencies>
```

### Installation - Python

```bash
# 1. Install Python
# Download from: https://www.python.org/downloads/

# 2. Install Selenium
pip install selenium

# 3. Install pytest (test framework)
pip install pytest

# 4. Install WebDriver Manager
pip install webdriver-manager
```

## 3. First Selenium Script

### Java Example

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

public class FirstSeleniumTest {
    
    public static void main(String[] args) {
        // Setup ChromeDriver
        WebDriverManager.chromedriver().setup();
        
        // Initialize WebDriver
        WebDriver driver = new ChromeDriver();
        
        try {
            // Navigate to URL
            driver.get("https://www.google.com");
            
            // Get page title
            String title = driver.getTitle();
            System.out.println("Page Title: " + title);
            
            // Verify title
            if (title.equals("Google")) {
                System.out.println("Test Passed!");
            } else {
                System.out.println("Test Failed!");
            }
            
        } finally {
            // Close browser
            driver.quit();
        }
    }
}
```

### Python Example

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

# Setup ChromeDriver
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))

try:
    # Navigate to URL
    driver.get("https://www.google.com")
    
    # Get page title
    title = driver.title
    print(f"Page Title: {title}")
    
    # Verify title
    assert title == "Google", "Title mismatch!"
    print("Test Passed!")
    
finally:
    # Close browser
    driver.quit()
```

## 4. Locating Elements

### Locator Strategies

```java
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;

// 1. ID - Most reliable
WebElement element = driver.findElement(By.id("username"));

// 2. Name
WebElement element = driver.findElement(By.name("email"));

// 3. Class Name
WebElement element = driver.findElement(By.className("login-button"));

// 4. Tag Name
WebElement element = driver.findElement(By.tagName("input"));

// 5. Link Text
WebElement element = driver.findElement(By.linkText("Click Here"));

// 6. Partial Link Text
WebElement element = driver.findElement(By.partialLinkText("Click"));

// 7. CSS Selector - Powerful and flexible
WebElement element = driver.findElement(By.cssSelector("#username"));
WebElement element = driver.findElement(By.cssSelector("input[name='email']"));
WebElement element = driver.findElement(By.cssSelector(".login-button"));

// 8. XPath - Most flexible but slower
WebElement element = driver.findElement(By.xpath("//input[@id='username']"));
WebElement element = driver.findElement(By.xpath("//button[text()='Login']"));
```

### Best Practices for Locators

**Priority Order:**
1. **ID** - Fastest and most reliable (if available)
2. **Name** - Good alternative to ID
3. **CSS Selector** - Powerful and faster than XPath
4. **XPath** - Use when nothing else works

**Avoid:**
- Absolute XPath (fragile)
- Complex locators (hard to maintain)
- Position-based locators (brittle)

### Advanced XPath

```java
// Basic XPath
//tagname[@attribute='value']

// Text-based
//button[text()='Login']
//button[contains(text(),'Log')]

// Axes
//input[@id='username']/following-sibling::input
//div[@class='form']/descendant::input

// Multiple attributes
//input[@type='text' and @name='username']

// Parent navigation
//input[@id='username']/parent::div

// Index-based (avoid if possible)
(//input[@type='text'])[1]
```

### CSS Selector Guide

```java
// ID
#username

// Class
.login-button

// Attribute
input[name='email']
input[type='text']

// Multiple attributes
input[type='text'][name='username']

// Contains
input[name*='user']

// Starts with
input[name^='user']

// Ends with
input[name$='name']

// Child
div.form > input

// Descendant
div.form input

// Adjacent sibling
label + input
```

## 5. Common Selenium Operations

### Interacting with Elements

```java
import org.openqa.selenium.*;
import org.openqa.selenium.support.ui.Select;

// Click button
driver.findElement(By.id("submit-button")).click();

// Enter text
driver.findElement(By.id("username")).sendKeys("test_user");

// Clear text
driver.findElement(By.id("username")).clear();

// Get text
String text = driver.findElement(By.id("message")).getText();

// Get attribute
String value = driver.findElement(By.id("username")).getAttribute("value");

// Submit form
driver.findElement(By.id("login-form")).submit();

// Check if element is displayed
boolean isDisplayed = driver.findElement(By.id("error-message")).isDisplayed();

// Check if element is enabled
boolean isEnabled = driver.findElement(By.id("submit-button")).isEnabled();

// Check if checkbox/radio is selected
boolean isSelected = driver.findElement(By.id("remember-me")).isSelected();

// Handle dropdowns
Select dropdown = new Select(driver.findElement(By.id("country")));
dropdown.selectByVisibleText("India");
dropdown.selectByValue("IN");
dropdown.selectByIndex(2);

// Handle checkboxes
WebElement checkbox = driver.findElement(By.id("terms"));
if (!checkbox.isSelected()) {
    checkbox.click();
}

// Handle radio buttons
driver.findElement(By.xpath("//input[@value='male']")).click();
```

### Browser Operations

```java
// Navigate
driver.get("https://example.com");
driver.navigate().to("https://example.com");

// Back, Forward, Refresh
driver.navigate().back();
driver.navigate().forward();
driver.navigate().refresh();

// Get current URL
String currentUrl = driver.getCurrentUrl();

// Get page title
String title = driver.getTitle();

// Get page source
String pageSource = driver.getPageSource();

// Maximize window
driver.manage().window().maximize();

// Set window size
driver.manage().window().setSize(new Dimension(1920, 1080));

// Full screen
driver.manage().window().fullscreen();

// Close current window
driver.close();

// Close all windows and quit
driver.quit();
```

## 6. Waits - Critical for Stable Tests

### Implicit Wait (Not Recommended)

```java
// Sets default wait time for all element searches
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
```

### Explicit Wait (Recommended)

```java
import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;

WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));

// Wait for element to be visible
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("submit-button"))
);

// Wait for element to be clickable
WebElement element = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("submit-button"))
);

// Wait for element to be present
WebElement element = wait.until(
    ExpectedConditions.presenceOfElementLocated(By.id("message"))
);

// Wait for text to be present
wait.until(
    ExpectedConditions.textToBePresentInElement(
        driver.findElement(By.id("message")), 
        "Success"
    )
);

// Wait for alert
wait.until(ExpectedConditions.alertIsPresent());

// Wait for title
wait.until(ExpectedConditions.titleContains("Dashboard"));

// Wait for URL
wait.until(ExpectedConditions.urlContains("dashboard"));
```

### Fluent Wait (Most Flexible)

```java
import org.openqa.selenium.support.ui.FluentWait;
import org.openqa.selenium.NoSuchElementException;

Wait<WebDriver> wait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofSeconds(2))
    .ignoring(NoSuchElementException.class);

WebElement element = wait.until(driver -> 
    driver.findElement(By.id("dynamic-element"))
);
```

## 7. Handling Complex Scenarios

### Alerts

```java
// Accept alert
driver.switchTo().alert().accept();

// Dismiss alert
driver.switchTo().alert().dismiss();

// Get alert text
String alertText = driver.switchTo().alert().getText();

// Enter text in prompt
driver.switchTo().alert().sendKeys("test input");
driver.switchTo().alert().accept();
```

### Frames

```java
// Switch to frame by index
driver.switchTo().frame(0);

// Switch to frame by name or ID
driver.switchTo().frame("frame-name");

// Switch to frame by WebElement
WebElement frameElement = driver.findElement(By.id("frame1"));
driver.switchTo().frame(frameElement);

// Switch back to main content
driver.switchTo().defaultContent();

// Switch to parent frame
driver.switchTo().parentFrame();
```

### Windows/Tabs

```java
// Get current window handle
String mainWindow = driver.getWindowHandle();

// Click link that opens new window
driver.findElement(By.linkText("Open New Window")).click();

// Get all window handles
Set<String> allWindows = driver.getWindowHandles();

// Switch to new window
for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
        break;
    }
}

// Perform actions in new window
// ...

// Close new window
driver.close();

// Switch back to main window
driver.switchTo().window(mainWindow);
```

### JavaScript Executor

```java
import org.openqa.selenium.JavascriptExecutor;

JavascriptExecutor js = (JavascriptExecutor) driver;

// Scroll to element
WebElement element = driver.findElement(By.id("footer"));
js.executeScript("arguments[0].scrollIntoView(true);", element);

// Scroll to bottom
js.executeScript("window.scrollTo(0, document.body.scrollHeight)");

// Click using JavaScript (when regular click fails)
js.executeScript("arguments[0].click();", element);

// Enter text using JavaScript
js.executeScript("arguments[0].value='test text';", element);

// Highlight element (for debugging)
js.executeScript("arguments[0].style.border='3px solid red'", element);

// Get page title
String title = (String) js.executeScript("return document.title;");

// Refresh page
js.executeScript("location.reload()");
```

### File Upload

```java
// Simple file upload
WebElement uploadElement = driver.findElement(By.id("file-upload"));
uploadElement.sendKeys("/path/to/file.pdf");

// Click upload button
driver.findElement(By.id("upload-button")).click();
```

### Screenshots

```java
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.OutputType;
import org.apache.commons.io.FileUtils;
import java.io.File;

// Take screenshot
File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(screenshot, new File("screenshot.png"));

// Screenshot of specific element
WebElement element = driver.findElement(By.id("login-form"));
File elementScreenshot = element.getScreenshotAs(OutputType.FILE);
FileUtils.copyFile(elementScreenshot, new File("element.png"));
```

## 8. Page Object Model (POM)

### Why POM?
- Better code organization
- Improved maintainability
- Reduces code duplication
- Easier to update when UI changes

### Example Implementation

**LoginPage.java**
```java
package pages;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
import org.openqa.selenium.support.PageFactory;

public class LoginPage {
    
    private WebDriver driver;
    
    // Page elements using @FindBy
    @FindBy(id = "username")
    private WebElement usernameField;
    
    @FindBy(id = "password")
    private WebElement passwordField;
    
    @FindBy(id = "login-button")
    private WebElement loginButton;
    
    @FindBy(id = "error-message")
    private WebElement errorMessage;
    
    // Constructor
    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }
    
    // Page actions/methods
    public void enterUsername(String username) {
        usernameField.clear();
        usernameField.sendKeys(username);
    }
    
    public void enterPassword(String password) {
        passwordField.clear();
        passwordField.sendKeys(password);
    }
    
    public void clickLoginButton() {
        loginButton.click();
    }
    
    public DashboardPage login(String username, String password) {
        enterUsername(username);
        enterPassword(password);
        clickLoginButton();
        return new DashboardPage(driver);
    }
    
    public String getErrorMessage() {
        return errorMessage.getText();
    }
    
    public boolean isLoginButtonDisplayed() {
        return loginButton.isDisplayed();
    }
}
```

**LoginTest.java**
```java
package tests;

import org.testng.annotations.*;
import pages.LoginPage;
import pages.DashboardPage;

public class LoginTest extends BaseTest {
    
    private LoginPage loginPage;
    
    @BeforeMethod
    public void setup() {
        driver.get("https://example.com/login");
        loginPage = new LoginPage(driver);
    }
    
    @Test
    public void testValidLogin() {
        DashboardPage dashboard = loginPage.login("valid_user", "valid_pass");
        Assert.assertTrue(dashboard.isWelcomeMessageDisplayed());
    }
    
    @Test
    public void testInvalidLogin() {
        loginPage.login("invalid_user", "invalid_pass");
        Assert.assertEquals(loginPage.getErrorMessage(), "Invalid credentials");
    }
}
```

## 9. TestNG Framework Integration

**testng.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Test Suite">
    <test name="Regression Tests">
        <classes>
            <class name="tests.LoginTest"/>
            <class name="tests.DashboardTest"/>
            <class name="tests.OrderTest"/>
        </classes>
    </test>
</suite>
```

**BaseTest.java**
```java
package tests;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.testng.annotations.*;
import io.github.bonigarcia.wdm.WebDriverManager;

public class BaseTest {
    
    protected WebDriver driver;
    
    @BeforeClass
    public void setupClass() {
        WebDriverManager.chromedriver().setup();
    }
    
    @BeforeMethod
    public void setupTest() {
        driver = new ChromeDriver();
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
    }
    
    @AfterMethod
    public void teardownTest() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

## 10. Data-Driven Testing

**Using TestNG DataProvider**
```java
@DataProvider(name = "loginData")
public Object[][] getLoginData() {
    return new Object[][] {
        {"user1", "pass1", "Invalid credentials"},
        {"user2", "pass2", "Invalid credentials"},
        {"valid_user", "valid_pass", null}
    };
}

@Test(dataProvider = "loginData")
public void testLogin(String username, String password, String expectedError) {
    loginPage.login(username, password);
    if (expectedError != null) {
        Assert.assertEquals(loginPage.getErrorMessage(), expectedError);
    } else {
        Assert.assertTrue(dashboardPage.isWelcomeMessageDisplayed());
    }
}
```

**Using Excel/CSV**
```java
import org.apache.poi.ss.usermodel.*;

public class ExcelReader {
    
    public static Object[][] getExcelData(String filePath, String sheetName) {
        // Read Excel file
        // Return data as Object[][]
    }
}
```

## 11. Best Practices from My Experience

### Code Organization
1. Use Page Object Model
2. Separate test data from test logic
3. Create reusable utility methods
4. Maintain proper package structure

### Element Interaction
1. Always use explicit waits
2. Verify element state before interaction
3. Handle stale element exceptions
4. Use try-catch for better error handling

### Test Design
1. Keep tests independent
2. One assertion per test (when possible)
3. Use meaningful test names
4. Add proper logging
5. Take screenshots on failure

### Maintenance
1. Use constants for URLs, timeouts
2. Externalize test data
3. Regular code reviews
4. Update dependencies regularly

## 12. Common Issues and Solutions

### StaleElementReferenceException
```java
// Problem: Element is no longer attached to DOM

// Solution 1: Re-locate element
WebElement element = driver.findElement(By.id("dynamic-element"));
element.click(); // May fail

// Better approach
driver.findElement(By.id("dynamic-element")).click();

// Solution 2: Use explicit wait
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
wait.until(ExpectedConditions.refreshed(
    ExpectedConditions.elementToBeClickable(By.id("dynamic-element"))
));
```

### ElementNotInteractableException
```java
// Solution: Wait for element to be clickable
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(
    ExpectedConditions.elementToBeClickable(By.id("button"))
);
element.click();
```

### NoSuchElementException
```java
// Solution: Use proper waits and verify locator
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(
    ExpectedConditions.presenceOfElementLocated(By.id("element"))
);
```

## 13. CI/CD Integration

**Jenkins Integration**
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/selenium-tests.git'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'mvn clean test'
            }
        }
        
        stage('Generate Report') {
            steps {
                publishHTML([
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'Test Report'
                ])
            }
        }
    }
}
```

## Conclusion

Selenium WebDriver is a powerful tool for test automation. The key to success is:
- Writing maintainable code with POM
- Using proper waits
- Handling exceptions gracefully
- Integrating with CI/CD
- Continuous learning and improvement

Start small, automate critical user journeys first, and gradually expand your automation coverage.

---

## Sample Project

I've created a complete Selenium automation framework with all best practices implemented. Download it from the Resources section.

---

*Questions about Selenium automation? [Let's connect](../contact.html).*

**Tags**: #Selenium #TestAutomation #WebDriver #QA #Java #Python
