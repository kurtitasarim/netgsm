## Introduction
With this package, you can send sms and get sms reports by using Netgsm service on laravel 6.x.

## Contents

- [Installation](#installation)
	- [Setting up the Netgsm service](#setting-up-the-Netgsm-service)
- [Usage](#usage)
    - [Service Methods](#methods)
    - [SMS Sending](#sms-sending)
	    - [Available SMS Interfaces](#available-sms-interfaces)
	- [Reporting](#reporting)
	    - [Available Reporting Interfaces](#available-report-interfaces)
- [Testing](#testing)
- [Security](#security)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)


## Installation

You can install the package via composer:

```bash
composer require tarfin-labs/netgsm
```
### Setting up the Netgsm service

Add the environment variables to your `config/netgsm.php`:

```php
// config/netgsm.php
...
'credentials' => [
    'user_code' => env('NETGSM_USER_CODE'),
    'secret' => env('NETGSM_SECRET'),
],
'defaults' => [
    'language' => env('NETGSM_LANGUAGE'),
    'header' => env('NETGSM_HEADER'),
    'sms_sending_method' => env('NETGSM_SMS_SENDING_METHOD'), //available methods (xml, http)
],
...
```

Add your Netgsm User Code, Default header (name or number of sender), and secret (password) to your `.env`:

```php
// .env
...
NETGSM_USER_CODE=
NETGSM_SECRET=
NETGSM_LANGUAGE=
NETGSM_HEADER=
],
...
```
### Usage
#### Service Methods

 - Netgsm::sendSms(AbstractSmsMessage $message):string $JobId

Sends an SMS message to the phone number on the message object passed as a parameter. 
If the message is sent successfully, a job id returned from the netgsm api service is returned.

 - Netgsm::getReports(AbstractNetgsmReport $report): ?Collection
 
Returns an sms report based on the report object passed as a parameter.

#### Sms Sending with Using Notification Channel

You can use the channel in your `via()` method inside the notification:

``` php
use TarfinLabs\Netgsm\NetGsmChannel;
use TarfinLabs\Netgsm\NetGsmSmsMessage;
use Illuminate\Notifications\Notification;

class NewUserRegistered extends Notification
{
    public function via($notifiable)
    {
        return [NetGsmChannel::class];
    }

    public function toNetgsm($notifiable)
    {
        return (new NetGsmSmsMessage("Hello! Welcome to the club {$notifiable->name}!"));
    }
}
```

You can add recipients (single value or array)

``` php
return (new NetGsmSmsMessage("Your {$notifiable->service} was ordered!"))->setRecipients($recipients);
```

You can also set the sending date range of the message. (It does not work on otp messages.)

``` php
$startDate = Carbon::now()->addDay(10)->setTime(0, 0, 0);
$endDate = Carbon::now()->addDay(11)->setTime(0, 0, 0);

return (new NetGsmSmsMessage("Great note from the future!"))
->setStartDate($startDate)
->setEndDate($endDate)
```

You can set authorized data parameter (It does not work on otp messages.)

If this parameter value is sent as true, only sms will be sent to phone numbers that have data permission.

``` php
return (new NetGsmSmsMessage("Your {$notifiable->service} was ordered!"))->setAuthorizedData(true);
```

Additionally you can change header

``` php
return (new NetGsmSmsMessage("Your {$notifiable->service} was ordered!"))->setHeader("COMPANY");
```

You can use NetGsmOtpMessage instead of NetGsmSmsMessage to send an otp message.

``` php
return (new NetGsmOtpMessage("Your {$notifiable->service} OTP Token Is : {$notifiable->otp_token}"));
```

For more information on sending OTP messages [Netgsm OTP SMS Documentation]("https://www.netgsm.com.tr/dokuman/#otp-sms")

#### Sms Sending with Using Netgsm Facade

You can also send sms or otp messages using Netgsm Facade directly:

``` php

$message = new NetgsmSmsMessage("Your {$notifiable->service} was ordered!");
->setHeader("COMPANY")
->setRecipients(['5051234567','5441234568']);

Netgsm::sendSms($message);

```

#### Reporting

You can get sms reports by date range or netgsm bulk id. 

In order to receive a report, a report object must be created.

``` php
$report = new NetgsmSmsReport();
```

##### Available Report Interfaces:

 - NetgsmSmsReport (basic reports): [Documentation](https://www.netgsm.com.tr/dokuman/#http-get-rapor)
 - NetgsmSmsDetailReport (detailed reports) [Documentation](https://www.netgsm.com.tr/dokuman/#detayl%C4%B1-rapor)


##### Object Parameters

| Method        | Description | Type | Required | NetgsmSmsReport Support | NetgsmSmsDetailReport Support | 
| :------------|: ---- | :---- | :-------- | :--------------- | :--------------------- |
| setStartDate() |Start Date| Carbon | No | Yes | Yes |
| setEndDate() | End Date |Carbon | No | Yes | Yes |
| setBulkId() | Netgsm Job Id | Integer | No | Yes | Yes |
| setStatus() | Message Status |Integer | No | Yes | No |
| setPhone() | Phone Number | String Seperated by comma | No | Yes | Yes |
| setHeader() | Header | String | No | Yes | Yes |
| setVersion() | API Version |  Integer | No | Yes | Yes |

##### Sample Usage

You can get the sms report by passing the report object to the Netgsm::getReports method. 
If successful, sms report results will be returned as a collection. 

``` php
// Start and end dates
        $startDate = Carbon::now()->subDay()->setTime(0, 0, 0);
        $endDate = Carbon::now()->setTime(0, 0, 0);
        
        $report = new NetgsmSmsReport();
        $report->setStartDate($startDate)
            ->setEndDate($endDate);

        $reports = Netgsm::getReports($report);
        Netgsm::getReports($report);
```

Fields in the report result may differ depending on the specified report type and the report version parameter sent.

Report Results

| Field | Version | NetgsmSmsReport Support | NetgsmSmsDetailReport Support
| :----- | :----- | :---------------------- | :-------------------- |
| jobId  | All    | Yes                     | Yes
| message | 1     | No                      | Yes
| phone   | All   | Yes                     | No
| status  | All   | Yes                     | Yes
| operatorCode | 2 | Yes                   | No
| length | 2 | Yes | No
| startDate | 2 | Yes | No
| startTime | 2 | Yes | No
| endDate | All | No | Yes
| errorCode | 2 | Yes | No
| header | All | No | Yes
| total | All | No | Yes


### Testing

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

### Security

If you discover any security related issues, please email development@tarfin.com instead of using the issue tracker.


## Credits

- [Turan Karatuğ](https://github.com/tkaratug)
- [Faruk Can](https://github.com/frkcn)
- [Yunus Emre Deligöz](https://github.com/deligoez)
- [Hakan Özdemir](https://github.com/hozdemir)
- [All Contributors](../../contributors)

###License
Laravel Netgsm is open-sourced software licensed under the MIT license.
