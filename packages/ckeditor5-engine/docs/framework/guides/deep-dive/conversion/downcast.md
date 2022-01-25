---
category: framework-deep-dive-conversion
menu-title: Model to view - downcast
order: 20
since: 33.0.0
---

# Model to view - downcast

## Introduction

The process of converting the **model** to the **view** is called a **downcast**.

{@img assets/img/downcast-basic.png 760 Basic downcast conversion diagram.}

The downcast process happens every time a model node or attribute needs to be converted into a view node or attribute.

The editor engine runs the conversion process and uses converters registered by plugins.

{@snippet framework/mini-inspector}

## Registering a converter

In order to tell the engine how to convert a specific model element into a view element, you need to register a **downcast converter** by using the `editor.conversion.for( 'downcast' )` method:

```js
editor.conversion
	.for( 'downcast' )
	.elementToElement( {
		model: 'paragraph',
		view: 'p'
	} );
```

The above converter will handle the conversion of every `<paragraph>` model element to a `<p>` view element.

{@snippet framework/mini-inspector-paragraph}

<info-box>
	This is just an example. Paragraph support is provided by the {@link api/paragraph paragraph plugin} so you don't have to write your own `<paragraph>` element to `<p>` element conversion.
</info-box>

<info-box>
	You just learned about the `elementToElement()` **downcast** conversion helper method! More helpers are documented in the following chapters.
</info-box>

## Downcast pipelines

CKEditor 5 engine uses two different views: **data view** and **editing view**.

**The data view** is used when generating editor output. This process is controlled by the data pipeline.

**The editing view**, on the other hand, is what you see when you open the editor, which is controlled by the editing pipeline.

{@img assets/img/downcast-pipelines.png 760 Downcast conversion pipelines diagram.}

The previous code example registers a converter for both pipelines at once. It means that `<paragraph>` model element will be converted to a `<p>` view element in both **data view** and **editing view**.

Sometimes you may want to alter converter logic for a specific pipeline. For example, in editing view you want to add some additional class to the view element.

```js
// dataDowncast for data pipeline
editor.conversion
	.for( 'dataDowncast' )
	.elementToElement( {
		model: 'paragraph',
		view: 'p'
	} );

// editingDowncast for editing pipeline
editor.conversion
	.for( 'editingDowncast' )
	.elementToElement( {
		model: 'paragraph',
		view: {
			name: 'p',
			classes: 'paragraph-in-editing-view'
		}
	} );
```

{@snippet framework/mini-inspector-paragraph}

## Converting text attributes

As you may know from the chapter about the model, an **attribute** can be applied to a model text node.

Such text nodes attributes can be converted into view elements.

In order to do so, you can register a converter by using attributeToElement() conversion helper:

```js
editor.conversion
	.for( 'downcast' )
	.attributeToElement( {
		model: 'bold',
		view: 'strong'
	} );
```

The above converter will handle the conversion of every `bold` model text node attribute to a `<strong>` view element.

{@snippet framework/mini-inspector-bold}

<info-box>
	This is just an example. Bold support is provided by the {@link features/basic-styles basic styles} plugin so you don't have to write your own bold attribute to strong element conversion.
</info-box>

## Converting element to element

Similar to the basic example, you can convert `<heading>` model element into `<h1>` view element with `elementToElement()` conversion helper.

```js
editor.conversion
	.for( 'downcast' )
	.elementToElement( {
		model: 'heading',
		view: 'h1'
	} );
```

Which is equivalent to:

```js
editor.conversion
	.for( 'downcast' )
	.elementToElement( {
		model: 'heading',
		view: ( modelElement, { writer } ) => {
			return writer.createContainerElement(
				'h1'
			);
		}
	} );
```

You learned that the `view` property can be a simple string or an object. The example above shows it is also possible to define a custom callback function to return created element instead.

{@snippet framework/mini-inspector-heading}

The `<heading>` element makes the most sense if you can set the heading level.

From the previous chapter you learned that you can apply attributes to text nodes. It is also possible to add attributes to elements.

```js
editor.conversion
	.for( 'downcast' )
	.elementToElement( {
		model: {
			name: 'heading',
			attributes: [ 'level' ]
		},
		view: ( modelElement, { writer } ) => {
			return writer.createContainerElement(
				'h' + modelElement.getAttribute( 'level' )
			);
		}
	} );
```

From now on, every time level attribute updates, the whole `<heading>` element will be converted to the `<h[level]>` element (for example `<h1>`, `<h2>`, etc).

You can check this in action by using the example below:

{@snippet framework/mini-inspector-heading-interactive}

<info-box>
	This is just an example. Heading support is provided by the heading feature {@link features/headings headings feature} so you don't have to write your own `<heading level="1">` to `<h1>` element conversion.
</info-box>

## Converting element to structure

Sometimes you may want to convert a **single model** element into a more complex view structure consisting of a **single view element with children**.

You can use `elementToStructure()` conversion helper for this purpose:

```js
editor.conversion
	.for( 'downcast' )
	.elementToStructure( {
		model: 'myElement',
		view: ( modelElement, { writer } ) => {
			return writer.createContainerElement( 'div', { class: 'wrapper' }, [
				writer.createContainerElement( 'p' )
			] );
		}
	} );
```

{@snippet framework/mini-inspector-structure}

<info-box>
	Using your own custom model element requires defining it in the schema. More information in the example below.
</info-box>

## Read next

{@link framework/guides/deep-dive/conversion/upcast View to model - upcast}
