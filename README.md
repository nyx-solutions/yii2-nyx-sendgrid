Yii2 NYX SendGrid
-----------------

Yii2 Mailer extension for SendGrid with batch mailing support.

[![Latest Stable Version](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/v/stable)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![Total Downloads](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/downloads)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![Latest Unstable Version](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/v/unstable)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![License](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/license)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![Monthly Downloads](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/d/monthly)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![Daily Downloads](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/d/daily)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)
[![composer.lock](https://poser.pugx.org/nyx-solutions/yii2-nyx-sendgrid/composerlock)](https://packagist.org/packages/nyx-solutions/yii2-nyx-sendgrid)

## Installation

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

```
php composer.phar require --prefer-dist nyx-solutions/yii2-nyx-sendgrid
```

or add

```json
"nyx-solutions/yii2-nyx-sendgrid": "~2.0.0"
```

to the require section of your application's `composer.json` file.

Then configure your `mailer` component in your `main-local.php` (advanced) or `web.php` (basic) like so:

```php

'mailer' => [
    'class' => \nyx\mail\sendgrid\Mailer::class,
    'viewPath' => '@common/mail',
    // send all mails to a file by default. You have to set
    // 'useFileTransport' to false and configure a transport
    // for the mailer to send real emails.
    'useFileTransport' => false,
    'apiKey' => '[YOUR_SENDGRID_API_KEY]',
],

```

Do not forget to replace `apiKey` with your SendGrid API key. It must have permissions to send emails.

## Usage

### Single Mailing

```php

$user = \common\models\User::find()->select(['id', 'username', 'email'])->where(['id' => 1])->one();

$mailer = Yii::$app->mailer;

$message = $mailer->compose()
    ->setTo([$user->email => $user->username])      // or just $user->email
    ->setFrom(['alerts@example.com' => 'Alerts'])
    ->setReplyTo('noreply@example.com')
    ->setSubject('Hey -username-, Read This Email')
    ->setHtmlBody('Dear -username-,<br><br>My HTML message here')
    ->setTextBody('Dear -username-,\n\nMy Text message here')
    //->setTemplateId('1234')
    //->addSection('%section1%', 'This is my section1')
    //->addHeader('X-Track-UserType', 'admin')
    //->addHeader('X-Track-UID', Yii::$app->user->id)
    //->addCategory('tests')
    //->addCustomArg('test_arg', 'my custom arg')
    //->setSendAt(time() + (5 * 60))
    //->setBatchId(Yii::$app->mailer->createBatchId())
    //->setIpPoolName('7')
    //->attach(Yii::getAlias('@webroot/files/attachment.pdf'))
    ->addSubstitution('-username-', $user->username)
    ->send();

if ($message === true) {
    echo 'Success!';
    echo '<pre>' . print_r($mailer->getRawResponses(), true) . '</pre>';
} else {
    echo 'Error!<br>';
    echo '<pre>' . print_r($mailer->getErrors(), true) . '</pre>';
}

```

### Batch Mailing

If you want to send to multiple recipients, you need to use the below method to batch send.

```php

$mailer = Yii::$app->mailer;
//$batchId = Yii::$app->mailer->createBatchId();
//$sendTime = time() + (5 * 60);      // 5 minutes from now

foreach (User::find()->select(['id', 'username', 'email'])->batch(500) as $users)
{

    $message = $mailer->compose()
        ->setFrom(['alerts@example.com' => 'Alerts'])
        ->setReplyTo('noreply@example.com')
        ->setSubject('Hey -username-, Read This Email')
        ->setHtmlBody('Dear -username-,<br><br>My HTML message here')
        ->setTextBody('Dear -username-,\n\nMy Text message here');
        //->setTemplateId('1234')
        //->addSection('%section1%', 'This is my section1')
        //->addHeader('X-Track-UserType', 'admin')
        //->addHeader('X-Track-UID', Yii::$app->user->id)
        //->addCategory('tests')
        //->addCustomArg('test_arg', 'my custom arg')
        //->setSendAt($sendTime)
        //->setBatchId($batchId)
        //->setIpPoolName('7')
        //->attach(Yii::getAlias('@webroot/files/attachment.pdf'));

    foreach ( $users as $user )
    {
        // A Personalization Object Helper would be nice here...
        $personalization = [
            'to' => [$user->email => $user->username],      // or just `email@example.com`
            //'cc' => 'cc@example.com',
            //'bcc' => 'bcc@example.com',
            //'subject' => 'Hey -username-, Custom message for you!',
            //'headers' => [
            //    'X-Track-RecipId' => $user->id,
            //],
            'substitutions' => [
                '-username-' => $user->username,
            ],
            //'custom_args' => [
            //    'user_id' => $user->id,
            //    'type' => 'marketing',
            //],
            //'send_at' => $sendTime,
        ];
        $message->addPersonalization($personalization);
    }

    $result = $message->send();
}

if ($result === true) {
    echo 'Success!';
    echo '<pre>' . print_r($mailer->getRawResponses(), true) . '</pre>';
} else {
    echo 'Error!<br>';
    echo '<pre>' . print_r($mailer->getErrors(), true) . '</pre>';
}

```

**NOTE:** SendGrid supports a max of 1,000 recipients. This is a total of the to, bcc, and cc addresses.
