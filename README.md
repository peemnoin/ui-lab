# ui-lab

ITCS386 lab: automated UI testing. The tool parts that I chose are **Part 2, Selenium WebDriver in Java**, and **Part 3, Robot Framework**.
Both parts automate the written test case QP-01 ("A custom pizza reaches the admin list") on https://quickpizza.grafana.com, and also test Selenium's web form.
My results and reflection are in [RESULTS.md](RESULTS.md).

Both parts read the path of ChromeDriver from the environment variable `CHROMEDRIVER` (Step 0.3), so no test holds a path from one computer.

## selenium-java/ (Part 2)

A Maven project with JUnit 6 and Selenium 4.52. `src/test/java/uilab` holds `BaseTest` (opens and closes Chrome, takes screenshots), the web form, wait and QP-01 tests, and the page objects `pages/HomePage` and `pages/AdminPage`. `screenshots/` holds the pictures of steps 3, 4, 5 and 7 from two full runs, and `reports/` holds the Surefire report of `QP01PageObjectTest`.

```bash
cd selenium-java
mvn test -Dheadless=true
```

## robot/ (Part 3)

Robot Framework 7.5 with SeleniumLibrary 6.9. `first.robot` and `web_form.robot` test the web form. `pizza_scenario.robot` is QP-01, and its keywords and locators are in `resources/quickpizza.resource`. `results/run1` and `results/run2` hold the log, report and four screenshots of two full runs.

```bash
cd robot
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
robot -d results/run1 -v BROWSER:headlesschrome pizza_scenario.robot
```
