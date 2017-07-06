# utilities
```js
require('chromedriver');
const selenium = require('selenium-webdriver');
const chrome = require('selenium-webdriver/chrome');
const fs = require('fs');
const sData = require('./sData');
const data = require('./data');
const pause = 3000;

var opt = new chrome.Options();
opt.options_.args = [ '--start-maximized' ];

const By = selenium.By;
const until = selenium.until;

const driver = new selenium.Builder()
  .forBrowser('chrome')
  .setChromeOptions(opt)
  .build();

main().then(() => console.log('done'));

async function main() {
  
  try {

    // Loading the page and clicking on i want to enroll link
    await driver.get(sData.url);
    (await defSel('a[ng-bind-html="enrollToBank"]')).click();

    //Now on Email and phone number screen
    (await defSel('[name="emailField"]')).sendKeys(getData('email'));
    (await defSel('[name="phoneField"]')).sendKeys(getData('phone'));
    (await defSel('[type="submit"]')).click();

    // Now on OTP screen
    var otp = await (await defSel(
      `[ng-class="{'pg-message-row-first': $first, 'pg-message-row-last': $last}"]`, false
    )).getText();
    otp = otp.substr(otp.length - 7, 6);
    (await defSel('[name="validationCodeField"]', false)).sendKeys(otp);
    (await defSel('[class="pg-cb pg-cb-not-selected"]')).click();
    (await defSel('[type="submit"]')).click();

    //Now in password screen
    var password = getPassword(await ( await defSel(
      '[class="pg-message-row ng-binding ng-scope pg-message-row-first"]'
    )).getText());
    (await defSel('[name="passwordField"]')).clear();
    (await defSel('[name="passwordField"]')).sendKeys(password);
    (await defSel('[type="submit"]')).click();

    //Now in Welcome to Essence Screen
    (await defSel('[class="pg-btn  pg-action-bar-button-highlighted"]')).click();

    //Now in change password screen
    (await defSel('[placeholder="your current password"]', 500)).sendKeys(password);
    (await defSel('[placeholder="write it your new password"]')).sendKeys(data.password);
    (await defSel('[placeholder="write it again"]')).sendKeys(data.password);
    (await defSel('[class="pg-btn  pg-action-bar-button-secondary"]')).click();
                            
    //Now at setting alias screen
    (await defSel('[type="text"]', 500)).sendKeys(getData('alias'));
    (await defSel('[class="pg-btn  pg-action-bar-button-highlighted"]', 500)).click();
    
    //Now in profile pic change screen
    (await defSel('[class="pg-btn  pg-action-bar-button-highlighted"]', 500)).click();

  } catch (err) {
    console.log(err);
  } finally {
    // await delay(10000);
    // driver.quit();
  }
}

function getData(field) {
  var val = data;

  switch (field) {
    case 'email':
      let temp = val.email.split('@');
      val.email = temp[0].substr(0, temp[0].length - 3) +
        (parseInt(temp[0].substr(temp[0].length - 3)) + 1) +
        '@' +
        temp[1];
      break;
    case 'alias':
      val.alias = val.alias.substr(0, val.alias.length - 3) +
        (parseInt(val.alias.substr(val.alias.length - 3)) + 1);
      break;
    case 'phone':
      val.phone = val.phone + 1;
      break;
    default :
      return data[field];
  }

  fs.writeFileSync('data.json', JSON.stringify(val, null, 2), 'utf8');
  return data[field];
}

// driver.quit();
async function defSel(cssSelector, putDelay = true) {
  try {
    var element = await driver.wait(until.elementLocated(By.css(cssSelector)));
    if(putDelay) {
      if(!isNaN(parseFloat(putDelay)) && isFinite(putDelay)){
        await delay(putDelay);
      } else {
        await delay(pause);
      }
    }
  } catch (err) {
    console.log(err);
  } finally {
    return element;
  }
}

function $(element) {
  driver.wait(until.elementLocated(element));
  return driver.findElement(element);
}

function delay(t) {
  return new Promise(resolve => setTimeout(resolve, t));
}

function getPassword(password) {
  password = password.substring(password.indexOf('[') + 1, password.indexOf(']'));
  return  password.split(':')[1];
}
```
