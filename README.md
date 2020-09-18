# Regressor
Visual Regression Tool to catch css bugs

Aye Spy is a high performance visual regression tool to catch UI regressions. 

## Inspiration

Regressor takes inspiration from existing projects such as [Wraith](https://github.com/BBC-News/wraith) and [BackstopJs](https://github.com/garris/BackstopJS).

I have found visual regression testing to be one of the most effective ways to catch regressions. It's a great tool to have in your pipeline, but the current solutions on the market were missing one key component we felt was essential for a great developer experience... performance!

With the correct set up you can expect 40 comparisons running under a minute.


## Concept

The idea behind visual regression is essentially image comparison over time.

As you make changes to your site Regressor will take new images.

As you make changes to your site Regressor will take new images of your site

Aye Spy will then compare the latest images against the baseline images.

If there are differences the build fails. A report can be generated and gives you the opportunity to check the differences are expected.

If they are expected, update the baseline images

## Setup

In order to get the most out of Regressor we recommend 

  - Using the [selenium images from docker hub](https://hub.docker.com/u/selenium/) for consistent repeatable state 
  - Cloud storage (currently supporting Amazon S3)

Create an S3 bucket to store your images. It will also need to have the correct permissions set up.

Example config:

```
{
    "gridUrl": "http://selenium-grid:4444/wd/hub",
    "baseline": "./baseline",
    "latest": "./latest",
    "generatedDiffs": "./generatedDiffs",
    "report": "./reports",
    "remoteBucketName": "regressor-example",
    "remoteRegion": "us-west-1",
    "scenarios": [
      {
        "url": "http://www.bellhelmets.com/",
        "label": "homepage",
        "removeSelectors": [".header-banner"],   // remove elements that are not static on refresh such as adverts
        "viewports": [{"height": 2400, "width": 1024, "label": "large"}],
        "cookies": [
          {
            "name": "cookie_name",
            "value": "cookie_value"
          }
        ],
        "waitForSelector": ["footer"],
        "onReadyScript": './scripts/clickSelector.js',
        "wait": 2000 // implicitly wait before taking a snap
      }
    ]
  }
```

## Using S3 Storage for images

In order to use the S3 Storage capabilities you will need to export some aws credentials:

```
export AWS_SECRET_ACCESS_KEY=secretkey
export AWS_ACCESS_KEY_ID=keyid
```

Create an S3 bucket to store your images. 
Make sure to configure the bucket policy to allow viewing of objects.

## on Ready Script

For scenarios where you need to interact with the page before taking a screenshot, a script can be run which has the [selenium-webdriver](https://github.com/SeleniumHQ/selenium/wiki/WebDriverJs) driver exposed. 

Example script:

```
const By = require('selenium-webdriver').By;
const clickElement = async browser => {
  await browser.findElement(By.css('#buttonId"]')).click();
};
module.exports = clickElement;
```

## Running

Take the latest screenshots for comparison:

`regressor snap --browser chrome --config config.json --remote`

Set your latest screenshots as the baselines for future comparisons:

`regressor update-baseline --browser chrome --config config.json --remote`

Run the comparison between baseline and latest:

`regressor compare --browser chrome --config config.json --remote`

Generate the comparison report: 

`regressor generate-report --browser chrome --config config.json --remote`