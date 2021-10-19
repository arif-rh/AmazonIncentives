# Amazon Incentives - Gift on Demand standa alone class

This is a abastract from (https://github.com/kamerk22/AmazonGiftCode) to be not dependend on Laravel base code.

Uses .env file to load configuration data

## _ENV variables needed

* AWS_GIFT_CARD_KEY
* AWS_GIFT_CARD_SECRET
* AWS_GIFT_CARD_PARTNER_ID
* AWS_GIFT_CARD_ENDPOINT
* AWS_GIFT_CARD_CURRENCY
* AWS_DEBUG (1/0)

## How to use

The class must be loaded with an autoloader (see test/autoloader.php for example).

The above _ENV variables must be set (Except AWS_DEBUG, defaults to off).

### create gift card

```php
$aws_gc = Amazon\AmazonIncentives::make()->buyGiftCard((float)$value);
// the two below are need if we want to cancel the card
// get gift card id (gcID)
$aws_gc->getId();
// get creation request id (creationRequestId)
$aws_gc->getCreationRequestId();
// the one below must be printed to the user
$aws_gc->getClaimCode();
// check status (SUCCESS/RESEND/FAILURE)
$aws_gc->getStatus();
// others:
// getAmount, getCurrency
```

### cancel gift card

```php
// use getCreationRequestId() and getId() from request
$aws_gc = Amazon\AmazonIncentives::make()->cancelGiftCard($creation_request_id, $gift_card_id);
// return is as above
```

### check balance

```php
$aws_gc = Amazon\AmazonIncentives::make()->getAvailableFunds();
```

## Exceptions

If the HTTPS request does not return 220 OK it will throw an exception.

The error code is the curl handler error code.
The error message is json encoded array with the layout
```php
[
	'status' => 'AWS Status FAILURE or RESEND',
	'code' => 'AWS Error Code Fnnn',
	'type' => 'AWS Error info',
	'message' => 'AWS long error message',
	'log_id' => 'If logging is on the current log id',
	'log' => 'The complete log collected over all calls',
]
```

`status`, `code` and `type` must be checked on a failure.

**NOTE**: if code is E999 then this is a request flood error:
In this case the request has to be resend after a certain waiting period.

## Debugging

If AWS_DEBUG is set to 1 and internal array will be written with debug info.

The Amazon\Debug\AmazonDebug class handles all this.

In the Amazon\AWS\AWS main class the debugger gets set
* setFlag that turns debugger on/off
* setId (to set unique id for each run)

New entries can be written with

`AmazonDebug::writeLog(['foo' => 'bar']);`

On sucessful run the log data is accessable with `$aws->getLog()`
On exception the log data is in the error message json (see exceptions)