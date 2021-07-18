# Cypress Modern Automation


## Install

```
npm i cypress --save-dev
```

## Open Test Runner
```
node_modules/.bin/cypress open
```



## Selectors

be = behavior

comparison = have.

Checkbox
--------

```
cy.get('#checkBoxOption1').check()
      .should('be.checked')
      .and('have.value', 'option1')

cy.get('#checkBoxOption1').uncheck()
      .should('not.be.checked')

// Multiple checkboxes, find common property: type,
// check values
cy.get('input[type="checkbox"]').check(['option2', 'option3'])

```

Dropdowns
---------

Static, tagname == select

Pass option text, or **attribute value**
```
cy.get('select').select('option2')
    .should('have.value', 'option2')
```

Dynamic, loop and check
```
cy.get('#autocomplete').type('ind')

cy.get('.ui-menu-item div').each($el, index, $this) => {
  if($el.text() == "India"){
    $el.click()
  }
})

cy.get('#autocomplete').should('have.value', 'India')
```

Modal
-----

```
cy.get('#displayed-text').should('be.visible')

cy.get('#hide-textbox').click()
cy.get('#displayed-text').should('not.be.visible')
```

Radio
-----
```
cy.get('[value="radio2"]').check().should('be.checked')
```

Popups
------
It auto accepts alerts and popups, ok

**Confirm**: and auto confirms, confirm - cancel

```
cy.get('#alertbtn').click()
cy.get('[value="Confirm"']).click()
```

### Browser Events
You can trigger it from cypress, and get the text
```
// window alert
cy.on('window:alert', (str) => {
  //Mocha
  expect(str).to.equal('The text of the alert')
})


//confirm alert
cy.on('window:confirm', (str) => {
  //Mocha
  expect(str).to.equal('The text of the confirm')
})

```

Child Tabs
----------
Go to another tab and continue execution

> **Does not** have knowledge to child tabs

### Workaround

target="_blank", will open href

make it open in the **same tab**

Verify target attribute, mimic the behavior
> Delete target attribute

Cypress can manipulate the dom, invoke jQuery functions (like .removeAttr())

```
cy.get('#opentab').invoke('removeAttr', 'target').click()
```

Browser Navigation
------------------
Assertions: include, be, have

Validate you are in the right page, retrieve url
```
// current url that is active
cy.url('').should('include', 'website.com')
```
```
cy.go('back')
```

Web Tables
----------
Validate a price is equal to row

**then()** applies to **get** method

* scan
* check the next column
```
cy.get('tr td:nth-child(2)').each($el, index, $list) => {
  const text = $el.text()
  if(text.includes('Python')){
    cy.get('tr td:nth-child(2)')
    .eq(index).next().then(function(price){
      const priceText = price.text()
      expect(priceText).to.equal('25')
    })
  }
})

```

Mouse Hover
----------
No Direct Support

Workaround, trigger jQuery mouse over

```
// Way 1: cy.get('#mousehover').invoke('show')
// Way 2: force option, includes hidden elements
cy.contains('top').click({force: true})
// url now has #top at the end
cy.url().should('include', 'top')
```

Child windows
-------------
2nd way, works on links with same domain ONLY

- Get the url before you navigate, href attribute
- visit url
```
cy.get('#opentab').then(function(el){
  const url = el.prop('href')
  cy.visit(url)
})
```

Limitation of Frames
--------------------
Cypress does not support iframes (or right click)

There is no alternate way.


Best practices in building Cypress Framework
--------------------------------------------
* Setting up test Ηοοks
* Data driven testing with fixtures
* Building Custom Cypress commands
* Parameterize test with Multiple Data sets
* Understand the test Debugging
* Build Page object Design Pattern for objects
* Configuration changes in Cypress.json
* Screenshots and Video recording for Test
* Exploring Cypress Dashboard
* Understand the Environmental variables of Cypress
* Generate Excellent reports for Test execution results
* Integrate Cypress tests with Jenkins CI


Test Hooks
----------
come from Mocha

conditions to run before / after / beforeEach / afterEach test


Property Validation
-------------------
// only validate if attr have a certain value
```
cy.get('input[name="name"]:nth-child(2)')
  .should('have.attr', 'minlength', '2')
```


Cypress Commands
----------------
Write a function that takes product name and click Add Button

Iterate through common selector and find title


Create new command: ```support/commands.js```

```
Cypress.Commands.add('selectProduct', (productName) => {
  cy.get('h4.card-title').each(($el, index, $list) => {
    if($el.text().includes(productName)){
      cy.get('button.btn.btn-info').eq(index).click()
    }
  })
})
```

and use command in integration test:
```
cy.selectProduct('Blackberry')
```

Loop through JSON fixtures
--------------------------
```
{
  "productName": ['Blackberry', 'Nokia Edge']
}
```

```
this.data.productName.forEach(function(el){
  cy.selectProduct(el)
})
```

Test Debugging
--------------
Pause test and inspect at runtime

```
cy.pause()

// or
cy.get(':nth-child(2) > .nav-link').click().debug()
```

Page Object Design Pattern
--------------------------

We collect cases from one specific place and write tests in one class

```cypress/integration/pageObjects/HomePage```

```
class HomePage {

  getEditBox(){
    return cy.get('input[name="name"]:nth-child(2)')
  }

  getTwoWayDataBinding(){
    return cy.get(':nth-child(4) > .ng-untouched')
  }

  getGender(){
    return cy.get('select')
  }

  getEnterpreneaur(){
    return cy.get('#inlineRadio3')
  }

  getShopTab(){
    return cy.get((':nth-child(2) > .nav-link'))
  }
}

export default HomePage
```

in Test:
```
import HomePage from '../pageObjects/HomePage'

...

const homePage = new HomePage()

homePage.getEditBox().type(this.data.name)
```

Global Configs
--------------
```cypress.json``
```
All Cypress Configs can be found in Test Runner > CypressJs > Settings.

Settings can be overwritten from ```cypress.json``` file

{
  "defaultCommandTimeout": 8000,
  "pageLoadTimeout": 10000
}

```

and overwritten per spec file.

Explicit wait mechanism:
```
Cypress.config('defaultCommandTimeout', 8000)
cy.get('elem').click()
// until the end of test
```

**Partial text comparison**
```
cy.get('.alert ').then(function(element){
  const actualText = element.text()
  expect(actualText.includes("Success!")).to.be.true
})
```


Sum of Numbers
--------------
Although Cypress will wait for it's commands to finish before executing the next one, JavaScript is a asynchronous.

That means that we need to promisify Cypress methods before using js
```
let sum = 0

cy.get('tr td:nth-child(4) strong').each(($el, index, $list) => {
  const amount = $el.text()
  let res = amount.split(" ")
  res = res[1].trim()
  sum +=  Number(res)
}).then(function(){
  cy.log(sum)
})

cy.get('h3 strong').then(function(element){
  const amount = element.text()
  let res = amount.split(" ")
  let total = res[1].trim()
  expect(Number(total)).to.equal(sum)
})
```

Environmental Variables
-----------------------
```cypress.json```

```
{
  "defaultCommandTimeout": 8000,
  "pageLoadTimeout": 10000,
  "env": {
    "url": "https://rahulshettyacademy.com/angularpractice/"
  }
}
```


```
cy.visit(Cypress.env('url'))
```

Test Only One File AND add Env
-------------------------------
```
$ node_modules/.bin/cypress run --spec cypress/integration/examples/TestFramework.js --env url=website.com --headed
```

**Env at Runtime**
command line --env takes priority, overwrites cypress.json env variables

On Failure, Cypress automatically takes screenshots and stores them.

Dashboard
---------

Dashboard > Runs > Login with Github

Set up project

1) Insert ProjectID into your cypress.json file

2) Run cypress while passing the record key
node_modules/.bin/cypress run --record --key the-key


Integrate Mochawesome reports
-----------------------------

```
npm install --save-dev mocha mochawesome
```

In ```cypress.json```
```
{
  ...,
 "reporter": "mochaawesome"
}
```

Generate mochawesome report for one spec

```
node_modules/.bin/cypress run --reporter mochawesome --spec cypress/integration/examples/TestFramework.js
```