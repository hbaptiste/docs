# Backbee Event Component
*Events* are a common and effective way to tie together loosely coupled components in an application.
An event is generally used to broadcast a change that has occured during a process. If a `component` is
interested in a particular event, he can *listen* to it. When this particular event is triggered, a `method`,
often called *callback* or *handler* is executed. Events are convenient ways to share
data between components.
In `BackBee`, events are triggered by an *EventDispatcher* and an *EventListener* is used to listen to them.

## Event

`Namespace: BackBee\Event\Event`

The Backbee event class extend `sfEvent` and allows us to create a generic `Event` object.
An generic event object has two properties: `target` and `args`;
The target property represents the target of the event. The `args` is an optional associated map.
When the `args` property is provided, it's sent to the listeners.
Custom Event can be created by extending the class `BackBee\Event\Event`.

```
$myHelloEvent = new Event($target, array("message"=>"Hello BackBee");

```

## Event Listener
`Namespace: BackBee\Event\EventListener`

To listen to an event we must use the `addListener` method of the `EventDispatcher`.
`addListener` takes three parameters. The first one is the name of the event that will be triggered, the second is an array whose first element
is a class path and the second is a method. The last parameter is the priority of the event. Listener with higher priority will be triggered first.

    $this->application->getEventDispatcher()->addListener("loading.event", array("BackBee/Event/Listener/HelloListener", "sayHello"));

Here we register the listener *HelloListener* to be executed when the event `loading.event` is occured.
	<?php

		namespace BackBee\Event\Listener;
		use BackBee\Event\Event;

		class HelloListener {

			public static sayHello (Event $event) {

				$dispatcher = $event->getDispatcher();

				if ($event->hasArguments("message")) {
					$message = $event->get("message");
					if ( null !== $dispatcher->getApplication()) {
						$dispatcher->getApplication()->getLogging()->notice($message);
					}
				}
			}
		}

	?>

Notice that the event object has access to the `application` *via* the `EventDispacher` and that the `sayHello` is a *static* method.

## Event Dispatcher
`Namespace: BackBee\Event\EventListener`

Events are *dispatched* or *triggered* by the `EventDispatcher` by using the method `triggerEvent`
The sample bellow shows how to listen to an event.

To trigger an event, we must use the `EventDispatcher`. It provides, among others, a `dispatch` method that takes two arguments. A name and an event object.

    $this->application->getEventDispatcher()->dispatch("loading.event", $myHelloEvent);

When the *loading.event* is dispatched by the `EventDispatcher`, all the registered listeners will be executed. In our example,
the static `sayHello` method of the `Backbee/Event/Listener/HelloListener` will be called.

















