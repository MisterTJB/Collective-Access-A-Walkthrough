![CA Logo](http://www.collectiveaccess.org/sites/all/themes/collectiveaccess/img/body_logo.png)

# Collective Access: A Walkthrough

## Document Outline

[Collective Access](http://www.collectiveaccess.org) is an extensible collection management system that has been designed to capture metadata about arbitrary objects, their relationships and aggregation, and their administration.

This document has been prepared to initiate software developers and systems administrators who may be unfamiliar with Collective Access, the collection management domain, or both.

The [Background](#background) section offers a dissection of the Chicago Film Archive’s website – which presents data and functionality provided by Collective Access – as a means of exposing the utility of Collective Access via an exploration of a canonical use case; moreover, this section is helpful for intellectually connecting user-facing data abstractions to their back-end implementation.

The [Installation](#installation) and [Data Import](#data_import) sections acts as a walkthrough for installing a Collective Access instance (either on a local machine or a remote server) and populating it with data such that the usage and administration of the system is made clear.

Lastly, the [Modification](#modification) section provides examples of adding fields and bundles; redefining or renaming fields; and deleting unnecessary fields.


## <a name="background"></a>Background

Fundamentally, Collective Access is an open source, platform-agnostic content management system that focuses on providing collection management functions to support the activities of organisations that act as guardians for artefacts having some long-term (usually cultural or intellectual) value.

Where a content management system might be almost-solely concerned with storing and rendering multimedia, a collection management system typically augments this with additional metadata that captures the provenance, conservation, rights management, and other information pertaining to the long-term preservation of an object.

Collective Access differs from most commercially-available collection management systems in that it provides an interface through which organisations are able to tailor the suite of fields, functions, and interfaces provided by the system. Although the source code for Collective Access may be forked and modified, the system has been designed in such a way as to encourage modifications to be made via the web-based administrator interface.

### Example – The Chicago Film Archive

The extensibility of Collective Access makes it somewhat-difficult to inscribe the core functionality of the system; likewise, it is also difficult to determine its absolute limitations. In theory, given enough time, money, and effort, the system is capable of doing 'anything' – certainly, there are several examples of use cases that have been shoe-horned in to the Collective Access framework – though it is perhaps more productive to identify canonical usage of the system, and common abuses, and try to mimic the former without  succumbing to the latter.

Although the ‘developer documentation’ might seem sensible, it will be much more instructive to survey the variety of Collective Access variants that have already been elegantly deployed by [other institutions](http://www.collectiveaccess.org/projects).

A canonical example of Collective Access being used in production is that of the Chicago Film Archive (CFA). Embedded within the CFA website is a collections browser whose functionality is predominantly provided by [Pawtucket](https://github.com/collectiveaccess/pawtucket) – the Collective Access presentation module that is responsible for providing access to data that is stored in the cataloguing module [Providence](https://github.com/collectiveaccess/providence).

In [browsing the CFA’s collections](http://www.chicagofilmarchives.org/collections/index.php/Browse/clearCriteria), some of the fundamental functions and limitations of Collective Access are revealed.

#### Basic Tables

Firstly, note the existence of several so-called object types<sup>[1](#footnote_types)</sup>. The most important of these (note that there are [several others](http://docs.collectiveaccess.org/wiki/Basic_Tables)) include: `entities` (referred to here as People and Organisations); `places`; `collections` (i.e. objects that are related by way of a shared provenance, or a common intellectual theme); and `objects` (in this scenario, objects are specialised to suit the domain, and have been referred to as Videos, Films, Audio, and Manuscripts and Ephemera).

Given that these types inform the structure of the database (they form the so-called [Basic Tables](http://docs.collectiveaccess.org/wiki/Basic_Tables)) that underlies Collective Access, as well as the associated models and controllers responsible for performing create, read, update, and delete operations, it is reasonable to consider that any data that cannot be captured by these types will probably require significant intervention.

##### Fields, Relationships, and Representations

Now consider the records for a [specific video](http://www.chicagofilmarchives.org/collections/index.php/Detail/Object/Show/object_id/9219), and a [specific film](http://www.chicagofilmarchives.org/collections/index.php/Detail/Object/Show/object_id/14329); there are at least three observations to be made:

Firstly, note that the set of fields used to describe the two kinds of objects are different (though they overlap). In particular, the Format field is irrelevant to a video (i.e. for the CFA, video is video – there are no video subtypes), though it is relevant for film (i.e. from the perspective of the CFA, 16mm film is consequentially distinct from 35mm film). 

This distinction hints at the extensibility of Collective Access: in this case, fields have added or removed to suit a specific object type (even though videos and films are both a `ca_object`.

Secondly, notice the hyperlinked fields at the top and bottom of the record (e.g. the Part Of links, and the links under Form, Subjects, Related Places, and Genre). It should be obvious that the relationships between records are essentially encoded as relationships between Basic Tables – that is, the video and the film are each a `ca_object` (i.e. in the parlance of the database layer) that are connected to records in each of the `ca_lists` (Form, Subjects, Genre), `ca_collections` (Part Of), and `ca_places` (Related Places) Basic Tables.

Lastly, observe that both records embed a digitised video of the original object. Collective Access refers to associated media files as _representations_.

#### Summary

This section introduced the high-level functionality of Collective Access with reference to low-level concepts. In particular, the notion of Basic Tables is an extremely important reference point for determining how data is stored internally; moreover, the predefined Basic Tables strongly hint at the extent of the set of entity types that the system is designed to manage.

Additionally, the object, entity, collection, and representation types are relevant to the dataset that will be used in the remainder of this walkthrough – the CFA website portrays the canonical usage for these types.

## <a name="installation"></a> Installation

For the sake of this walkthrough, instructions will be given for an [Ansible provisioning](https://github.com/netsensei/ansible-collectiveaccess) of Collective Access on a local machine (note, though, that these steps are very similar for a [remote deployment](https://github.com/andre-geldenhuis/ansible-collectiveaccess/tree/remote_server)).

First, ensure that [Vagrant](https://www.vagrantup.com/downloads.html) and [Ansible](http://docs.ansible.com/ansible/intro_installation.html) are installed on your local machine. If these dependencies are properly installed, installing Collective Access is [very simple](https://github.com/netsensei/ansible-collectiveaccess#install).

The provisioning script installs Collective Access, and configures the database, PHP, and server. The important variables are summarised in the following table:

| Variable          | Value         |
|-------------------|---------------|
| Database Name     | ca            |
| Database User     | collective    |
| Database Password | access        |
| IP                | 192.168.2.145 |

If the installation has completed successfully, you should be able to access the Collective Access installer UI by navigating to 192.168.2.145 in a web browser<sup>[2](#footnote_php)</sup>.

![The installer interface](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/installer_screen.png "The installer interface")

At this point, it is worth noting that there are two aspects to the install process: in the first step, which was carried out automatically by the provisioning script, the necessary backend technologies are configured and the relevant variables are written in to a file at `path/to/collectiveaccess/setup.php`

![The setup.php file](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/setup_php.png "The setup.php file")

The second step, which we will attend to next, configures the structure of the Collective Access database and its UI to suit a specific use case. This configuration is informed by an [installation profile](http://docs.collectiveaccess.org/wiki/Installation_profile) – an XML document conforming to a [domain-specific language](http://docs.collectiveaccess.org/wiki/Installation_Profile_Syntax) that expresses the structure and semantics of all of the fields, lists, user interfaces, etc. in a Collective Access instance.

A common technique for establishing a Collective Access instance for a specific use case is to select the installation profile that is most-similar to the intended use case, and then modify that installation profile. For the purpose of this walkthrough, the Visual Resources Collection profile is most relevant.

To proceed, enter any conformant email address in the email address box (x@y.com isn't a bad choice) and choose Visual Resources Collection from the dropdown box – the UI may or may not provide a progress indicator, but installation usually takes about three minutes and a new screen will render when it has finished.

_Remember to note the username and password provided at the end of the install process_


## <a name="data_import"></a> Data Import

### _The Gallery Data_

For the purpose of demonstration, this walkthrough provides a [dataset](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Data/example_dataset.csv) that mimics that of a small art gallery. The data (the content of which has been obfuscated, while retaining its structure) was sourced from a university art gallery CMS and represents a subset of the entire dataset in the sense that the number of fields has been restricted to the twenty most-relevant fields (note, though, that all of the ca. 500 records – each representing an artwork – are represented).

It is important to note that this restricted dataset omits ca. 250 other fields that are relevant to the Gallery’s collection management functions. In particular, location tracking; valuation history; and conservation management information (each of which could be captured by Collective Access) are not present.


### Data Mapping Document

The process for mapping existing data in to a Collective Access instance is informed by way of a data mapping document, which is captured in an [Excel spreadsheet](http://docs.collectiveaccess.org/wiki/File:Data_Import_Mapping_template.xlsx) that adheres to a predefined structure and vocabulary.

Though there is useful documentation available that details the [mapping process](http://docs.collectiveaccess.org/wiki/Data_Import:_Creating_and_Running_a_Mapping) and [functions](http://docs.collectiveaccess.org/wiki/Data_Importer), it is extremely broad; instead, this specific example of the Gallery data provides a more-gentle introduction. Note that – for the sake of simplicity – this walkthrough does not leverage all of the mapping functionality supported by Collective Access; in particular, groups, replacement values, and several refineries have not been covered.

The remainder of this section will make reference to the supplied [data mapping](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Data/partial_mapping.xlsx) document. Note that this mapping is a 'partial' mapping in that not every field will have a target in the default Collective Access configuration – the complete mapping can be used once the necessary [modifications](#modification) have been made.

To begin, open the data mapping document ('partial_mapping.xlsx') and observe its structure: each row represents a column in the dataset to be mapped, and each column represents some action to be carried out on that specific data element as it is mapped in to Collective Access – in some sense, these columns describe a pipeline for the original data element.

The group of settings at the bottom of the document can be thought of as a means of configuring the behaviour of the mapping as a whole: they prescribe a name for the mapping, parameters pertaining to error handling, etc.

##### Skipped Fields

Glancing down the first and second columns, it is clear that we have the option to either migrate data (i.e the rule type will be _Mapping_) or not. For columns 7, 8, 11, 12, 13, 14, 17, and 18 in the dataset<sup>[3](#footnote_indexing)</sup>, we will elect not to map them (i.e. by choosing SKIP) on account of the fact that the data is irrelevant, incomplete, or because it cannot currently be mapped in to a field in Collective Access (i.e because it hasn't been created yet).

##### Direct Mapping

Next, observe that – as indicated in the _Source_ column – fields 1, 3, 4, 5, 15, and 16 (equivalently columns A, C, D, E, O, and P in the spreadsheet containing the data) have _Mapping_ as their ‘rule type’, an entry in the ‘CA table.element’ column, and no other specifications; this indicates that the data in these columns will be mapped directly to the field with the corresponding _bundle identifier_ in the ‘CA table.element’ column.

###### A Note On Determining Bundle Identifiers

A bundle identifier is essentially an 'address' for a specific field within the structure of Collective Access. Owing to the structure of Collective Access, it is possible to use a metadata element in a multitude of contexts: _date_, for instance, could refer to any one of an exhibition date; a date of birth; a date of accession; a date of sale; a date of valuation, and so on. To differentiate between specific _date_ fields, it is necessary to specify a bundle identifier, which uniquely identifies the specific _date_ field.

There are numerous ways to determine the bundle identifier for a specific field. The simplest method is to first consider the Basic Table that is most-relevant – if you are using the data to describe an _object_, this will be `ca_object`; a _person_ will be `ca_entity` <sup>[4](#footnote_basic_tables)</sup> – and use this to form the lefthand side of the dot-separated bundle identifier.

The righthand side (i.e. the field name) can be determined via the Collective Access UI:

1.	Log in to Collective Access
2.	Access the User Interfaces editor by selecting Manage->Administration->User Interfaces
3.	Click the edit button (the paper icon) in the row that corresponds to the editor whose Basic Table is relevant
4.	Select the Screen that displays the relevant field by clicking the associated edit icon
5.	Under Screen content->Elements to display on this screen” hover over the relevant field name and note the Bundle name; this will form the righthand side of the Bundle Identifier.


##### Refineries

Fields 2, 6, 9, and 10 are mapped using a _refinery_. Anecdotally, a useful rule-of-thumb for determining whether a refinery is relevant is to consider whether the data element to be mapped should be embedded (in which case, direct mapping is more appropriate) or referential (i.e. as in, referenced using a foreign key, in the relational sense).

As an example, Field 2 uses an entitySplitter refinery to create a reference to a person (i.e. `ca_entity`) record. The parameters to the refinery (expressed as JSON) indicate that the `relationshipType` (i.e. the role, activity, property right, etc. that connects an entity to an object) is ‘artist’ (i.e. the referenced entity was the artist that created the work); the `entityType` is ‘individual’ (i.e. as opposed to an organisation); and the `delimiter` is a space (indicating that the name of the entity is space-delimited such that all characters leading up to the first space forms a forename, then a middle name, then a surname, etc.).

Note that the `relationshipType` and `entityType` are types that _must_ already be defined. In general, the easiest way to discover these is to browse through the relevant lists in the Collective Access UI.

![The Lists and vocabularies heriarchy](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/lists_and_vocabularies.png "The Lists and vocabularies heirarchy") 

Types are found in the _Lists & vocabularies_ hierarchy, which can be accessed via Manage->Lists & vocabularies. The valid `entityTypes` are easily discovered under the Entity types (entity_types) heading – the value in brackets (e.g. individual) indicates the identifier for that type that should be passed to the refinery.

The relevant `relationshipType` can also be determined via the Collective Access UI by accessing the _Relationship Types_ hierarchy under Manage->Administration->Relationship Types. In this scenario, it is necessary to relate an object to an entity; as such, the most relevant type is object<->entity relationships. Here, the ‘had as artist’ (to be read as ‘this artwork had as its artist’) relationship type is identified internally as _artist_.

The same steps were taken to determine the refinery parameters for the other refinery mappings used in creating the data mapping.

##### Regular Expressions

Field 6 uses a regular expression to ‘trim’ extra information from the dimensions field. Dimensions are expressed in the data set in millimetres, then imperial units (which are themselves a conversion carried out internally in Vernon CMS anyway) – the regular expression will remove everything from ‘mm’ onwards leaving only the length x width expression.

Given that the result of the regular expression is not returned during the data import phase, it is important to test that the specified regular expression performs as expected; it is helpful to use a PHP regular expression tester to validate the expression<sup>[5](#footnote_phpliveregex)</sup>.

##### Settings 

Documentation surrounding the data mapping settings is not particularly helpful with regard to the table and type settings – a data mapping will probably work, even if all but these two settings are set incorrectly.

The [documentation](http://docs.collectiveaccess.org/wiki/Data_Importer#Settings) for the table setting indicates that this setting “Sets the table for the imported data Sets the table for the imported data”. In practice, it seems that a heuristic for setting this should be “what does my dataset principally describe?” – if the dataset predominantly contains information about objects, then the table setting should be set to `ca_objects`; if it predominantly contains information about people (e.g. biographical data pertaining to artists), then `ca_entities` would be more appropriate.

The type setting is defined as the “Type to set all imported records to...”. In the Gallery example, type is set to painting – this was derived from the Lists & vocabularies hierarchy (similarly to the relationshipType in the Refineries subsection).

Note that the dataset actually contains object types other than paintings, though their type is not explicitly identified (nor automatically inferrable) and as such the type specified in the settings cannot be overridden (i.e. by mapping to `ca_objects.type_id`) on a per-record basis.

### Importing Data

Data can be imported via the UI or via the command line; these instructions use the UI.

1.	Load the Collective Access UI and navigate to Import->Data
2.	Drag and drop the mapping document
3.	Choose the importer (its name corresponds to the name given in the name setting in the data mapping document
4.	Choose the file containing the data
5.	Hit “Execute data import”

![The data import UI](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/data_import.png "The data import UI")

Note that the process takes 2-3 minutes for the Gallery dataset.

##### Importing Media

The images associated with the Gallery data have been named such that it is possible to match a filename with a Object identifier. As such, the process of attaching images to their associated records is trivial.


It is first necessary to move the relevant images to the /import directory of the Collective Access installation. Assuming that the images are in the correct directory<sup>[6](#footnote_image_directory)</sup>, the following steps should pair properly-named images with the records that were created during the data import step.


1.	Load the Collective Access UI and navigate to Import->Media
2.	The import mode should be set to “Import only media that can be matched with existing records” – this ensures that images are paired with object records, and that object records having no contextual information are created
3.	Given that new records aren’t being created, it isn’t necessary to alter any other options
4.	Hit “Execute media import”


![A successful media import will result in an image being paired with its associated record](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/media_import_success.png "A successful media import will result in an image being paired with its associated record")

Note that some supplied images cannot be paired with any records due to improper naming, and vice versa. These anomalies are retained to demonstrate how the importer responds to these situations.



## <a name="modification"></a>Modification

The Visual Resources Collection (VRC) installation profile – used to set up Collective Access in the [installation](#installation) section – is the most suitable out-of-the-box profile for the Gallery data; however, there are a number of fields that do not map directly, or whose semantic is not preserved by the default nomenclature or behaviour imposed by the VRC profile. Additionally, a number of fields are superfluous and should be removed.

The following table lists fields from the original dataset that require modification of the VRC profile before their data can be accommodated; as a means of determining what kind of modification might be necessary, the purpose and type of each element is declared, and a means of adapting the schema to accommodate the data is suggested.


| Field Name              | Purpose                                                                                                                   | Field Type                                       | Solution                                                                                                                                                                                               | Modification                                                                                   |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Department              |                                                                                                                           |                                                  | It is unclear that this field is relevant – the decision to SKIP this field need not change                                                                                                                    | –                                                                                              |
| Other id                | Lists one or more other identifiers that were once ascribed to the object                                                 | Uncontrolled string                              | This field should be added on the Basic Info screen alongside the Object identifier field                                                                                                              | Add string field                                                                               |
| Current location reason | Indicates the reason that an object is stored in a particular location                                                    | Controlled string sourced from a validation list | This information could be included on the Storage screen within the Past storage locations bundle                                                                                                      | Add list field to Past storage locations bundle and delete the Current Storage Location bundle |
| Current location date   | Indicates the date on which an object was moved to a particular location                                                  | Date                                             | This information can already be captured within the Past storage locations bundle                                                                                                                      | -                                                                                              |
| Current owner           | Declares the owner for an object                                                                                          | References an Entity (e.g. Victoria University)  | The most appropriate screen for this information could be the Relationships screen, perhaps under Related entities with the relationship type “has as owner”                                           | Add “owner” relationship type                                                                  |
| Acquisition method      | Declares the method by which an object was acquired for the collection                                                    | Controlled string sourced from a validation list | The Acquisition reason (which should be paired with the Acquisition method) was mapped to the Provenance field. These two fields would more appropriately be validated fields in an Acquisition bundle | Create an Acquisition bundle with Reason and Method lists                                      |
| Credit line             | Indicates how interested parties should be credited when an artwork is published (e.g. in an exhibition, catalogue, etc.) | Free text                                        | The credit line could be added to either the Basic Info screen or the Administrative Info screen, perhaps by repurposing an existing field (e.g. Inscription)                                          | Rename Inscription to Credit line                                                              |
| Item count              | It is unclear that this field is relevant – the decision to SKIP this field need not change                                                                                      |                                                  |                                                                                                                                                                                                        | –                                                                                              |

#### Adding Fields

##### Other id

First it is necessary to create a free text field that can hold the “Other id” information. A new metadata element to support the data can be created by selecting New in the Metadata Elements screen of the Manage->Administration module.

Any fitting name, description, and internal element code for the the new field will suffice. Most importantly, though, the correct datatype must be identified – it has already been determined that Text is the most relevant.

Having chosen Text, a number of options for validating and displaying values are made available; the default settings are fine. 

Lastly, it is important to add a Type restriction that binds the Other ID attribute to objects (i.e. it doesn’t make sense for any concept in the system to have an Other ID associated with it). Note that selecting one of {Documents, Drawings, etc.} from the dropdown box will only allow an Other ID to be associated with a specific kind of object – the “-“ option generalises the association to all objects, which is the behaviour that we expect.

Having create the Other ID metadata element, it should be possible to go to Manage->Administration->User Interfaces->Standard object editor->Basic info and drag the Other ID field from the “Available editor elements” to the “Elements to display on this screen” column.

![The Other ID field may now be dragged from the Available editor elements column to the Elements to display on this screen column](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/add_other_id.png "The Other ID field may now be dragged from the Available editor elements column to the Elements to display on this screen column")

The Other ID field will now be available in the standard object editor UI. Notice that – as desired – many alternative identifiers can be associated with an object. This behaviour was induced by not imposing a “Maximum number of attributes of this kind that can be associated with an item” under the type restrictions that were declared when the metadata element was created.


![The Other ID field has been added to the list of metadata elements to be displayed on the standard object editor screen](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/populated_basic_info_screen.png "The Other ID field has been added to the list of metadata elements to be displayed on the standard object editor screen")

#### Adding Bundles

##### Acquisition

For the sake of this exercise, we will consider that the metadata pertaining to the _acquisition_ of an object is best-modelled as a collection of several fields that are bound together (i.e. visually). Collective Access groups fields together in a structure called a _bundle_ (the word _container_ is also used, depending on context – a _bundle_ seemingly refers to a visual component, whereas a _container_ is a metadata element).

To create a bundle, first navigate to Manage->Administration->Metadata Elements and choose 'new' from the top-left corner of the screen. At the very least, give the new element a `Name` (e.g. Acquisition) and `Element code` (e.g. acquisition_container), and ensure that the `Datatype` is set to _Container_.

After selecting _Save_, several configuration options will appear under the heading `Datatype-specific options`. The two most relevant of these are the `Type restrictions` and `Sub-elements` options.

Firstly, we should add a type restriction such that this metadata element can be associated with objects – proceed as in the last section by selecting _Add type restriction_, choosing _objects_ from the _Bind attribute to_ list, and then choosing the hyphen such that availability of the field will generalise to all sub-types.

Now navigate to the bottom of the page to the `Sub-elements` section.

Choosing `Add subtype` will lead us to an identical input screen as that from which we created the Acquisition container – in this context, we are using this screen to create fields that will be children of the Acquisition container.

As before, give the field a name and an element code (e.g. Method and acquisition_method). This time, however, set the `Datatype` to Text and choose Save.

On the left-hand side of the screen, under the heading _Viewing metadata element_, there should be a hyperlink that will take us back to the Acquisition container that we had created – click on this link, and observe that our Method field has been added as a sub-element of our Acquisition container.

Now, add a second sub-element with the `name` Reason, the `element_code` acquisition\_reason, and the `datatype` Text.

This completes the construction of the Acqusition container; it should now be available to add to the Standard Object Editor (Manage->Administration->User Interfaces->Standard object editor edit icon).

This bundle probably belongs on the Administrative Info screen, so select its edit icon. Observe that Acquisition is now represented in the `Available editor elements` list – drag it to `Elements to display on this screen` and choose Save.

![The Acquisition bundle is now available in the Available editor elements list](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/acquisition_bundle_available.png "The Acquisition bundle is now available in the Available editor elements list")

Create a new object via the UI and observe that an Acquisition bundle is now available. Note, however, that and `Add Acquisition` button is available; this doesn't make much sense, given that an object can probably only be acquired in one way.

![The bundle is configured to allow many entries under Acquisition](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/repeating_sub_elements.png "The bundle is configured to allow many entries under Acquisition")

The problem here is that we have allowed for repeating sub-elements. This behaviour can be changed by returning to Manage->Administration->Metadata Elements->Acquisition and ensure that "Maximum number of attributes of this kind that can be associated with an item" is set to one (here, zero means "unbounded").

Choose Save and return to the Administrative Info screen of an arbitrary object – it should only be possible to add one entry under Acquisition.

![Only one entry can be created under Acquisition](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/non_repeating_sub_elements.png "Only one entry can be created under Acquisition")


###### Aside: Validation Lists

A second deficiency in our newly-created bundle is that acquisition methods and reasons should probably conform to a vocabulary, otherwise it would be difficult to report on the overall provenance of the collection (e.g. it is difficult to report that 35% of the collection has been donated if designations such as "donation", "gift", "donated" and "gifted" have all been used synonymously).

As such, given the nature of the data and its utility, it seems reasonable to populate the _reason_ and _method_ fields from validation lists.

To do this, we must first create lists that will be populated (either by hand or during data import) with controlled terms.

Navigate to Manage->Lists & vocabularies and choose "Add new list". First, create a list with the Preferred label "Acquisition Method" and the List code "acquisition\_method\_list". Choose Save.

Do the same for a second list having the Preferred label "Acquisition Reason" and the List code "acquisition\_method\_list".

Now, navigate to Manage->Administration->Metadata Elements->Acquisition edit icon, navigate to the bottom of the screen, and choose the edit icon next to Method.

This should present the metadata element editor UI for the Method field that we created earlier. We previously selected Text as the Datatype; we should change this to List, and choose Acquisition method from the Use list (for list elements only) dropdown list.

Choose Save, and repeat the exercise for the Reason field.

![The Acquisition Reason will now be bound to the Acquisition Reason list](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/acquisition_reason_configuration.png "The Acquisition Reason will now be bound to the Acquisition Reason list")

Now navigate to the Administrative Info screen for an arbitrary object and note that the Reason and Method are represented as dropdown lists. Note, however, that the lists aren't populated – they can be populated by navigating to Manage->Lists & vocabularies, selecting the relevant list, and then choosing to add a new concept under the selected list (lists are selected by choosing the spot or arrow on their right).

![Adding a new list item](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/add_new_list_item.png "Adding a new list item")

For the sake of demonstration, add a new list item to Acquisition Method by clicking on the plus icon; give the new list item the singular name "Gift"; the plural "Gifts"; the identifier "gift"; and the item value "gift".

Choose Save, then navigate back to the Administrative Info screen of an arbtirary object and observe that "Gift" is now available in the validation list for Method.

![Gift is now available as an Acquisition Method](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/acquisition_method_gift.png "Gift is now available as an Acquisition Method")

#### Redefining Fields

##### Credit Line

As a primer, we will alter the Inscription field such that its display label is “Credit Line”. This is achieved by navigating to Manage->Administration->Metadata Elements and selecting the edit icon in the row corresponding to Inscription. Renaming this field is as simple as changing the Name to Credit Line (note, too, that we can delete the French label).

Observe, now, that the Inscription field on the Basic Info screen now reads “Credit Line”. Note, though, that the internal element code is still “inscription”. This could be changed to credit_line, though this will require adding the new reference to the list of fields in the Basic Info screen (this process is described below).

![The Inscription field has been renamed to Credit Line](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/renamed_field.png "The Inscription field has been renamed to Credit Line")

##### Location

The Art Gallery storage location information is comprised of (location, date, reason) triplets that should be grouped together within a bundle. One such candidate is the Past storage locations bundle on the Storage screen – this bundle already has fields for a location (albeit freetext rather than a StorageLocation type), date, and authorisation.

It seems reasonable<sup>[7](#footnote_storage_location)</sup> to redefine the location field as a StorageLocation and the authorisation could be repurposed as a Current location reason field.

To do this, the Past storage locations metadata element should be edited by choosing the edit icon associated with the Manage->Administration->Metadata Elements->Past storage locations element.

Under the Sub-elements heading, first edit the Date of Move such that its Name is now simply “Date”.

Similarly, choose to edit the Storage Location container, then edit the datatype of its Storage Location Sub-element – change this to StorageLocations.

Lastly, we will use these same techniques to redefine the Authorisation field to be labelled Reason, with the datatype List. Before associating the field with the List datatype, though, it is first necessary to define a storage_reasons validation list.

This is achieved by navigating to Manage->Lists & vocabularies; choose “Add new list”. Give the list a suitable label (e.g. Storage Location Reason) and list code (e.g storage_reason), and set the list to be used as a vocabulary.

It is now possible to define Reason as a list whose elements are sourced from the Storage Location Reason validation list.

Now, rename the Past storage locations bundle and its element code to something sensible (e.g. Past storage locations should probably be Storage Locations, since we will remove the Current storage location field as it did not encode enough detail for the Art Gallery’s data)

Lastly, to eliminate redundancy, remove the Current storage location bundle from the standard object editor UI – to do this, edit the Storage screen of the standard object editor UI and drag the Current storage locations element from the “Elements to display on this screen” column to the “Available editor elements” column.

Note that, given that Past storage locations was renamed, it will need to be dragged from “Available editor elements” to the “Elements to display…” column.

##### Deleting Fields

Having added or renamed several fields in order to accommodate data, it is clear that there are a number of superfluous fields and validation list entries that should be removed in order to prevent ambiguity or tempt divergences in cataloguing practices.

There are two ways to delete a field: removing a field from a UI screen could be considered a "soft-delete", whereas removing a field altogether can be achieved by removing its corresponding metadata element (note, though, that a metadata element may be used in many contexts and deleting it altogether could have adverse side-effects).

Removing a field from a user interface can be achieved by navigating to Manage->Administration->User Interfaces->[Interface to edit] edit icon->Screen to edit. Now just drag the field from the "Elements to display on this screen" list to the "Available editor elements" list.

Now consider that we don't want a particular metadata element to be available for a particular kind of basic table – say, External links should never be associated with a Place.

First, navigate to Manage->Administration->Metadata Elements->[Element to restrict] edit icon. Now, click the delete icon next to the Places type restriction. It will no longer be possible to record External Links for a Place.

Lastly, to delete a field from the system altogether, navigate to Manage->Administration->Metadata Elements->[Element to delete] and hit the delete icon – you will be asked whether you really want to delete the field.

As an exercise, consider deleting the Edition Number field, which is irrelevant to the Gallery dataset. Navigate to Manage->Administration->Metadata elements and click the delete icon next to Edition number.

Now observe that Edition Number is no longer present on the Basic Info screen for an object.

### Data Import: Second Attempt

Having created fields that will act as targets for in accordance with the table at the beginning of this section, we can now augment the original data mapping and 'append' the data that we were forced to SKIP in our first iteration. The new data mapping document will be an iteration of the original mapping with the following actions will need to be taken:

* Field 8 (Other Id) should be mapped directly to the `ca_objects.other_id` field
* Fields 11-13 will be mapped to each of **TODO**, **TODO**, **TODO**
* Fields 14-15 will be mapped to **TODO** and **TODO**
* Field 17 (Credit Line) should be mapped directly to `ca_objects.inscription` field (note that the internal identifier for what is now the Credit Line field was not altered)

The data import process will be identical; however, in the settings section of the spreadsheet, `existingRecordPolicy` should be set to `merge_on_idno` – this orders the mapping process to merge the new data in to records having the same `idno` (i.e. accession number, in the parlance of the dataset) as the row of the dataset that is being processed.

For the sake of verification, a [working mapping](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Data/complete_mapping.xlsx) document has been supplied.

## Additional Tips

##### Re-Initialisation

It is often necessary to initialise a Collective Access instance. Given that the state of a Collective Access instance is typically tied to the state of the database (i.e. if the configuration files in /app/conf have not been substantially altered), it is usually unnecessary to reinstall the entire system; rather, dropping the database is more appropriate.

The following process can be used to revert a Collective Access instance to its initial state.

1.	From the command line, run mysql as root
2.	Drop the Collective Access database (here, the database name is ca – this is specified in setup.php)
3.	Create an empty database with an identical name to the database that was dropped
4.	Access the Collective Access web interface
5.	Run the installer


![Reinstallation steps  1-3](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/initialising_providence.png "Reinstallation steps  1-3")

If these steps have been carried out correctly, Collective Access will present the installer interface.

![The web-based installer interface](https://github.com/MisterTJB/Collective-Access-A-Walkthrough/blob/master/Screenshots/installer_screen.png "The web-based installer interface")



## <a name="footnotes"></a>Footnotes
1. <a name="footnote_types"></a>The full list of types is enumerated [here](http://docs.collectiveaccess.org/wiki/Basic_Tables)
2. <a name="footnote_php"></a>Note that you may encounter a splash screen stating that "An error in your system configuration has been detected", citing the lack of the PHP cURL dependency. This will occur because the Ansible provisioning script retrieves the latest version of Collective Access (1.6) – requiring PHP cURL – but does not install PHP cURL. This can be remedied from the command line by ssh'ing in to the Vagrant virtual box using the command `vagrant ssh` from the ~/Workspace/ansible-collectiveaccess directory. Having logged in to the Vagrant box, run `sudo apt-get install curl php5-curl`
3. <a name="footnote_indexing"></a>Note that 1-indexing is used such that Column A in the dataset corresponds to 1, B to 2, etc.
4. <a name="footnote_basic_tables"></a>Recall that the [Basic Tables documentation](http://docs.collectiveaccess.org/wiki/Basic_Tables) maps concepts to Basic Table names
5. <a name="footnote_phpliveregex"></a>[PHPLiveRegex](http://www.phpliveregex.com) was helpful
6. <a name="footnote_image_directory"></a>If, after step one, you see that there are files listed under “Directory to import”, then the files are in the correct location
7. <a name="footnote_storage_location"></a>Note that this is something of an abuse, presented here as a demonstration of making modifications to bundles and redefining fields and their datatypes – the storage location subsystem is important for inventory tracking and is associated with, for instance, the Movement functionality. There is undoubtedly a better method for recording location metadata
