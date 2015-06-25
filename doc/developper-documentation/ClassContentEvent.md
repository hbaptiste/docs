# ClassContent and events

One of the main purpose of `Backbee` is to render contents. As we have seen, a
content is defined in a `.yml` file. Content creation and content rendering go
through many phases that `Backbee` exposes with events. By instance, an event is triggered
when a content is created, before and after a content has been saved, when the rendering process
is started, while the content is been rendered, after the renderer process and so on. these events
allow us the alter in different ways the contents lifecycle or the rendering process itself.

ClassContent events are similar to custom events. But, as they take place when the main application
is dealing with a request, a common way to listen to them is to use an `event.yml` config file.
Will try to local event file in the `repository/Config` folder. Bellow is an example of how to
use the file.

```
    article.article.render:
		listeners:
			- [BackBee\Event\Listener\ArticleListener, onRender]

	social.facebook.prerender:
		listeners:
			- [BackBee\Event\Listener\Socialistener, PrerenderFacebook]
```

`events.yml` is a declarative way to add listeners to `ClassContenteEvents` event. Under the hook,
this yaml file is parsed and the `EventDispatcher` is used to register the listeners.
In our example,  `article.article.render` and `social.facebook.prerender` are events name.
By convention `BackBee` transforms ClassContent class path to event name. Let's take a look at our events files.

According to our `event.yml`, the `article.article.render` event will be triggered when
the content `ClassContent/Article/Article.yml` is about the be rendered whereas
the 'social.facebook.prerender' will be triggered *before* the content `ClassContent/Social/Facebook.yml`
was rendered.
`*.render`  and `*.prerender` are events triggered by the `Renderer` object. Bellow is the list of
all the Renderer events.

- *.prerender
- *.render
- *.postrender

*prerender* is triggered *before* the content is been rendered. *render* when the
 content is about te be rendered and *postrender* after the render process. How ever There is no strong difference *render* and *postrender*.

As in BackBee all the contents are `Doctrine` entities, these doctrine events are also available for all the contents.

- preremove
- postremove
- postupdate
- preupdate
- update
- prepersisttheser
- postpersist
- postload
- onflush


## ClassContent Listener
To listen to a ClassContentEvent we have to create a `Listener` class. By default Backbee
will looks for Listerner in the repository/Config/Listener folder.

```
    <?php
		namespace BackBee\Event\Listener,
		use BackBee\Event\Event;

		class ArticleListener extends Event {

			public static function onRender(Event $event){
				/* The renderer object */
				$renderer = $event->getEventArgs();

                /* The eventDispatcher */
				$eventDistacher = $event->getDispatcher();

                /* the backbee application */
				$application = $eventDistacher->getApplication();

				/* The article classcontent */
				$content = $renderer->getObject();

				/*Add a new parameter that will be available in the content template*/
				$renderer->assign('myParams', "my Param value");
			}
		}
	?>
```

## Event inheritance
The last feature we will cover is ClassContent event inheritance. In `BackBee`
content can be inherited from one another. The `EventDispatcher` respects this inheritance.
If a event is triggered for a subContent it will also be triggered for its parent. So *class content* Events bubble up in BackBee.