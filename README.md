# McGill Minerva Automatic register
The goal of this project is to use a cloud service like AWS or GCP and create a cron job in order to run them at specific intervals

The script first webscrappes VSB to determine if there is a place available in the class, then it will proceed to register. If you think that I'm stealing your credentials for minerva, just go look at the source code, it is 148 lines. 

**WARNING**: This program is to be used with people with knowledge with cloud infrastructure and node.js. I tried my best to make the instructions clear but anyone with any prior knowledge should be able to figure things out. Contact me if you need help!
### Dependencies
* Node.js with npm
* Puppeteer
* SendGrid for email services (Optional)

## How To use locally
1. Go on VSB and copy the URL after you have chosen your classes.
2. Modify the content of `config` JSON in `src/multiclassRegister.js` to match yours. 
4. The CRN is a list of all CRN of the class you would like to register. Those can be found on VSB also.
4. To run, simply write `npm install` then `npm start` from the root of this project in your command line.


### How to host on a Google Cloud Service
1. Create a _Cloud Function_ with content of `src/multiclassRegister.js`. Use HTTP Trigger, 512mb of ram, function to execute is `Registration`, allow unauthorized requests (makes life easier).
2. Modify the `config` json variable as needed in the script.
3. Use the following `package.json` for GCP since the `npm start` is slightly different from the one located at the root of the project.
```
{
  "name": "mcgill-autoregistration",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "index.Registration"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "puppeteer": "^2.1.1",
    "@sendgrid/mail": "^6.5.5"
  }
}
```
4. Once deployed, create _Cloud Scheduler_ that calls the url generated by the function with a `GET` request. Choose an appropriate frequency (The more you run the function, the higher the costs!)\
4.1 I'm running mine every 30mins or so, if you would like to customize the time use this [amazing tool](https://crontab.guru/)

### To use SendGrid
If you do not want to have email notifications, set `"wantEmail": false` in `config.wantEmail`.\
To receive emails follow these steps:
1. Sign up on [_sendgrid_](https://sendgrid.com/).
2. Copy down the API key and use it in the program.
3. Write your email on line 110 of `multiclassRegister.js`.

A free trial should be more than enough for our purposes as we get 100 free emails a day.

### Documentaion of Config
Below, all the fields of the `config` JSON are described
* `email`: McGill Minerva username
* `password`: McGill Minerva password
* `term`: Term you wish to register for, found in VSB URL (ex. term=202005)
* `CRN`: List of CRNs you wish to register for, found in VSB at bottom of screen
* `url`: Select the classes you want on VSB for your selected term then copy the URL over
* `wantEmail`: Set to true if you want an email notification upon successful registration
* `notifEmail`: Where to send notification email to, leave as `""` if wantEmail is false
* `sgApiKey`: Received when registered for sendgrid, leave as `""` if wantEmail is false

### Couple notes:
* There is a **MAXIMUM** of 10 CRN that you can have, those included the tutorial's CRN.
* Each CRN must be within double quotes `"YOUR_CRN"`
* The `semester`field needs to follow this pattern: YYYYMM
* YYYY represents the academic year
* MM is the semester where Fall is 09, Winter is 01, Summer is 05.\
For example, if I want to register for fall 2020, I would write 202009.
* The script checks on vsb first to see if there are spots in the class. If so, it will proceed to register on Minerva. 
This is to prevent using all the registration limit which is around 1000 per semester.
* To run on the cloud, we need to use the headless version of chrome ` headless: true`, if you want to see the browser,
simply set to ` headless: false` in `multiclassRegister.js`. 
* You can have conflicting classes schedule in VSB, the program will still work like this:
![image](https://user-images.githubusercontent.com/43629633/78501009-e05f6700-7727-11ea-91d5-e3f98ce7e77b.png)\
You will need to find the CRN of them separately because VSB will not generate a schedule for you.


### Exmaple of Config
Currently I want to register to all the classes shown in the image for the summer term, my `config` variable will look like this:

```
const config = {
    "regEmail": "firstname.lastname@mail.mcgill.ca",
    "password": "password",
    "term": "202005",
    "CRN": ["527", "528", "491", "850", "285", "288", "289"],
    "url": "https://vsb.mcgill.ca/vsb...",
    "wantEmail": true,
    "notifEmail": "",
    "sgApiKey": "SG.xxx..."
};
```
Notice how CRN 527 (lecture) and 528 (tutorial) for FACC300 is inputted.

## Upcoming features and todos
- [x] Ping VSB, and check if the class is free. If yes, proceed to register. 
- [ ] Log files generated by local script.
- [x] Make code more modular (Help needed!). 
- [ ] Determine the update frequency on VSB

## Feature request 
Create an issue and we will talk about it!
