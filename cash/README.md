# Cash

> index.js, constants.js and cash.js

![](images/feature-currency-conversion.jpg)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [🐣 Introduction](#-introduction)
- [📁 The different files](#-the-different-files)
  - 🏃‍♀️ constants.js
  - 🏃‍♀️ cash.js
  - 🏃‍♀️ index.js

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## 🐣 Introduction
The current code is about converting currency, for example USD into Euro, British Pund Sterlingand Japanese Yen.

```sh
❯ cd /path/to/workspace/3-musketeers/cash
❯ npm i
❯ node bin/index.js

# this is what we can in the console
√ 0.883 (EUR) Euro
√ 0.773 (GBP) British Pound Sterling
√ 110.549 (JPY) Japanese Yen

Conversion of USD 1
```

Let's see in detail how to do a program like this.


## 📁 The different files
Before anything else, create a new folder named `cash`, with 2 folders in it named `bin` and `library`.   
`bin` will be the folder in which we will stock all of our `file.js` (meaning `cash.js`, `constants.js` and `index.js`, that we will present later).   
`library` will be the folder in which we will stock a JSON file with every currency symbol definitions.

After creating those folder, you need to install `package.json` by doing in the console :

```sh
❯ npm install
```

Then check in the `package.json` created that it looks like this :

```sh
{
	"name": "cash",
	"license": "MIT",
	"main": "index.js",
	"bin": {
		"cash": "./bin/index.js"
	},
	"dependencies": {
		"chalk": "^2.4.1",
		"conf": "^2.1.0",
		"got": "^9.4.0",
		"meow": "^5.0.0",
		"money": "^0.2.0",
		"ora": "^3.0.0"
	}
}
```

We can now begin to code the currency conversion program.


### 🏃‍♀️ constants.js
In your `bin` folder, create a new file named `constants.js`.

```sh
const API = 'https://api.exchangeratesapi.io/latest';
const DEFAULT_TO_CURRENCIES = ['USD', 'EUR', 'GBP', 'JPY'];

module.exports = {
  API,
  DEFAULT_TO_CURRENCIES
}
```

The purpose of this code is to get all the exchange rate for all currency based on the Euro, according to the latest update of the [API](https://api.exchangeratesapi.io/latest). For example, we can know that 1 EUR equals 1.1328 USD.   
Then, in `DEFAULT_TO_CURRENCIES` we stock the currencies we are interesting in.   

To be able to use those constants in an other file (meaning here in `cash.js`), we export them.


### 🏃‍♀️ cash.js
In your `bin` folder, create a new file named `cash.js`.   
The very first step is to import the libraries we will have to use for the function to implement.

```sh
'use strict';

const got = require('got');
const money = require('money');
const chalk = require('chalk');
const ora = require('ora');
const currencies = require('../lib/currencies.json');

const {API} = require('./constants');
```

To install the libraries, we need to install them thanks to npm like (be careful to be in the right path of your project) :

```sh
❯ npm install got money chalk ora
```

`got` is a human-friendly and powerful HTTP request library.   
`money` is a library for realtime currency conversion and exchange rate calculation, from any currency, to any currency.   
`chalk` is a library for terminal string styling done right.   
`ora` is a library for an elegant terminal spinner.   

Then we require the `currencies.json` file and the const `API` created in the `constants.js` file that we have exported.   

```sh
const cash = async command => {
	const {amount} = command;
	const from = command.from.toUpperCase();
	const to = command.to.filter(item => item !== from).map(item => item.toUpperCase());

  // ...

};
```
To define `from` and `to` for the conversion from a certain currency to an other.

```sh
const cash = async command => {
  // ...

	console.log();
	const loading = ora({
		text: 'Converting...',
		color: 'green',
		spinner: {
			interval: 150,
			frames: to
		}
	});

	loading.start();

  // ...

};
```
To design the execution of the program, with the price in green and a spinner to get animation.

```sh
const cash = async command => {
  // ...

	await got(API, {json: true}).then(response => {
		money.base = response.body.base;
		money.rates = response.body.rates;

		to.forEach(item => {
			if (currencies[item]) {
				loading.succeed(`${chalk.green(money.convert(amount, {from, to: item}).toFixed(3))} ${`(${item})`} ${currencies[item]}`);
			} else {
				loading.warn(`${chalk.yellow(`The "${item}" currency not found `)}`);
			}
		});

		console.log(chalk.underline.gray(`\nConversion of ${chalk.bold(from)} ${chalk.bold(amount)}`));

	}).catch(error => {
		if (error.code === 'ENOTFOUND') {
			loading.fail(chalk.red('Please check your internet connection!\n'));
		} else {
			loading.fail(chalk.red('Internal server error :(\n${error}'));
		}
		process.exit(1);
	});
};
```
The got method is used to get from the API the information we are interested in. The forEach loop is converting the `money.base` currency
fixing the `money.rates` to 3 decimals.

To be able to use those constants in an other file (meaning here in `index.js`), we export them.


### 🏃‍♀️ index.js
In your `bin` folder, create a new file named `index.js`.   
The very first step is to import the libraries we will have to use for the function to implement.

```sh
'use strict';

const Conf = require('conf');
const meow = require('meow');
const chalk = require('chalk');
const cash = require('./cash.js');

const config = new Conf();
const argv = process.argv.slice(2);

const {DEFAULT_TO_CURRENCIES} = require('./constants');
```

To install the libraries, we need to install them thanks to npm like (be careful to be in the right path of your project) :

```sh
❯ npm install conf meow
```

`conf` is a human-friendly and powerful HTTP request library.   
`meow` is a library for realtime currency conversion and exchange rate calculation, from any currency, to any currency.   
We already install chalk so we do not need to install it a second time.   

Then we require `cash.js` because it corresponds to our conversion program. An we also need `DEFAULT_TO_CURRENCIES` from `constants.js`.   

```sh
const cli = meow(`
	Usage
		$ cash <amount> <from> <to>
		$ cash <options>
	Options
		--set -s 			Set default currencies
	Examples
		$ cash 10 usd eur pln
		$ cash --set usd aud
`);

if (argv.indexOf('--save') !== -1 || argv.indexOf('-s') !== -1) {
	config.set('defaultFrom', argv[1] || config.get('defaultFrom', 'USD'));
	config.set('defaultTo', (argv.length > 2) ? process.argv.slice(4) : config.get('defaultTo', DEFAULT_TO_CURRENCIES));
	console.log(chalk.green('Saved default currencies to ' + config.path));
	process.exit(0);
}

const command = {
	amount: parseFloat(argv[0]) || 1,
	from: argv[1] || config.get('defaultFrom', 'USD'),
	to: (argv.length > 2) ? process.argv.slice(4) : config.get('defaultTo', DEFAULT_TO_CURRENCIES)
};

cash(command);
```
This code is about defining what currency we want to convert and know more about the exchange rate.   
Then we execute `command` using `cash` which is the reference to the function written in `cash.js`.
