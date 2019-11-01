## Magento 2 module AWS EventBridge integration

Send Magento's events to [Amazon EventBridge](https://aws.amazon.com/eventbridge/) service to be able to integrate Magento with many different AWS serverless services. 

![Packagist](https://img.shields.io/packagist/v/bitbull/magento2-module-awseventbridge)

![Magento](https://img.shields.io/badge/magento-~2.2.7-orange)

![PHP](https://img.shields.io/packagist/php-v/bitbull/magento2-module-awseventbridge)

## Installation Instructions

Install this module using composer: 

```bash
composer require bitbull/magento2-module-awseventbridge
```

## IAM permissions required

Create a new [IAM Policy](https://docs.aws.amazon.com/en_us/AWSEC2/latest/UserGuide/iam-policies-for-amazon-ec2.html) with these content:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "events:PutEvents"
      ],
      "Effect": "Allow",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "events:source": "example.com"
        }
      }
    }
  ]
}
```
change `events:source` according to your module configuration.

read more about IAM permissions at: 
- https://docs.aws.amazon.com/en_us/AmazonCloudWatch/latest/events/auth-and-access-control-cwe.html
- https://docs.aws.amazon.com/en_us/AmazonCloudWatch/latest/events/policy-keys-cwe.html

If you are using EC2 instance add this policy to attached [IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html).

read more about using IAM Role with EC2 at:
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html

If your are running Magento locally, on-premises or with an other Cloud Provider different from AWS follow these steps:

1. Create a new [AWS IAM User](https://docs.aws.amazon.com/en_us/IAM/latest/UserGuide/id_users.html)
2. Attach the created policy
3. Generate access keys for the user

read more about creating IAM users at:
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_change-permissions.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html

## Setup

Go to "Stores" > "Configuration" > "Services" > "AWS EventBridge", then start configuring this module.

### Credentials

You can set your access keys for IAM Users:  

![Credentials keys](./doc/imgs/config-credentials-keys.png?raw=true)

Retrieving from environment variables AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY:

![Credentials env](./doc/imgs/config-credentials-env.png?raw=true)

Using EC2 Instance Role:

![Credentials ec2](./doc/imgs/config-credentials-ec2.png?raw=true)

### Options

Edit module options:

![Options](./doc/imgs/config-options.png?raw=true)

- Set the correct region where you want to receive events, for example "eu-west-1".
- Set event source name with a value that can be filtered (`events:source`) when you connect to these events.
- Set event bus name, leave empty to use your account default.
- Enable tracking to add `tracking` property to data object.
- Enable debug mode if you want a more verbose logging in `var/log/aws-eventbridge.log` log file.
- Enable CloudWatch Events fallback to use this service instead of EventBridge (for backward compatibility).
- Enable dry run mode to activate the module actions and integrations without actually sending events data.

![Events](./doc/imgs/config-cart-events.png?raw=true)

These sections contain a list of supported events that can be activated and used to trigger Lambda functions.

## Events 

This module inject observers to listen to Magento 2 events, elaborate the payload and then send the event to AWS services.

### Data specification

Event will be pass data into `Details` event property:
```php
(
    [sku] => WJ12-S-Blue
    [qty] => 1
    [tracking] => Array
        (
            [transport] => HTTP
            [hostname] => f3a501ad4988
            [time] => 1566821650383
        )
)
```

Additionally (activating tracking option in backend options) every event will be enriched with `tracking` property that contain infos about client, session and framework, for example:
```php
[tracking] => Array
    (
        [transport] => HTTP
        [hostname] => f3a501ad4988
        [time] => 1566594699836
        [storeId] => 1
        [version] => Array
            (
                [module] => dev-master
                [php] => 7.1.27-1+ubuntu16.04.1+deb.sury.org+1
                [magento] => 2.2.7
            )
        [user] => Array
            (
                [id] => 3
                [username] => fabio.gollinucci
                [email] => fabio.gollinucci@bitbull.it
            )
        [ip] => 172.17.0.1
        [userAgent] => Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    )
```
when using Magento CLI `user` is based on the system user that execute the command:
```php
[tracking] => Array
    (
        [transport] => SHELL
        [hostname] => f3a501ad4988
        [time] => 1566821355758
        [storeId] => 1
        [version] => Array
            (
                [module] => dev-master
                [php] => 7.1.27-1+ubuntu16.04.1+deb.sury.org+1
                [magento] => 2.2.7
            )
        [user] => Array
            (
                [name] => www-data
                [passwd] => x
                [uid] => 1000
                [gid] => 33
                [gecos] => www-data
                [dir] => /var/www
                [shell] => /usr/sbin/nologin
            )
    )
``` 

### Supported Events

Here a list of supported events that can be enabled:

#### Cart events

`CartProductAdded`
A product is added to cart by a customer

`CartProductUpdated`
A cart is updated by a customer

`CartProductRemoved`
A product is removed from cart by a customer

#### Admin user events

`UserLoggedIn`
An admin user logged in

`UserLoggedOut`
An admin user logged out

`UserLoginFailed`
An admin user failed login

#### Customers events

`CustomerLoggedIn`
A customer user logged in

`CustomerLoggedOut`
A customer user logged out

`CustomerLoginFailed`
A customer user failed login

`CustomerSignedUp`
A customer user sign up

`CustomerSignedUpFailed`
A customer user failed sign up

#### Newsletter events

`NewsletterSubscriptionChanged`
A customer user change newsletter subscription preference

#### Order events

`OrderPlaced`
A customer place a new order

#### Cache events

`CacheFlushAll`
An admin user flush the cache

`CacheFlushSystem`
An admin user flush system cache

`CacheFlushCatalogImages`
An admin user flush catalog images cache

`CacheFlushMedia`
An admin user flush media cache

`CacheFlushStaticFiles`
An admin user flush static files cache
