# Solution for auto-generating forms and their associated schema
Inspired from the core principles of [Netlify Cms](https://www.netlifycms.org/), this system was built and adapted to better fit this project specificity.

The following documentation will attempt to explain how forms and schema pages can be auto-generated by using a simple `.yml` file, alongside the form and schema layouts and the translation documents in the `_data/i18n/` folder. However, as of now, there are still some limits to the automation process, and some javascript is still required to make the forms work.

This document describes how each solution element works. Instructions on how to use, update and create solution elements are also presented.

## Pages
Found under the `_pages/` folder, pages must follow the template below so that all the data that you will set up next be pulled.

**WARNING** When creating or updating pages, make sure that changes are applied to both languages! Find the corresponding pages in `_pages/en` and `_pages/fr`. Look for pages that have the same `ref` value in their header.

In general, pages have the header variables presented below. These header variables must be included in any form of schema page (an example is provided at the end of the list).
 - `layout`: defines which layout structure to follow. Layouts are defined in the `_layouts/` folder. Its default value is simply "default".
 -  `ref`: defines the page's id, which is unique between different pages. However, a french page must have the same `ref` value as its corresponding english page and vice-versa, allowing the website to redirect to the proper page when switching between languages, as pages might not have the same name in both languages. For instance, an english index page and a french index page must both have the same `ref` variable value, and this value will be different for all other pages.

> How is `ref` used? When switching between languages, pages are generated using Jekyll/Liquid. It looks for all the pages that have the same `ref` value as the current page, but a different `lang` value, and adds their links to the language switcher navbar.

 -  `lang`: defines the associated language of the page. Used mostly as `[page.lang]` throughout the templates, it selects the correct value for a translation. The accepted values are either in `EN` or `FR` (case is important).
> Every translation follows this template:
> ```yaml
> example:
>    en: English value
>    fr: French value
>    ```
>    So to call the translation for the "example" keyword, we would use the following: ``{{ example[page.lang] }}``. Using ``[page.lang]`` simplifies content creation, as we don't have to worry about the current language for each individual page. Il also enables us to freely use includes without having to create a duplicate for each language, as page variables are applicable to all components included in said page.
 - `permalink`: The permalink for the page (at least the part that will be appended to the base URL of the website) overrides the file location in the folders when the website is compiled. Follow the convention `/[lang]/[page name].html`.

**Header example for a normal page**
```yaml
---
layout: default
ref: example
lang: en
permalink: /en/example.html
---
```

### Form page
Form pages require a few adjustments:
 - `layout`: form pages use "form" as their layout value instead of the default one.
 - `config`: refers to the name of the Yaml file in `_data/forms/` associated with the current form. That's how the relation is created between the current layout and the required data for the form.

**Header example for a form page**
```yaml
---
layout: form
ref: exampleForm
lang: en
permalink: /en/example-form.html
config: config-example  # Will fetch _data/forms/config-example.yml
---
```

### Schema page
Schema pages use the same parameters as form pages, but the `layout` variable must be set to "schema". In addition, the schema and form pages that are associated to one another must have the same `config` value. That way, both pages can pull from the same data.

**Header example for a schema page**
```yaml
---
layout: schema
ref: exampleSchema
lang: en
permalink: /en/example-schema.html
config: config-example
---
```

## Layouts
Forms and schema pages pull from the same two layouts that act as templates in which the data extracted from the config files will be added.

### Form layout
Found under `_layouts/form.html`, it inherits from the default layout as to include the header, footer and other fixed components of the website.

#### How it works
First, using `{{ page.content }}`, the content defined in the page using this layout is added. The aim is to include any page-specific content you wish to add. Then comes the form.

The part of the form included in the layout has classes and ids that are required for the form validation plugin (that comes with the [WET-BOEW](https://wet-boew.github.io/wet-boew/docs/ref/formvalid/formvalid-en.html) template). The `idPresets` variable is then defined. It refers to the first-level group "presets" in the translation file. It is declared only once for the forms in order to avoid duplication of definition. Up next comes the "pièce de résistance": an `include` call to the form loop, allowing the template system to look for the values in the config file and to include the right elements to the form.

The submitter file is then included. Contact information of the person submitting the form (name and email address) are required for the creation of the pull requests. Since we're currently using an anonymous bot to create all pull requests, these information allow us to keep track of who submitted the changes.

Finally, the submit and reset buttons are added to the form. Note that the submit button has a unique id corresponding to `prbotSubmit{{ page.ref }}` where `{{ page.ref }}` refers to the value you added to `ref` in the header of the page. This will come up later in the JavaScript files, when binding the submit event to the button. It was added to better distinguish between the different submit function calls and to prevent errors. Various alerts are then added, improving the user experience.

All these elements are located in the layout instead of the page or config file since they should all appear in the form page, regardless which form the page is referring to.

### Schema layout
Found under `_layouts/schema.html`, it inherits from the default layout as to include the header, footer and other fixed components of the website.

#### How it works
First, using `{{ page.content }}`, the content defined in the page using this layout is added. The aim is to include any page-specific content that you wish to add. The loop for the schema is then included. Nothing else is added to the schema pages.

## Config files
The config files are the backbone of the solution. They specify each component that should be included in the form and schema, as well as its parameters.

**Here's an example file with default values:**
 - `~` represents an undefined default value
 - Every value is a string, unless specified otherwise using comments
	 - `# enum()` indicates that the field has specific values and that any other value will result in an error
 - The list for every available preset and widget is specified later
	 - Certain presets and most widgets have their own specific parameters and are explained later in their corresponding section
```yaml
---

id: example

formGroups:
  - preset: ~  # enum()
  - widget: ~  # enum()
    title: ~
    type: text
    rule: ~  # enum()
    required: true  # bool

```

### Presets
Found under `_includes/[form/schema]/presets/`, presets are a set of (mostly) static sections that rarely take in parameters and should be used as is. Each preset was created for either of the following two purposes: 
-  when a section is called from two or more forms, as it made more sense to have it predefined instead of having to specify the same parameters of a widget twice
- when the section is too specific or doesn't fit in any of the widgets, either because it has some minor changes that would have been too specific to add as a parameter, or because the format is completely different from any widget

The following sections explain each of the available presets:
 - adminCode
 - contact
 - dates
 - description
 - empty
 - homepageURL
 - hr
 - languages
 - licences
 - newAdmin
 - orgLevel
 - provinceSelect
 - relatedCode
 - selectCode, selectSoftware, and selectStandard
 - status
 - submitter
 - tags

#### adminCode
The adminCode preset displays a `<select>` widget for administrations, which are separated into groups for each level of government (federal, provincial, municipal, aboriginal, etc.). You can check the list of administrations in `_data/administrations/[level of gov.].yml`.

The select widget is followed by a button, allowing the user to create a new administration in case his administration was not already in that list. The button opens a new section in the form, containing the required fields for the creation of a new administration. For more information on these fields, consult the **newAdmin** preset section.

When selecting the appropriate administration (depending on the type of the form), JavaScript should be added to auto-fill the administration section of the form. This follows the same logic as for the **selectCode**, **selectSoftware**, and **selectStandard** presets.

This preset takes no additional parameters.
```yaml
  - preset: adminCode
```

#### contact
The contact preset displays a group of fields for the contact information (URL, email address, name, phone) of the person or organization responsible of the open source element.

This preset takes one *optional* parameter:
 - `phone`: specifies if the phone field should be added or not. Its default value is `true`.
```yaml
  - preset: contact
    phone: true  # bool
```

#### dates
The date preset displays a group of fields for the project specific dates; when the project was created, when it started, when it was last modified, and when its data on ore-ero were last updated.

This preset takes three **required** parameters:
 - `created`: specifies if the created date input should be added
 - `started`: specifies if the started date input should be added
 - `modified`: specifies if the last modified date input should be added
```yaml
  - preset: dates
    created: false  # bool
    started: false  # bool
    modified: false  # bool
```
Note that these parameters are required. The date group will be left empty in case they were not defined, except for the date when the metadata was last updated: this parameter is included by default and its value is locked to today's date, as the form submitting process implies that data were updated.

Also, to simplify the code you write, you can simply omit any value you wish to hide, and specify to true the values that you wish to display.
```yaml
  - preset: dates
    created: true
    modified: true
```

#### description
The description preset displays two fields, one for the english and the other for the french description of the current project. This preset was created since at least two of the forms made use of a description field.

This preset takes no additional parameters.
```yaml
  - preset: description
```

#### empty
The empty preset does not display anything in the form, but rather serves only as an [array] wrapper in the schema when there is a list of elements that are only updated as individual elements in the form. For instance, for the "release" array of the code schema (`_data/schemaCode.yaml`), it makes no sense to change the layout of the form to adapt to an array since only one release can be updated at a time. However, this should still be displayed as an array in the schema page.

This preset takes two **required** parameters:
 - `start` defines whether it is the start or the end of the array element in the schema (decides to add either the beginning or the ending html markup).
- `title` defines the name of the array element in the schema (also used for the translations, see the widget section for more information about title parameters)
```yaml
  - preset: empty
    start: false  # bool
    title: example
# Beginning a group / array
```
OR
```yaml
  - preset: empty
# Closing a group / array
```
The beginning and the ending markup are located under `_includes/schema/components/wrap_start.html` and `_includes/schema/components/wrap_end.html`. These are the files that are included in the schema pages.

#### homepageUrl
The homepageUrl preset displays a duo of fields, one for the english and the other for the french homepage URL of the current project. This preset was created since at least two of the forms made use of a homepage URL.

This preset takes no additional parameters.
```yaml
  - preset: homepageUrl
```

#### hr
The hr preset is a simple `<hr>` tag, but was created to specify sections in which the fields could be auto-completed when using a select input. For instance, in the Open Source Software form, selecting an already existing project would fill its information, leaving the user to fill only the remaining part of selecting their administration and updating its uses.

This preset takes no additional parameters.
```yaml
  - preset: hr
```

This preset does not appear in the schema.

#### Languages
The languages preset displays a list of check boxes, allowing the user to select programming languages associated to the current project. It also allows the user to add other programming languages that are not listed. This preset was created since its html markup differed from the other components and widgets.

This preset takes no additional parameters.
```yaml
  - preset: languages
```

#### Licences
The licences preset displays two fields for the licence URLs (for english and french URLs) and a field for the spdxID of the licence. The label for the spdxID contains a link to a list and definition of spdxIDs.

This preset takes no additional parameters.
```yaml
  - preset: licences
```

#### newAdmin
The newAdmin preset has two roles. In the administration standalone form, it displays all the required fields to create a new administration. In any other form, it comes with the **adminCode** preset and appears only when the user clicks on the "add a new administration" button.

This preset contains five fields.
 - A select field for the administration's level of government(locked to municipal and aboriginal)
 - A field for the code of the new administration
 - A select field for the corresponding province
 - Two fields for both the english and french names of the new administration

This preset takes one *optional* parameter:
 - `optional`: Distinguish between the standalone version of the form, or the included hidden by default version that is added with the **adminCode** preset.
```yaml
---

id:  admin

formGroups:
  - preset:  newAdmin

# _data/forms/config-admin.yml
```
OR
```html
{%- include  form/presets/newAdmin.html  id=include.id  optional=true  -%}
<!-- _includes/form/presets/adminCode.html -->
```

#### orgLevel
The orgLevel preset displays a single `<select>` widget with the different level of government (federal, provincial, municipal, aboriginal) where the "federal" and "provincial" values are disabled, since there can't be any new provinces or federal administrations.

This preset takes no additional parameters.
```yaml
  - preset: orgLevel
```

#### provinceSelect
The provinceSelect preset displays a single `<select>` widget with the list of all provinces and territories of Canada.

This preset takes no additional parameters.
```yaml
  - preset: provinceSelect
```

#### relatedCode
The relatedCode preset displays four fields:
 - The URLs (english and french) of the related code
 - The Names (english and french) of the related code

This preset takes no additional parameters.
```yaml
  - preset: relatedCode
```

#### schemaVersion
The schemaVersion preset displays a read-only field for the schema version, currently at "1.0". It should be added to each form.

This preset takes no additional parameters.
```yaml
  - preset: schemaVersion
```

#### selectCode, selectSoftware and selectStandard
The selectCode, as well as selectSoftware and selectStandard presets display a `<select>` widget allowing the user to select an already existing project in order to edit it or add a new linked element (releases, uses, administrations, etc.). Using JavaScript, the whole point of these presets are to auto-fill the corresponding section of the form when selecting an existing project.

These do not show in the schema pages. Their only use is to allow an auto-fill feature.

This follows a similar principle as the **adminCode** preset.

These presets take no additional parameters.
```yaml
  - preset: selectCode
```

#### status
The status preset displays a `<select>` input with the possible project status (Alpha, Beta, Maintained, Deprecated or Retired). It was created since at least two or more form used it.

This preset takes no additional parameters.
```yaml
  - preset: status
```

#### submitter
The submitter preset displays a separated section of the form asking for the user contact information (name, email address), in order to keep track of who submitted the form.

The submitter preset shouldn't be added in the config files since it's already included in the form layout. See the section about the form layout for more information.

#### tags
The tags preset displays fields for adding tags to the current project. There are sections for both english and french tags, as well as a button to add more tags, other than the first required one.

This preset takes no additional parameters.
```yaml
  - preset: tags
```

### Creating a new preset
The steps to creating a new preset are presented below. But first, check whether this new preset is really necessary. As explained in the presets description, they should be created only on two occasions: when a certain form component appears in more than one form in order to reduce code duplication, or when a certain form component does not follow the simple markup of any other widget. However, for the new structure or markup that appears more than once, creating a more generic widget instead might also be beneficial, and hence, more reflection is necessary to make the best decision.

#### Add your new preset in the form components
Under `_includes/form/presets/`, create a file for your new preset. Its name should follow the camelCase convention. Fill it with html markup. Inspiration from other presets would help. Don't be afraid to simply include widgets in it using specific parameters, that's how most presets work when they are used in more than one form.

Then, add a call to your new preset in the form loop (`_includes/form/loop.html`) under `{%- case formGroup.preset -%}`. Follow an alphabetical order to simplify search and match the display in the folders. In the following example, change `[preset]` for the name of your new preset:
```html
{%- if formGroup.preset -%}
  {%- case formGroup.prese  -%}
    {%- when '[preset]' -%}
      {%- include form/presets/[preset].html id=idPreset [parameter=formGroup.value] -%}
      [...]
```
Keep the `id=idPreset` parameter and value intact since they will be used for the translations. Remove `[parameter=formGroup.]` if you don't need to specify other parameters (this is usually the common case), or replace it with your own custom parameter, where `value` has the same value as what you will add in the config file, and, for the sake of consistency, `parameter` should also match `value`. The `formGroup` parameter is simply the name given to each iterated element in the loop (`for formGroup in formGroups`). For example:
```html
{%- when 'example' -%}
  {%- include form/presets/example.html id=idPreset test=formGroup.test -%}
  [...]
```
AND
```yaml
  - preset: example
    test: myValue
```
The value for `test` can then be accessed in your preset as `include.test`.

> Include is the keyword used to access parameters declared in any file under the `_includes/` folder. The same is true for `page.test` in `_pages/`. Data is a bit different as it requires first a call to site `site.data.test` where "test" can be a file or a folder in `_data/`.

#### Add your new preset in the schema components
Under `_includes/schema/presets/`, create a file for your new preset. Its name should match the one you previously created in the form folder. To fill it, follow the same advice as for the form.

Then, add it to the schema loop (`_includes/schema/loop.html`) using the same indications as for the form.

Unless your preset has a different behaviour between the form and the schema (like the empty, hr or selectCode presets), you should always create the preset file in both the form and schema folders. If your preset doesn't have the same behaviour, add its case `{%- when  'example'  -%}` to both loops but keep the case content empty. That way, you will avoid throwing errors.

### Widgets
Found under `_includes/[form/schema]/widgets/`, widgets are generic components that take more parameters and thus can be configured to fit different sections of a form.

Example Widget with default values
```yaml
  - widget: example  # enum()
    title: ~
    type: text  # enum()
    rule: ~  # enum()
    required: true  # bool
```

 - `widget`: specifies which widget to include. This parameter is **required**, otherwise the form will display an error message.
 - `title`: the title of the widget. Must be a unique id (with exceptions, see the group widget). Also used for translations. This parameter is **required**, otherwise the widget won't display any text. Its value should be the same as the corresponding element in the schema since it's used as is in the schema page template (further information is presented in the translation section).
> The title parameters corresponds to the id used in the translation file (`_data/i18n/form.yml`), as a second-level element under the first-level element corresponding to the id of the form declared at the beginning of the config file.
> ```yaml
> example:  # The id of the form
>   title:  # The title of the widget
>       [...]
>   ```
>   More on translations can be found in the translations section.
 - `type`: defines the input type. Available values are the appropriate values for an html input tag (text, URL, email, etc.) included in a way similar as `<input type="{{ widget.type }}">`. This widget is *optional* and defaults to "text". Specify it in case you wish to have a different type.
 - `rule`: specifies a custom rule of validation. Available rules can be found in `_data/forms/rules.yml`. You can use any of the key (`key: value`) you may find in this document as a value for the rule parameter. This parameter is *optional* and defaults to none.
> The custom rules allows different types of validation which allows or forbids different characters in the fields. Check `assets/js/src/custom-form-validation.js` to see the full list of rules and their accepted characters.
- `required`: specifies if the field(s) in the widget is(are) required or not. This field is *optional* since all fields are required by default. Specify this parameter only when you wish to indicate that a widget should not be required.

The following sections explain each of the available widgets:
 - group
 - select
 - string-i18n
 - string

#### group
The group widget displays a list of other widgets under a single title.

This widget takes one additional parameter, in addition to the ones available to widgets.
 - `fields`: an array of widgets and their own parameters to display in the same logical group.
```yaml
  - widget: group
    title: example
    fields:
      - widget: string
        title: test1
      - widget: string-i18n
        title: test2
      [...]
```
The widget declaration under fields acts the same way as if they were declared as top-level fieldGroups.

Also, if you add the required parameter to the group widget, it will apply automatically to all of its children, unless specified otherwise at the child's level. The following example shows how the required parameter can be used:
```yaml
  - widget: group
    title: example
    required: false
    fields:
      - widget: string
        title: test1
        required: true
      - widget: string-i18n
        title: test2
      [...]
```
In this example, the `required: false` of the parent `group` overwrites the default required value of all its widgets children, so in this example, the `string-i18n` widget would not be required. However, it would still be required in the case of the `string` widget since we specified it directly. The following example shows another way on how the required parameter can be used:
```yaml
  - widget: group
    title: example
    fields:
      - widget: string
        title: test1
        required: false
      - widget: string-i18n
        title: test2
      [...]
```
In this example, since the required parameter is not specified on the `group` level, all the children are required by default. However, specifying the required parameter on a child has the same behaviour as usual, meaning that, in this case, the `string` child widget, and only this one, would not be required.

It is not possible to add presets to group widgets.

#### select
The select widget was created in order to have a more generic version of a `<select>` tag. However, all the widgets that were using it were later transformed to presets as we discovered they were used in more than one form. Currently, the select widget is not used, but since it's already built, I decided to keep it in.

This widget takes one additional parameter in addition to the ones available to widgets.
 - `options`: a list of the options available as an array.
```yaml
  - widget: select
    title: example
    options: [a, b, c]
```
The options values in the array would be also used in the value attribute of the option tag (`<option value="{{ option[i] }}"`). These options are also the keys used for the translations. For further information, check the translation section.

#### string-i18n
The string-i18n widget displays two fields for a single value (in english and french).

This widget takes all the parameters available to widgets.
```yaml
  - widget: string-i18n
    title: example
```
This widget takes also an additional parameter that was added to fix a duplicate id error in the html markup.
 - `prepend`: A string to prepend to the id when generating the html markup. It allows to keep the title parameter clear (since it's used in the schema as well as for translations) without creating conflicts in ids. It feels like a cheat, and thus should be used wisely.

#### string
The string widget displays a single field for values that are unique and don't need translating.

This widget takes all the parameters available to widgets.
```yaml
  - widget: string
    title: example
```

### Creating a new Widget
Creating a widget is similar to creating a preset, but follows a more generic idea. Creating a new widget when multiple form components follows the same basic and simple structure. A widget shouldn't be complex and should allow for more variations than the presets.

#### Add your new widget in the form components
Under `_includes/form/widgets/`, create a file for your new widget. Its name should follow camelCase convention. Fill it with html markup, use inspiration from other widgets, and, if possible, include components in it instead of adding new code.

Then, add a call to your new widget in the form loop (`_includes/form/loop.html`) under `{%- case formGroup.widget -%}`. Follow an alphabetical order to simplify the search process and match the display in the folders. In the following example, change `[example]` for the name of your new widget:
```html
{%- else -%}
  {%- case formGroup.widget -%}
    {%- when '[example]' -%}
      {%- include form/widgets/[example].html id=id title=formGroup.title type=formGroup.type rule=formGroup.rule required=formGroup.required [parameter=formGroup.value] -%}
      [...]
```
*Don't forget to add the default widget parameters.*

Remove `[parameter=formGroup.value]` if you don't need to specify other parameters, or replace it with your own custom parameters, where `value` has the same value as what you will add in the config file, and, for the sake of consistency, `parameter` should also match `value`.  The `formGroup` is simply the name given to each iterated element in the loop (`for formGroup in formGroups`). For example:
```html
{%- when  'example'  -%}
  {%- include form/presets/example.html [default parameters] test=formGroup.test -%}
```
AND
```yaml
  - widget: example
    test: myValue
```
The value of `test` can then be accessed in your preset as `include.test`.

> Include is the keyword used to access parameters declared in any files under the `_includes/` folder. The same is true for `page.test` in `_pages/`. Data is a bit different as it requires first a call to site `site.data.test` where "test" can be a file or a folder in `_data/`.

#### Add your new widget in the schema components
Under `_includes/schema/widgets/`, create a file for your new widget. Its name should match the one you previously created in the form folder. To fill it, follow the same advice as for the form.

Then, add it to the schema loop (`_includes/schema/loop.html`) using the same indications as for the form.

Unless your widget has a different behaviour between the form and the schema (like the empty, hr or selectCode widgets), you should always create the widget file in both the form and schema folders. If your widget doesn't have the same behaviour, add its case `{%- when 'example' -%}` to both loops, but keep the case content empty. That way, you  will avoid throwing errors.

## Includes

### The loop
Both loops act the same way. Basically, for each element in the `formGroups` array in the config file, the loop checks if the current element is a preset or a widget and a switch case checks the type of each preset or widget that is found. The loops includes then the correct file and passes the file's parameters either from the config file or directly in the include call.

The for loop goes like this: `{%- for formGroup in site.data.forms[page.config].formGroups -%}`. The "in" part is a liquid call that includes the content of the include file into the file where the include call is made: `site.data` is equivalent to requiring the `_data/` folder; forms is a call to the `forms` folder under data, linking to `_data/forms/`; and `[page.config]` is a call to the page parameter added to the page header.
```yaml
---
[...]
config: config-example
```
So `[page.config]` translates to, in this case, `config-example`, which will concatenate to `_data/forms/config-example.yml`. Not only you can mix folders and files at the same time, but also mix in parameters from inside files through `formGroups`: the `formGroup` parameter in `_data/forms/config-example.yml`.

### Components
Components are even more generic than widgets. They were created solely for the purpose of less duplication of code. But in order to make it simple, it became a bit complicated underneath, and if you don't need to debug, you can skip this section and go directly to translations.

TODO

## Translations
Translations for the forms and schema are located in `_data/i18n/form.yml`. They are separated in sections for more generic translations as well as first-level groups for each of the forms.

#### Yaml Variables
Note that it is possible to create variables in Yaml files. Use `&variable` for instantiation and `*variable` for using. In this particular case, it's used to declare and use generic values (name, email, URL, etc.). But the neat thing is that it can act as a string or as an object, so we can easily wrap translations inside a single variable.

Here's an example of what it looks like in the files:
```yaml
name: &name
  en: Name
  fr: Nom
[...]
example:
  label: *name
```
It is equivalent to:
```yaml
example:
  label:
    en: Name
    fr: Nom
```
But without translation duplication.

### Presets
All the presets translations are already included. When modifying  any of the presets files (`_includes/[form/schema]/presets/`), you should also check the translations.

When creating a new preset, it should be added under the presets first-level group, unless you specified a different parameter in the loop (`includes/[form/schema]/loop.html`).
 - In general, the preset includes specifying "presets" as an id: `{% include file.yml id=idPreset %}`
 - However, you can use `id` instead of `idPreset` to use the id defined in the config page. This means that you should put the translations under the equivalent first-level element named after the `id` instead of "preset".

### Translations for Widgets
Here's how to translate widgets:
 - For each widget, add a second-level element (under the first-level named after `id`). Its name should be the same as the value you put under `title` in the config file. The title should be the same as the value in the schema page since it's included as is in the schema page.
```yaml
first-level:  # either preset, admin, code, software, standard, etc...
  example:  # replace example with the title value of the widget
    [...]
```
 - You will also need to, bear with me it might be confusing, add a title element under it. Don't mistaken it for the second-level element, where title should be replaced with the title value of the current widget. This title is a third-level element named title which will hold the english and french value for the title. This value appears in the form page as a section's title and as the section description in the schema page:
```yaml
first-level:
  example:
    title:
      en: Example
      fr: Exemple
```
 - Each widget has also other specific elements that need to be added. The following sections describe each of theses elements.

#### String Widget
For a string widget, follow this template:
```yaml
first-level:
  example:
    title:
      en: Example
      fr: Exemple
    label:
      en: The english label
      fr: The french label
```

#### String-i18n Widget
For a string-i18n widget, follow this template:
```yaml
first-level:
  example:
    title:
      en: Example
      fr: Exemple
    labels:
      en:
        en: The english label for the english page
        fr: The french label for the english page
      fr:
        en: The english label for the french page
        fr: The french label for the french page
      schema:
        en: The generic english label for the schema (without specificity for either english of french)
        fr: The generic french label for the schema (without specificity for either english of french)
```

#### Select
For a select widget, follow this template:
```yaml
first-level:
    example:
      title:
        en: Example
        fr: Exemple
      options:
        option-name:  # where option-name is the value added in the options array in the config file
          en: Option
          fr: Option
        [...]
```
#### Group
For a group widget, the values under `labels` depends on the fields' widget type. Follow this template:
```yaml
first-level:
  title:
    en: Example
    fr: Exemple
  labels:
    string:  # replace string with the title of a string widget
      en: The english value for the label
      fr: The french value for the label
    string-i18n:  # replace string-i18n with the title of a string-i18n widget
      en:
        en: The english value for the english page
        fr: Then french value for the english page
      fr:
        en: The english value for the french page
        fr: The french value for the french page
  titles:
    widget-name:  # replace widget-name with each widget names
      # The title is used for the schema page
      en: Widget Name
      fr: Nom du widget
    [...]
```

## JavaScript
Each form needs its own custom JavaScript which you will have to create.

All the scripts are located under `assets/js/src/`. The scripts for each form follows the naming convention `[form id]Form.js` with camelCase. There are other scripts that fall under the utility category: `programmingLanguages.js` for adding and removing custom programming languages in the preset of the same name, and `tags.js` for adding and removing tags in the tags preset.

### Footer links
Links to the scripts are added in the footer located in `_includes/footer.html`.

At the very bottom, there's an inline script declaring some constants that are used in other script files: `REPO_NAME`, `USERNAME`, and `PRBOT_URL` . Their values are taken from the `_config.yml` and pulled through Jekyll/Liquid. The scripts for the forms are added right above that inline script. Up next, we add the script only on its particular form page with a simple `if` using the page `ref` (declared in the page header).

### PRB0T
[PRB0T](https://github.com/PRB0t/PRB0t) is an open source solution we [forked](https://github.com/j-rewerts/PRB0t) and customized to fit our specific needs. It was used with the forms in order to submit changes. Basically, we merge the necessary files with the new content pulled from the form, and then we submit a pull request in the Github repo.

### Javascript files
At the beginning of the file, there's a comment that tells ESLint that the function does exist but in another file. Hence, it won't throw errors or fail the build.

Two javascript objects are then declared constants. Since they're used all through the file, it made sense to declare them only once at the top.

There's also the jQuery, the equivalent of `document.ready`, in which we added the function binding between different selects for auto-fill, the submit and reset for the form.

#### Submit
When submitting the form, we first validate the form and initiate the alerts. We continue only once it's validated using the `submitInit()` function from the `validation.js`. Then, if there is a new admin button, we check if the admin form is filled and then call either the submit function with or without the new administration.

#### Reset
Some forms require a specific function for reset, especially when dealing with tags and programming languages.  The user is able to add and remove tag fields and we want to make sure to reset those fields since the basic reset does not take care of that by default.

#### getObject
The `get[form id]Object()` function takes all the values from the form inputs and assigns them to an object variable. That object follows the hierarchy as the schema for the form. It is separated into two sections. First, all mandatory fields are added to the object. Then, the function probes each single non-mandatory field, checking whether it has a value or not. In case it has a value, the non-mandatory field is added to the object. That way, no keys with empty values will be added to the data files.

#### addValuesToFields / resetFields
When the user selects an already existing project or administration, data is pulled from github. The addValuesToFields fills some input fields with the fetched data. Now if the user chooses to change back the select to its default null value, the resetField removes the values that were added automatically to the input fields from fetched data. 
In both cases, not all form fields are modified automatically: in the open software form, selecting an existing software and selecting an existing administration do not auto-fill the same fields, and resetting one select does not mean that the other select also needs resetting.
