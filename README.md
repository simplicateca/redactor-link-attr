# redactor-link-attr

This redactor plugin modifies the link modal to include a configurable set of custom attributes (i.e. data-* and aria-*) that you can assign to inline text links.

![redactor link attriubte plugin screenshot](https://simplicate.nyc3.digitaloceanspaces.com/simplicate/assets/site/images/redactor-link-attr.png)

This is an early draft of the plugin and any bug reports would be appreciated.

## Installation

If you're using Craft CMS, copy the file `src/linkattr.js` from this repository into your `cms/config/redactor/plugins` directory.


## Configuration

You can configure the available attribute options within your redactor JSON config file:

*Sample Redactor-Config.json*

```
  ...
  "plugins": ["linkattr"],
  "linkAttrs" : [
    { "label": "Accessibility Label", "attr": "aria-label", "type": "text" },
    { "label": "Tracking ID", "attr": "data-tracking", "type": "text" },
    { "label": "Open in Modal?", "attr": "data-modal", "type": "boolean" },
    { "label": "Class", "attr": "class", "type": "enum", "options": [
      { "name": "Regular Button", "value": "button" },
      { "name": "Large Button", "value": "button large" }
    ] }
  ],
  ...
```

## Craft CMS and HTMLPurifier

If you're using Craft CMS, it's likely that the contents of your Redactor fields will be run through HTMLPurifier. The default HTMLPurifier configuration for Craft CMS does not allow for data-* or aria-* attributes within links.

In order to change that you'll have to include an event listener that modifies the HTMLPurifier config within a custom site module. 

If you don't already have a custom module for your site, you should definitely checkout Andrew's Welch's excellent blog article ["Enhancing a Craft CMS 3 Website with a Custom Module"](https://nystudio107.com/blog/enhancing-a-craft-cms-3-website-with-a-custom-module) for detailed instructions on how / why you should have one.

Once you have a custom module created, this is an example of the code you'll have to add to it to teach the HTMLPurifier config to allow the attributes you specify in your config. 

```
// ...

use craft\redactor\Field AS RedactorField;

// ...

class YourCustomModule extends Module
{
    // ...

    public function init()
    {
        parent::init();

        // ...

        // CUSTOM HTMLPURIFIER RULES
        Event::on(
            RedactorField::class,
            RedactorField::EVENT_MODIFY_PURIFIER_CONFIG,
            function (Event $event) {
                if( $event->config ) {
                    if( $def = $event->config->getDefinition('HTML', true) ) {
                        // http://htmlpurifier.org/docs/enduser-customize.html#addAttribute
                        $def->addAttribute('a', 'aria-whatever', 'Text');
                        $def->addAttribute('a', 'data-whatever', 'Text');
                    }
                }
            }
        );
    }

    // ...
}
```

The above example allows the attributes `aria-whatever` and `data-whatever` on <a> elements.
