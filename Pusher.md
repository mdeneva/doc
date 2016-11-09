#Integrating Pusher with Drupal 8

##Introduction

Drupal is an open-source web content management platform for content, community, blogging and e-commerce. It currently serves at least 2.2% of all websites worldwide.
Since it's an open-source platform there are thousands of contributors world-wide and modularity is one of its core principles.
In this tutorial we'll be looking at integrating Pusher realtime message broadcasting into your Drupal websites or apps.

###What are Push Notifications

In essence push notifications are similar to instant messages, but are targeted at the users of a particular application. They are used to provide important real time updates and in the case of a mobile application can work even if the device is locked or the application is not currently running (provided the application is installed and the user opted in to receive push notifications). Similarly, push notification is also supported in modern browsers and is able to present users with updates and information even when their browser is not open at the application site.
Having said that, support for push messages introduces additional complexity as different vendors use different APIs and onboarding processes to support push notifications. For example, supporting Android, iOS and web browsers might require different implementations in your application.

###Pusher's Advantage

Using Pusher allows you to use unified API to deliver instant updates and notifications to your customers across all platforms.
With Pusher you can instantly update browsers, apps and IoT devices with new messages, content or events. Our event-based abstraction makes it simple to bind UI interactions to events that are triggered from any client or server. Once configured your push notification service will work on all platforms and devices your customers use.

##Integrating Pusher

###Prerequisites
* Existing Drupal 8.x installation with **Composer Manager** and **Entity** drupal modules installed.
* You have created an account at [Pusher](https://Pusher.com "Pusher's Homepage")
* You will need your app_id, key and secret.
* You have enabled client events on your App settings tab after logging into [Pusher](https://Pusher.com "Pusher's Homepage")


###Installing the Pusher Notification Module
1. Download or copy the link of the module from the [Pusher Notification Module Download Page ](https://www.drupal.org/project/push_notifications).
2. Install it by going to your Drupal extensions admin page and either upload it or paste the link.
3. Run an update with Drush to pull in dependencies (aka “drush up”).
4. Configure the module (`<name_of_your_Drupal_website>/admin/config/Pusher_integration`). Enter your Pusher *app_id*, *app key* and *secret* - those can be seen when you login into the  [Pusher](https://Pusher.com "Pusher's Homepage") website and create an app. Choose the same cluster you are using for your app to be used by the Drupal module and save your configuration.
5. Enable the module.
6. Save your configuration and check for any additional modules that need to be installed.


###Configuring the Channel-Path-Map

To use Pusher, you must create a mapping between the Pusher channels and your pages/paths.
For each specific page/path on your website, you need to configure a channel.
Open the <b>Channel-Path-Map</b> tab and enter a Pusher channel name and a *path pattern*. Since the path patterns support regex, you can quickly and easily create global channels that affect your whole site, or just certain sections of your Drupal website.

Pusher CHANNEL NAME can contain the following values:

* presence-ANYSTRING: to create a presence channel
* private-ANYSTRING: to create a private channel
* ANYSTRING: Without <b>presence-</b> or <b>private-</b> in it, to create a public channel

>The Pusher Notification module will automatically set up the Pusher connection for you (based on your config settings). It can optionally create a presence channel automatically. All of those setting can be changed in the admin panel of your Drupal website.

###Setting up the Dependency
In your *.info.yml file, you need to set up the dependency so that Drupal knows that installing your module will require installing this one as well.

```yaml
name: MyModule
type: module
description: Some description here.
package: MyModule
core: 8.x
dependencies:
  - Pusher_integration:Pusher_integration
```

###Configuring the Server-Side PHP

On the server side (in your app or module) you can simply broadcast commands as follows:

```java
    use Drupal\Pusher_integration\Controller\PusherController;

    ...

    $this->Pusher = new PusherController( $configFactory, $currentUser );

    $data = array('someVar'  => 'Some value',
                  'otherVar' => 'Some other value');

    $this->Pusher->broadcastMessage( $this->configFactory,
				                            'my-channel-name-here',
                                    'my-event-name-here',
                                     $data );
```


Here is a more pseudo-complete example, with dependency injection:


```java
namespace Drupal\my_module\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Config\ConfigFactory;
use Drupal\Core\Session\AccountInterface;

use Drupal\Pusher_integration\Controller\PusherController;

class MyController extends ControllerBase {

  protected $configFactory;
  protected $currentUser;
  protected $Pusher;

  public function __construct( ConfigFactory $configFactory, AccountInterface $account )
  {
    $this->configFactory = $configFactory;
    $this->currentUser = $account;

    $this->Pusher = new PusherController( $configFactory, $currentUser );
  }

  /**
   * {@inheritdoc}
   */
  public static function create(ContainerInterface $container) {
    return new static(
      $container->get('config.factory'),
      $container->get('current_user'),
    );
  }

  ...

  public function foo()
  {
		...

    $data = array(
      'someVar' => 'Some value',
      'anotherVar' => 'Some other value'
    );

    $this->Pusher->broadcastMessage($this->configFactory,
                                    'my-channel-name-here',
                                    'my-event-name-here',
                                    $data);

		...
  }

	...
}
```
### Configuring the Client-Side Javascript


The module will create a global Javascript object called Pusher. You may use it to access the Pusher connection that is created automatically for you on page loads. Additionally, it will create the following global Javascript variables that can be used to access various Pusher channels:

+ *Pusher*
the global Pusher object

+ *PusherChannels*
array of public channels the user is subscribed to

+ *privateChannels*
array of private channels the user is subscribed to

+ *presenceChannels*
array of presence channels the user is subscribed to

Here is an example of creating your own channel and subscribing to an event:

```javascript
var myChannel;

if (Pusher)
{
  myChannel = Pusher.subscribe(‘your-channel-name-here');

  // Bind to the “your-event-name-here" event, so we can listen for it to come across
  Pusher.bind(‘your-event-name-here', function(data) {
	// Access your event information via the "data" object once the event is received by the client/browser
	console.log( data );
  });

}

```


If you have the need, you can also trigger/broadcast events straight from your app via Javascript as well:



```javascript
var triggered = someChannel.trigger('some-event-name', { your: data }

```
