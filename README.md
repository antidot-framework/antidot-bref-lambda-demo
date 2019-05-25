# Running Antidot Framework in AWS Lambda using Bref

This guide helps you run Antidot framework applications on AWS Lambda using Bref. These instructions are kept up to 
date to target the latest Symfony version.

A demo application is available on GitHub at 
[github.com/antidot-framework/antidot-bref-lambda-demo](https://github.com/antidot-framework/antidot-bref-lambda-demo).

## Setup

Assuming your are in existing Antidot Framework project, let's install Bref via Composer:

```
composer require mnapoli/bref
```

Then let's create a `template.yaml` configuration file (at the root of the project) optimized for Antidot Framework:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
    Website:
        Type: AWS::Serverless::Function
        Properties:
            FunctionName: 'antidot-fw-website'
            CodeUri: .
            Handler: public/index.php
            Timeout: 30 # in seconds (API Gateway has a timeout of 30 seconds)
            MemorySize: 1024
            Runtime: provided
            Layers:
                - 'arn:aws:lambda:us-east-1:209497400698:layer:php-73-fpm:6'
            Events:
                HttpRoot:
                    Type: Api
                    Properties:
                        Path: /
                        Method: ANY
                HttpSubPaths:
                    Type: Api
                    Properties:
                        Path: /{proxy+}
                        Method: ANY

    Console:
        Type: AWS::Serverless::Function
        Properties:
            FunctionName: 'antidot-fw-console'
            CodeUri: .
            Handler: bin/console
            Timeout: 120 # in seconds
            Runtime: provided
            Layers:
                - 'arn:aws:lambda:us-east-1:209497400698:layer:php-73:6' # PHP
                - 'arn:aws:lambda:us-east-1:209497400698:layer:console:6' # The "console" layer

Outputs:
    DemoApi:
        Description: 'URL of our function in the *Prod* environment'
        Value: !Sub 'https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/'
```

Now we still have a few modifications to do on the application to make it compatible with AWS Lambda.

Since [the filesystem is readonly](/docs/environment/storage.md) except for `/tmp` we need to customize where the cache 
and the logs are stored in the `config/config.php` and `config/cli-config.php` files. 

```php
<?php
// config/config.php
$cacheConfig = [
    'config_cache_path' => 'tmp/cache/config-cache.php',
];
...
```

```php
<?php
// config/cli-config.php
...
$cacheConfig = [
    'cli_config_cache_path' => 'tmp/cache/cli-config-cache.php',
];
...
```

## Deploy

Your application is now ready to be deployed. Follow [the deployment guide](/docs/deploy.md#deploying-with-sam).

## Console

As you may have noticed, we define a function of type "console" in `template.yaml`. That function is using the [Console runtime](/docs/runtimes/console.md), which lets us run the Symfony Console on AWS Lambda.

To use it follow [the "Console" guide](/docs/runtimes/console.md).

## Logs

We need to override monolog config

```yaml
# config/services/dependencies.prod.yaml
  monolog:
    handlers:
      default:
        type: 'stream'
        options:
          stream: 'php://stderr'
          level: 400
```

The secrets (e.g. database passwords) must however not be committed in this file: define them in the [AWS Console](https://console.aws.amazon.com).
