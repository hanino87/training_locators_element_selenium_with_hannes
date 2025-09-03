# Selenium: Locating Elements & DOM Best Practices

This guide explains how Selenium locates elements on a webpage and best practices to create **stable and maintainable locators**.

---

## 📌 What is Locators?

In the context of web automation and testing (like using Selenium, Cypress, or Playwright), a locator is a way to identify and interact with elements on a web page. Web pages are made up of elements like buttons, text fields, links, etc., and to automate actions (like clicking, typing, or verifying text), we need a precise method to "locate" them.

## 📌 What is DOM?

**DOM** stands for **Document Object Model**.  

- It’s a **tree-like structure** that the browser builds when it loads an HTML page.  
- Every HTML element becomes a **node** in this tree.  
- The DOM allows **JavaScript, Selenium, and other automation tools** to access, read, and manipulate page elements.  

### Example HTML

```html
<html>
  <body>
    <div id="root">
      <form id="login-form">
        <label>Username</label>
        <input placeholder="Username" type="text">
        <label>Password</label>
        <input placeholder="Password" type="password">
        <button id="signup">Sign Up</button>
      </form>
    </div>
  </body>
</html>
```

### Corresponding DOM Tree

```
html
└── body
    └── div#root
        └── form#login-form
            ├── label (Username)
            ├── input (placeholder="Username")
            ├── label (Password)
            ├── input (placeholder="Password")
            └── button#signup
```

---

## 🛠️ How to Inspect the DOM on a Webpage

1. Right-click on the page.  
2. Select **Inspect** (or press `F12`).  
3. Explore the **Elements** panel to see the DOM tree.

---

## 🛠️ Selenium Element Locators

Selenium needs to **locate elements in the DOM** to perform actions like typing, clicking, etc.

### List of availabe Locators for Selenium to identify 

ID – usually unique, e.g., <input id="username"> → #username
Name – e.g., <input name="password"> → [name="password"]
Class – e.g., <button class="btn-primary">
CSS selectors – flexible, e.g., div > input[type='text']
XPath – more powerful, e.g., //button[text()='Log in']
Testautomation Tools own built in Locators 


### 1. Prefer IDs (if available)

```python
driver.find_element(By.ID, "signup")
```

- IDs are **unique and stable**, so these locators rarely break.

---

### 2. Use `name` or `data-*` attributes

If no ID exists:

```html
<input name="username">
<input data-test="username-field">
```

Selenium:

```python
driver.find_element(By.NAME, "username")
driver.find_element(By.CSS_SELECTOR, '[data-test="username-field"]')
```

---

### 3. Avoid Fragile Absolute XPath

❌ Example (fragile):

```xpath
/html/body/div/div/div/input[1]
```

- If developers add/remove `<div>` elements, this path breaks.  
- Depends too much on the **layout structure**, not the element itself.  
- GUI changes make it fragile in test automation.

---

### 🔑 Golden Rule

> Don’t let locators depend on *where* the element sits in the DOM — make them depend on *what* the element is.

---

## 💡 Solutions for Robust Locators

### 1️⃣ Attribute-based Locators

- Use unique attributes to locate elements.  
- Example:

```python
driver.find_element(By.XPATH, '//input[@placeholder="Username"]')
```
using this generic you have sometims to switch tag and attribute if its label and //tag[@attribute="value"] 

```python

//label[@for="username"]

```




- Benefits:
  - Ignores the depth in the DOM.
  - Works even if the element is moved inside other containers.
  - Stable and maintainable.

---

### 2️⃣ Handle Multiple Matching Elements

If multiple elements have the same attribute:

#### a) Using Index (less preferred):

```python
(//input[@placeholder="Username"])[2]  # selects the second match
```

⚠️ Fragile if the DOM changes.

#### b) Anchor to a Unique Parent/Container (preferred):

```html
<form id="login-form">
    <input placeholder="Username">
    <input placeholder="Password">
</form>
```

XPath:

```python
driver.find_element(By.XPATH, '//form[@id="login-form"]//input[@placeholder="Username"]')
```

- Targets the Username input **inside a specific form**.  
- More robust than using an index.

---

### 4 Class as an option for selectors

<input class="username">

```python
username_input = driver.find_element(By.CLASS_NAME, "username")
```

### 5. Testautomation Tools own built in locators 

Some Testautomation tool have their own built-in locators to use. This is the best practice to use those selectors. 

Selenium has its own built-in locators from the module BY that you import from Selenium

Built-in locators:

By.id()

By.name()

By.className()

By.tagName()

By.linkText()

By.partialLinkText()

By.cssSelector()

By.xpath()

## 📌 Locator Priority & Best Practices

1. Testautomation Tools own built-in locators By from Selenium 

2.ID → Most stable

3.data-test / data-testid → For automation-friendly attributes

4.name

5.placeholder / type / class (only if stable & unique) / other attributes

6.Anchored XPath (parent/container + attributes)

7nth-of-type or index → Last resort

**Tips:**

- Keep selectors **short and readable**.  
- Avoid relying on absolute paths like `div[3]` or `input[1]`.  
- Request frontend developers to add **stable test attributes** like `data-test="username"`.

---

This setup ensures your Selenium tests are **stable, readable, and maintainable**, even if the webpage layout changes.

## Code Example Below of some locators with element for a login test with module By from seleniums. It´s Seleniums own built-in locators. 

```py

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
from selenium.webdriver.common.by import By

APP_URL='http://localhost:5173'

options = Options()
options.add_experimental_option("excludeSwitches", ["enable-logging"])

def test_register_a_user():
    
    #Arrange testsetup 
    driver = webdriver.Chrome(options=options)
    driver.get(APP_URL)

    password="123456"
    
    username="Hannes"

    #Assign a variable to signup button 
    signup_btn=driver.find_element(By.ID, "signup")

    # Perform click on signup button 
    signup_btn.click()

    # Assign usernname and password fields to variables 
    username_field=driver.find_element(By.XPATH,'//input[@placeholder="Username"]')
    password_field=driver.find_element(By.XPATH,'//input[@placeholder="Password"]')
    
     # Fill in texts in password and username field
    password_field.send_keys(password)
    username_field.send_keys(username)
  
    # Teardown - end of test 
    driver.quit()



```

## Code Example Below of some locators with element for a login test without module BY from Selenium use for older Selenium version and it`s not a built-in locators you can use in Selenium. 

```py

from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time

APP_URL='http://localhost:5173'

options = Options()
options.add_experimental_option("excludeSwitches", ["enable-logging"])

def test_register_a_user():
    
    #Arrange testsetup

    password="123456"
    
    username="Hannes"
    
    driver = webdriver.Chrome(options=options)

    # Use driver to go to webpage 
    driver.get(APP_URL)

    # Assign signup button to a variable 
    signup_btn=driver.find_element("id", "signup")

    # Perform click on signup button 
    signup_btn.click()

    # Assign usernname and password fields to variables 
    username_field=driver.find_element("xpath",'//input[@placeholder="Username"]')
    password_field=driver.find_element("xpath",'//input[@placeholder="Password"]')

    # Fill in texts in password and username field
    password_field.send_keys(password)
    username_field.send_keys(username)
  
    # Teardown - end of test 
    driver.quit()


```


