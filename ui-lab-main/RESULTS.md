# UI lab results

Tool parts that I chose: Part 2, Selenium WebDriver in Java, and Part 3, Robot Framework

All runs below were made on 23 November 2026 on Linux x86_64. Runs with a visible window used a virtual display (`xvfb-run`).

## Part 0
- Step 0.1, Chrome version: Google Chrome for Testing 153.0.8010.12 (linux64)
- Step 0.3, ChromeDriver version: ChromeDriver 153.0.8010.12 (downloaded from Chrome for Testing into `~/drivers/chromedriver-linux64`, outside the repository). The ChromeDriver that came with the machine was 147.0.7727.24, which did not match Chrome 153, so I downloaded the matching one as Step 0.2 says.
- Step 0.4, `#floating_filled` finds 3 elements; `//label[normalize-space()='Max Calories per Slice']/preceding-sibling::input` finds 1.
- Step 0.4, locator of Pizza, Please!: `[name="pizza-please"]` (1 element)
- Step 0.4, locator of Excluded tools: `select` (1 element, the only list on the page)
- Step 0.4, locator of Must be vegetarian: `#default-checkbox` (1 element)
- Step 0.4, locator of Custom Pizza Name: `input#pizza-name` (1 element; after the answer appears `#pizza-name` alone finds 2, because the answer heading has the same id)
- Step 0.5, pizza name, tool and calories of the run by hand: Lab Pizza 20261123-055222, tool Scissors, 475 calories per slice

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1    | Heading reads "Looking to break out of your pizza routine?" | Pass |
| 2    | Max Calories per Slice (shows 1000), Excluded tools, Must be vegetarian and Custom Pizza Name appear | Pass |
| 3    | Fields show 600, Knife, vegetarian ticked, Lab Pizza 20261123-055222 | Pass |
| 4    | Name: Lab Pizza 20261123-055222, Tool: Scissors, Calories per slice: 475 | Pass |
| 5    | "Please log in first." | Pass |
| 6    | Both boxes were pre-filled with admin; after Sign in the page shows "Latest pizza recommendations" | Pass |
| 7    | "Lab Pizza 20261123-055222 (tool=Scissors, ingredients_number=8) (newest)" | Pass |

## Part 2: Selenium WebDriver in Java
- Step 2.4, the failure line for the wrong title: `[ERROR]   FirstTest.webFormOpens:20 expected: <Web Form> but was: <Web form>`
- Step 2.9, the failure for "Received": `expected: <Received> but was: <Received!>`
- Step 2.10, WaitTest without a wait: `no such element: Unable to locate element: {"method":"css selector","selector":"#box0"}`
- Step 2.15, QP01Test run 1, result and time: Tests run: 1, Failures: 0, BUILD SUCCESS, 5.109 s
- Step 2.15, QP01Test run 2, result and time: Tests run: 1, Failures: 0, BUILD SUCCESS, 5.069 s
- Step 2.20, full run 1, time and number of tests: Tests run: 5, Failures: 0, Errors: 0, BUILD SUCCESS, total time 16.031 s (screenshots in `selenium-java/screenshots/20261123-055515`)
- Step 2.20, full run 2, time and number of tests: Tests run: 5, Failures: 0, Errors: 0, BUILD SUCCESS, total time 15.562 s (screenshots in `selenium-java/screenshots/20261123-055532`, report in `selenium-java/reports`)

## Part 3: Robot Framework
- robot --version: Robot Framework 7.5.1 (Python 3.13.18 on linux)
- Step 3.2, failure message for the wrong title: `Title should have been 'Web Form' but was 'Web form'.`
- Step 3.8, run 1, result, elapsed time and pizza name: 1 test, 1 passed, 0 failed, elapsed 00:00:03.516, Lab Pizza 20261123-055638 (tool Scissors, 450 calories)
- Step 3.8, run 2, result, elapsed time and pizza name: 1 test, 1 passed, 0 failed, elapsed 00:00:03.596, Lab Pizza 20261123-055642 (tool Scissors, 550 calories)

## Part B
1. Max Calories per Slice, Min Number of Toppings and Max Number of Toppings all have `id="floating_filled"`. In the Console `$$("#floating_filled").length` returned 3, so `By.id("floating_filled")` would always pick the first box, which only happens to be the calories box. The tests go through the label text instead, which finds exactly 1 element. I propose giving each box its own id, such as `id="max-calories"`, and pointing its label to it with `for="max-calories"`. Then a tester can use the shortest and most stable locator, and the test no longer breaks when the label wording or the order of the boxes changes.

2. It is a consistency oracle: it does not know the right tool in advance, but it checks that two parts of the system agree with each other (the recommendation on the home page and the line in the admin list). In my runs both sides said Scissors or Pizza cutter, e.g. "Lab Pizza 20261123-055642 (tool=Scissors, ingredients_number=5)". It would catch a fault where the server stores a different pizza or tool from the one it sent to the customer, for example if the admin list read the tool from a stale record or overwrote it with a default value.

3. To check vegetarian, the test needs the answer to say whether the pizza is vegetarian. The page does not show it, but the JSON answer of `POST /api/pizza` does have a field `"vegetarian": true`. A check could read that answer (with a request interceptor or from the API), assert `vegetarian == true`, and also assert that no ingredient in the list is meat, for example none of Pepperoni, Prosciutto or Salami. For the second check the test needs a list of the meat ingredients from QuickPizza's data or from the developers.

4. Both tools waited on elements, never with a fixed pause. In Java I wrote each wait explicitly with `wait.until(ExpectedConditions...)`, and when I left the wait out, WaitTest failed with `NoSuchElementException` for `#box0`. In Robot Framework, keywords like `Wait Until Element Contains` and `Wait Until Page Contains Element` do the same with less code, and `timeout=10s` on the library import set the limit once. QP-01 was quicker to write in Robot: about 25 short keyword lines against a Java test plus two page object classes. Robot's `log.html` was the most readable evidence, with every keyword, its arguments, the values like `${tool} = Scissors`, and the screenshots inline. Maven only gave a pass/fail line and a time (about 5 s for QP-01).

5. I would choose Robot Framework for Project 2. First, the resource file gave page-object-style reuse almost for free, and the test file reads like the written test case (Open QuickPizza, Turn On Advanced, ...), which makes it easy to show that each step and expected result is covered. Second, every run writes its own log, report and screenshots into a folder (`results/run1`, `results/run2`), which is exactly the evidence Project 2 asks for, and a full QP-01 run took about 3.5 s headless.

6. (Part 2) In `QP01Test` the XPath of the calories box is written inline in step 2 of the test, so I would have to edit the test method itself, and every other test that copied the same XPath. In `QP01PageObjectTest` nothing changes: only the constant `MAX_CALORIES` in `HomePage.java` changes, and `showAdvanced`, `setMaxCalories` and `maxCalories` all use it. Because the locator lives in one place, the repair is one line, and the test code that mirrors QP-01 stays the same.

7. (Part 3) I would change only the variable `${MAX CALORIES}` in the Variables section of `resources/quickpizza.resource`. `Turn On Advanced` and `Fill In The Pizza Request` use that variable, and `pizza_scenario.robot` only calls keywords like `Fill In The Pizza Request    600    Knife    ${name}`, so the test file never sees the locator and needs no change.
