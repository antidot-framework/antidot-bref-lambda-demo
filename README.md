# Running Antidot Framework in AWS Lambda using Bref

[This guide](https://antidotfw.io/#/framework/running-application) helps you run Antidot framework applications on AWS Lambda using Bref. 

## Requirements

* PHP ^7.4|^8.0
* [Bref.sh](https://bref.sh/)
* [Antidot Framework](https://antidotfw.io)
* [Serverless Framework](https://www.serverless.com/)

## Usage

With PHP8 it will work out of the box

```bash
git clone git@github.com:antidot-framework/antidot-bref-lambda-demo.git lambda
cd lambda
composer install --prefer-dist --optimize-autoloader --no-dev
serverless deploy
```

With PHP7.4 change the runtime in `serverless.yaml` file.


