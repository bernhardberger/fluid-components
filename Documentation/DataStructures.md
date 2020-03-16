# Fluid Components: Data Structures

One key aspect of components is the clearly defined interface between the integration
and the the component renderer via parameters. By default, Fluid
and Fluid Components support scalar data types, such as `string` or `integer`, and the
compound type `array` for arguments. But it is also possible to pass PHP objects to
ViewHelpers or components.

This allows us to define complex data structures which make the component interface
more formal but also more flexible during integration.

## Links and Typolink

`SMS\FluidComponents\Domain\Model\Link` (alias: `Link`)
`SMS\FluidComponents\Domain\Model\Typolink` (alias: `Typolink`)

TYPO3's backend link wizards generate a link definition in the so-called Typolink format:

```
t3://page?uid=123 _blank myCssClass "Title of the link"
```

It contains the link itself, but also allows you to specify a target, a link title, and additional css classes.

When a component takes a link as one of its parameters, it should on the one hand be able
to use a Typolink, but on the other hand it should be able to use a simple HTTP url. This
is where the Typolink datastructure as well as argument converters come into play:

Our sample component `Atom.Link`:

```xml
<fc:component>
    <fc:param name="label" type="string" />
    <fc:param name="link" type="Typolink" />

    <fc:renderer>
        <a href="{link.uri}" target="{link.target}" title="{link.title}" class="{link.class}">
            {label}
        </a>
    </fc:renderer>
</fc:component>
```

This component can be called in the following ways thanks to the implicit conversion
Fluid Components offers:

```xml
<my:atom.link
    label="Link to sitegeist website"
    link="https://sitegeist.de/"
/>

<my:atom.link
    label="Link to email"
    link="mailto:test@example.com"
/>

<my:atom.link
    label="Link to TYPO3 page with uid 123"
    link="123"
/>

<my:atom.link
    label="Link to TYPO3 page with alias 'myalias'"
    link="t3://page?alias=myalias"
/>

<my:atom.link
    label="Link to TYPO3 page with uid 456 in new window"
    link="456 _blank"
/>
<my:atom.link
    label="Link to TYPO3 page with uid 456 in new window"
    link="{uri: 456, target: '_blank'}"
/>

<my:atom.link
    label="Link to TYPO3 page with uid 789 with title and CSS class"
    link='789 - myCssClass "This is my link title"'
/>
<my:atom.link
    label="Link to TYPO3 page with uid 789 with title and CSS class"
    link="{uri: 789, class: 'myCssClass', title: 'This is my link title'}"
/>
```

In addition to the Typolink parsing, the Link data structure also gives access to the individual parts
of the URI by using `parse_url`. You get access to the following link properties in the component renderer:

* `originalLink`: TYPO3 link data structure, e. g. `['type' => 'page', 'pageuid' => 1]`
* `target`: e. g. `_blank`
* `class`: additional CSS classes that should be set on the `<a>` tag
* `title`: title attribute of the `<a>` tag
* `uri`: complete url, e. g. `https://myuser:mypassword@www.example.com:8080/my/path/file.jpg?myparam=1#myfragment`
* `scheme`: e. g. `https`
* `host`: e. g. `www.example.com`
* `port`: e. g. `8080`
* `user`: e. g. `myuser`
* `pass`: e. g. `mypassword`
* `path`: e. g. `/my/path/file.jpg`
* `query`: e. g. `myparam=1`
* `fragment`: e. g. `myfragment`

## Images

Components should be able to accept an image as a parameter, no matter where it come from. TYPO3 already has data structures
for images that are stored in the File Abstraction Layer. However, there are cases where you want to use
images from inside extensions or even external image urls. This is what the Image data structures offer:

* `SMS\FluidComponents\Domain\Model\Image` (alias: `Image`) is the base class of all image types as well as a factory
* `SMS\FluidComponents\Domain\Model\LocalImage` wraps a local image resource, e. g. from an extension
* `SMS\FluidComponents\Domain\Model\RemoteImage` wraps a remote image uri
* `SMS\FluidComponents\Domain\Model\FalImage` wraps existing FAL objects, such as `File` and `FileReference`
* `SMS\FluidComponents\Domain\Model\PlaceholderImage` generates a placeholder image via placeholder.com

This is how it could look like in the `Atom.Image` component:

```xml
<fc:component>
    <fc:param name="image" type="Image" />
    <fc:param name="width" type="integer" optional="1" />
    <fc:param name="height" type="integer" optional="1" />

    <fc:renderer>
        <f:switch expression="{image.type}">
            <f:case value="FalImage">
                <f:image
                    image="{image.file}"
                    alt="{image.alternative}"
                    title="{image.title}"
                    width="{width}"
                    height="{height}"
                />
            </f:case>
            <f:defaultCase>
                <img
                    src="{image.publicUrl}"
                    alt="{image.alternative}"
                    title="{image.title}"
                    width="{width}"
                    height="{height}"
                />
            </f:defaultCase>
        </f:switch>
    </fc:renderer>
</fc:component>
```

And these are the different options to call that component:

```xml
<!-- Use static images (local path) -->
<my:atom.image image="EXT:my_extension/Resources/Public/Images/logo.png" />
<my:atom.image image="{file: 'EXT:my_extension/Resources/Public/Images/logo.png'}" />
<my:atom.image image="{resource: {path: 'Images/logo.png', extensionName: 'myExtension'}}" />
<my:atom.image image="{resource: {path: 'Images/logo.png', extensionKey: 'my_extension'}}" />

<!-- Use static images (remote uri) -->
<my:atom.image image="https://www.example.com/my/image.jpg" />
<my:atom.image image="{file: 'https://www.example.com/my/image.jpg'}" />

<!-- Use existing FAL objects -->
<my:atom.image image="{fileObject}" />
<my:atom.image image="{fileObject: fileObject}" />
<my:atom.image image="{fileReferenceObject}" />
<my:atom.image image="{fileObject: fileReferenceObject}" />

<!-- Use FAL file uid -->
<my:atom.image image="123" />
<my:atom.image image="{fileUid: 123}" />

<!-- Use FAL file reference uid -->
<my:atom.image image="{fileReferenceUid: 123}" />

<!-- Use FAL file reference based on table, field, uid, counter -->
<my:atom.image image="{
    fileReference: {
        tableName: 'pages',
        fieldName: 'media',
        uid: 123,
        counter: 0
    }
}" />

<!-- Use a placeholder image with certain dimensions -->
<my:atom.image image="{width: 1000, height: 750}" />

<!-- Add title and alternative text to all array variants -->
<my:atom.image image="{
    fileReferenceUid: 456,
    title: 'My Image Title',
    alternative: 'My Alternative Text'
}" />
```

Inside the component renderer, you get access to the following image properties,
no matter which image variant is used:

* `type`: name of the image implementation, e. g. `PlaceholderImage`
* `publicUrl`: url that can be used in an `<img src...` attribute
* `alternative`: alt text for the image, to be used in `<img alt...`
* `title`: title for the image, to be used in `<img title...`

In addition, the different implementations offer additional properties that can
be used safely after checking the `type` property accordingly.

## Type Aliases

The included data structures can also be defined with their alias. These are `Image`, `Link` and `Typolink`.

```xml
<fc:component>
    <fc:param name="image" type="Image" />
</fc:component>
```

To register aliases for other classes extend the typeAliases array in your `ext_localconf.php` (e.g. Phonenumber)

```php
$GLOBALS['TYPO3_CONF_VARS']['EXTCONF']['fluid_components']['typeAliases']['Phonenumber'] = \VENDOR\MyExtension\Domain\Model\Phonenumber::class;
```