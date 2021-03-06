
---
title: Oyranos/Code Generator
permalink: wiki/Oyranos/Code_Generator/
layout: wiki
---

How it works
------------

### Overview

The **oyAPIGenerator** executable takes 3 arguments as input.

-   A directory the templates files using the grantlee format.
-   A source directory with all included code from the templates.
-   An output directory were all auto-generated code is created.

All are optional, and in an *in source* build none of them is actually
used, because they get the default values (respectively):

-   ./templates
-   ./sources
-   ./API\_generated

Actually, the **extract\_sources.sh** shell script is used for
convenience. It cleans the API\_generated/ directory, creates the code,
and then builds it. The paths work for an *in source* build.

### The details

#### Template system

The templates are written in the [Django template
language](http://docs.djangoproject.com/en/dev/topics/templates/). A
Django template is a plain text file with some specially formatted
**tags** and **variables**. So, a template is just a C or C++ file that
contains some extra **{\% tag \%}** and **{{ variable }}** embedded code
that is replaced by the code generator. This is much like *php* code is
replaced in *html* files. The built in **tags** and **variables** are
[documented
here](http://docs.djangoproject.com/en/dev/ref/templates/builtins/). The
Django template engine is written in python and can be used [in stand
alone
mode](http://docs.djangoproject.com/en/dev/ref/templates/api/#configuring-the-template-system-in-standalone-mode).
Instead, the *generator* is using the [Grantlee template
system](http://www.grantlee.org/), which uses the same template
language, but is written in C++ and depends on
[Qt](http://qt.nokia.com/).

So, how is a new source code file auto-generated?

-   First the *generator* scans the templates/ directory for template
    files. They have a special file name in the form of
    file\_name.*template*.ext, e.g. oyranos\_module.*template*.h
-   Then the various template variables are loaded (taken from the
    source code metadata) and the template is rendered in memory
-   A new file is created in API\_generated/ with the *.template.*
    removed, e.g. oyranos\_module.h

#### Template files

There are mainly two kinds of template files.

-   A [*base*](#Files "wikilink") template.
-   A *child* template that extends a *base* template. Most templates
    are *child* templates.

The *child* templates use the {\% extends \%} **tag**, which is how
[template
inheritance](http://docs.djangoproject.com/en/dev/topics/templates/#template-inheritance)
is implemented in the django language. There is a 1-1 relationship
between the class inheritance of the Oyranos object system and the
django template inheritance. For example, that means if *oyCMMapi10\_s*
inherits from *oyCMMapiFilter\_s*, then *CMMapi10\_s.template.c* will
extend *CMMapiFilter\_s.template.c*. And if *oyCMMapiFilter\_s* extends
*oyCMMapi\_s*, then *CMMapiFilters\_s.template.h* will extend
*CMMapi\_s.template.h*.

| Oyranos Objects   |     | Template Files             |
|-------------------|-----|----------------------------|
| oyStruct\_s       | ↔   | Struct\_s.template.h       |
|                   |     | ↑                          |
| ↑                 |     | Base\_s.h                  |
|                   |     | ↑                          |
| oyCMMapi\_s       | ↔   | CMMapi\_s.template.h       |
| ↑                 |     | ↑                          |
| oyCMMapiFilter\_s | ↔   | CMMapiFilter\_s.template.h |
| ↑                 |     | ↑                          |
| oyCMMapi10\_s     | ↔   | CMMapi10\_s.template.h     |

`The example above shows the inheritance graph for a public header file,`  
`of a common Oyranos class.`

| Oyranos Objects |     | Template Files          |
|-----------------|-----|-------------------------|
| oyStruct\_s     |     |                         |
|                 |     | Base\_s\_.c             |
| ↑               |     | ↑                       |
|                 |     | BaseList\_s\_.c         |
|                 |     | ↑                       |
| oyOptions\_s\_  | ↔   | Options\_s\_.template.c |

`The example table above shows the inheritance for a private implementation file,`  
`of a special kind of Oyranos class - a `*“`list`”*` class. Note that the `*`Base_s_.c`*  
`template does not extend a supposed `*`oyStruct_s_.template.c`*` file, because oyStruct_s`  
`only has a public interface and so no private `*`oyStruct_s_.*`*` implementation files.`

If in doubt, each generated source file has it's inheritance graph
embedded, at the top of the *@file* doxygen comment.

#### Template file auto-generation

As stated above, there are two types of template files.

-   The [*base*](#Files "wikilink") template files: *Base\_s\*\[ch\]*
    and *BaseList\_s\*\[ch\]*. These templates contain all the generic
    template code. All template files that are used to generate a C
    source file, inherit (directly or indirectly) one of the respective
    Base\* files. These files should not be edited, unless a global
    change to all generated source files is needed.
-   Files named as *<class_name>\*.template.\** . Each one of these
    files is used to auto-generate a source file in API\_Generated/
    (with the “.template” stripped from the filename). Each time the
    generator is run, it checks the sources/<class_name>.dox file for
    the [**\[notemplates\]**](#<class>.dox "wikilink") tag. If it's not
    there, the *<class_name>\*.template.\** file is auto-generated from
    the respective *Class\_s.\[ch\]* file. Their usage is for overriding
    the inherited blocks, eg to [insert include
    files](#How_to_insert_include_files "wikilink"), etc.

So, the second type of templates is autogenerated at first, and then
edited by the user (if needed).

Code Organisation
-----------------

### File Structure

#### API\_generated/

All the files in this directory are auto-generated from the templates.

##### Classes

Each class is implemented by four files.

oyClass\_s.h  
This is the public header file that exports the class API

oyClass\_s.c  
The implementation of all the public class parts.

oyClass\_s\_.h  
Private declarations

oyClass\_s\_.c  
Private definitions

##### Misc

oyranos\_object.h  
This file includes some vital oyranos headers and also contains

definitions necessary for, but **not** part of the object system.

It is included by all the *oyClass\_s.h* headers.

oyranos\_object\_internal.h  
This file includes some private oyranos headers and definitions
necessary for,

but **not** part of the object system.

It is included by the private *oyClass\_s\_.c* implementation files.

oyTest.\[cc|h\]  
Tests using the Qt test framework. For every generated class the basic
functionality

like create, copy destroy is tested.

CMakeLists.txt  
The generated file for building the code with cmake.

##### Modules

oyranos\_generic.h  
For everything that belongs to the Generic Objects API and is not part
of a specific class.

oyranos\_generic\_internal.h  
All Generic Objects API helper code, that is for internal usage only.

It is included by all the private implementations (*oyClass\_s\_.c*
sources) that belong to the Generic Objects group.

oyranos\_generic.c  
For private/public functions that belong to the Generic Objects API but
do not belong to a specific class.

<!-- -->

oyranos\_devices.h  
Exports all public declarations that are part of the Device API.

oyranos\_devices.c  
The implementation for the Device API.

oyranos\_devices\_internal.\[ch\]  
All Device API code that is for internal usage.

oyranos\_module.h  
All declarations that are part of the Module APIs, but not part of a
specific class, and should be exported.

oyranos\_module\_internal.h  
Used for all members of the Module APIs that are not part of any class
and should not be exported.

It is included by all the private implementations (*oyClass\_s\_.c*
sources) that belong to the Module APIs group.

oyranos\_profile.h  
All declarations that are are part of the Profile API, but not part of a
specific class, and should be exported.

oyranos\_conversion.h  
All public declarations of the Conversion API's that do not belong to a
specific class, go here.

oyranos\_image.h  
All public definitions of the Image API that do not belong to a specific
class, go here.

oyranos\_image\_internal.h  
All private definitions of the Image API that do not belong to a
specific class, go here.

#### sources/

Here is all the source code that does not need to be inside the
templates.

##### <class>.dox

This is the Doxygen description for the class, with some additional
tags.

<code>

`/** @struct  oyClass_s`  
` *  @ingroup some_group`  
` *  @extends oyStruct_s`  
` *  @brief   Brief description`  
` *  @internal`  
` *`  
` *  Multi line description`  
` *  @note New templates will not be created automaticly [notemplates]`  
` *  @note Create templates using `“`opaque`` ``pointer`”` [opaquepointer]`  
` *  @note This class holds a list of objects [list]`  
` *`  
` *  @version Oyranos: x.x.x`  
` *  @since   YYYY/MM/DD (Oyranos: x.x.x)`  
` *  @date    YYYY/MM/DD`  
` */`

</code>

The basic idea is that the *generator* needs to know all kinds of
information **(metadata)** about the class and these are provided here.
Along with the doxygen tags, a few additional are also needed and are
put in the *@note* tag.

\[notemplates\]  
Each class has a template file for each generated source file.

At class creation, these are also created automaticly and are read-only.

When for any reason you want to override some default template block and

edit the class templates, change their permissions to read-write and

remove the \[notemplates\] tag

\[list\]  
This tag specifies that the class is a special kind of class, a *list*
of values. The

convention is that the class name is in plural *(ends with s)* and the
list item type

is the class with the same name without the *s*. E.g. oyFilterPlugs\_s
-&gt; oyFilterPlug\_s

\[opaquepointer\]  
This is an alternative way of the API and should not be used now.

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s.h* file.

##### <class>.members.h

A list of the class members, but *not* inherited members from parent
classes, e.g. for **CMMapi6.members.h**

<code>

` /** oyCMMapi4_s::context_type typic data; e.g. `“`oyDL`”` */`  
` char           * data_type_in;`  
` /** oyCMMapi7_s::context_type specific data; e.g. `“`lcCC`”` */`  
` char           * data_type_out;`  
` oyCMMdata_Convert_f oyCMMdata_Convert;`

</code>

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s\_.h*
file.

##### <class>.public.h

Here goes code for the oyClass\_s.h public header file, e.g. for
**Options.public.h** <code>

`typedef oyStruct_s * (*oyStruct_Copy_f ) ( oyStruct_s *, oyPointer );`  
`typedef int       (*oyStruct_Release_f ) ( oyStruct_s ** );`  
`typedef oyPointer (*oyStruct_LockCreate_f)(oyStruct_s * obj );`  
`...`  
`extern oyStruct_LockCreate_f   oyStruct_LockCreateFunc_;`  
`extern oyLockRelease_f         oyLockReleaseFunc_;`  
`extern oyLock_f                oyLockFunc_;`  
`extern oyUnLock_f              oyUnLockFunc_;`  
`...`

</code>

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s.h* file.

##### <class>.public\_methods\_declarations.h

All declarations of the class's public interface, that is not generated
automaticly. E.g, for Profile.public\_methods\_declarations.h: <code>

`...`  
`OYAPI oyProfile_s * OYEXPORT`  
`                   oyProfile_FromMD5(  uint32_t          * md5,`  
`                                       oyObject_s          object );`  
`OYAPI int OYEXPORT`  
`         oyProfile_GetChannelsCount ( oyProfile_s * colour );`  
`OYAPI icSignature OYEXPORT`  
`             oyProfile_GetSignature (  oyProfile_s       * profile,`  
`                                       oySIGNATURE_TYPE_e  type );`  
`...`

</code>

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s.h* file.

##### <class>.public\_methods\_definitions.c

The implementation file of the above public interface.

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s.c* file.

##### <class>.private.h

All class definitions, enums, structs, etc that should not be exported
should go here.

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s\_.h*
file.

##### <class>.private\_methods\_declarations.h

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s\_.h*
file.

##### <class>.private\_methods\_definitions.c

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s\_.c*
file.

##### <class>.private\_custom\_definitions.c

**<u>NOTE:</u>** This file will be embedded in the *oyClass\_s\_.c*
file.

#### templates/

##### Files

###### Base\_s.h

###### Base\_s.c

###### Base\_s\_.h

###### Base\_s\_.c

###### BaseList\_s.h

###### BaseList\_s.c

###### BaseList\_s\_.h

###### BaseList\_s\_.c

###### CMakeLists.template.txt

###### oyTest.template.h / oyTest.template.cc

##### Directories

The directories are named by the group name (*@ingroup* tag) and hold
the template files of the classes that belong to that group.

### Naming conventions

#### Function prototypes

Public member functions  
All input/output variables use the public class interface. E.g:

<code>

`OYAPI oyProfile_s* OYEXPORT`  
`  oyProfile_Copy( oyProfile_s *profile, oyObject_s obj );`

</code>

Private member functions  
The input/output variables of the function class type use the private
class interface. E.g:

<code>

`oyProfile_s_*`  
`  oyProfile_Copy_( oyProfile_s_ *profile, oyObject_s object);`

</code>

  
The rest variables use their public interface. E.g:

<code>

`oyProfileTag_s * oyProfile_GetTagByPos_ ( oyProfile_s_    * profile,`  
`                                          int                 pos );`  
`int                oyProfile_TagMoveIn_ ( oyProfile_s_      * profile,`  
`                                          oyProfileTag_s   ** obj,`  
`                                          int                 pos );`

</code>

How to import a new class
-------------------------

### Steps to use

#### 1. Initial commit

Directory  
sources/

-   (a) cp Class.dox <class>.dox & edit
-   (b) run generator
-   (c) Edit <class>.members.h
-   (h) run generator

*`NOTE`*` Skip steps (c)&(h) for list classes.`

git short comment  
\* Create skeleton files for oyClass\_s

#### 2. Import class members like enums,typedefs,...

Search for them in oyranos sources  
grep 'memberof \*oy<class>\_s' \* -B3

-   Private ones in sources/<class>.private.h
    git short comment  
    \[sources\] Import oy<class>\_s private \[enums,typedefs,...\]

-   Public ones in sources/<class>.public.h
    git short comment  
    \[sources\] Import oy<class>\_s public \[enums,typedefs,...\]

-   Add proper include files in oyClass\_s.h and oyClass\_s\_.h
    Find them by trying to compile the object files  
    cd /API\_generated/

    make oyClass\_s.o

    make oyClass\_s\_.o

    git short comment  
    \[templates\] Add include files to oyClass\_s.h

#### 3. Implement constructor \[oyClass\_New\]

files:  

`sources/`<class>`.private_custom_definitions.c`

git short comment  
\[review\] \[sources\] Implement the constructor for oyClass\_s

#### 4. Implement copy constructor \[oyClass\_Copy\]

Files:

`sources/`<class>`.private_custom_definitions.c`

git short comment  
\[review\] \[sources\] Implement the copy constructor for oyClass\_s

#### 5. Implement destructor \[oyClass\_Release\]

sources/

`.private_custom_definitions.c`

\[review\] \[sources\] Implement the destructor for oyClass\_s

#### 6. Import private methods for oyClass\_s

sources/

`.private_methods_declarations.h`  
`.private_methods_definitions.c`

\[sources\] Import private methods for oyClass\_s

#### 7. Adopt oyClass\_s private methods

sources/

`.private_methods_declarations.h`  
`.private_methods_definitions.c`

Make a different commit for each refactored function:

\[review\] \[sources\] Adopt oyClass\_XXX\_() to “hidden struct”
interface.

#### 8. Import public methods for oyClass\_s

sources/

`.public_methods_declarations.h`  
`.public_methods_definitions.c`

\[sources\] Import public methods for oyClass\_s

#### 9. Adopt oyClass\_s public methods

sources/

`.public_methods_declarations.h`  
`.public_methods_definitions.c`

Make a different commit for each refactored function:

\[review\] \[sources\] Adopt oyClass\_XXX() to “hidden struct”
interface.

### Tips n' Tricks

#### How to insert include files

To use include files in e.g. *oyClass\_s.c* override the
LocalIncludeFiles block in the *Class\_s.template.c* template file.
There are 2 blocks available for every template.

LocalIncludeFiles  
For local tree include files

GlobalIncludeFiles  
For system files

For example, to include a system *icc34.h* file in *oyProfile\_s\_.h*,
add the following in *Profile\_s\_.template.h*: <code>

`{\% block GlobalIncludeFiles \%}`  
`{{ block.super }}`  
`#include <icc34.h>`  
`{\% endblock \%}`

</code> To also include *oyStructList\_s.h*, *oyProfileTag\_s.h*,
*oyConfig\_s.h*: <code>

`{\% block LocalIncludeFiles \%}`  
`{{ block.super }}`  
`#include `“`oyStructList_s.h`”  
`#include `“`oyProfileTag_s.h`”  
`#include `“`oyConfig_s.h`”  
`{\% endblock \%}`

</code> The above blocks will render as (in *oyProfile\_s\_.h*): <code>

`...`  
`#define oyProfilePriv_m( var ) ((oyProfile_s_*) (var))`  
  
`#include <icc34.h>`  
  
`#include <oyranos_object.h>`  
  
`#include `“`oyStructList_s.h`”  
`#include `“`oyProfileTag_s.h`”  
`#include `“`oyConfig_s.h`”  
  
`#include `“`oyProfile_s.h`”  
  
`typedef struct oyProfile_s_ oyProfile_s_;`  
`...`

</code>

#### How to use the f\_compare.sh script

This script can show you the differences of a function definition
between two branches. The function should have a doxygen header (even an
empty one). <code>

`Usage: extras/f_compare.sh [-h|-s] `<function>` [branch:]`<file1>` [branch:]`<file2>  
  
`-h      Show this help message`  
`-s      Show just the diff output`

</code> If *branch* is ommited, the current branch is assumed. Two files
are created at the working directory, each having the function body as
it's contents. Without the *-s* switch, *vimdiff* is run on those two
files. They are named as: *branchname-filename-funcname.c*

So, why is this script useful? When a function is imported to the
template system, it is copied from the oyranos\_alpha.c or
oyranos\_cmm.c files, into the relevant one in the sources/ directory.
That happens in the *gsoc2011* branch. If the function is changed in
*master*, before the working branch gets merged back, then the copied
function in sources/ has to be updated, too.

For example, *oyConfig\_Find()* is imported from oyranos\_alpha.c in
sources/Config.public\_methods\_definitions.c. If we are in the
*gsoc2011* branch and want to check if the function has been updated in
master, we have to call the f\_compare.sh script like: <code>

` ./extras/f_compare.sh oyConfig_Find master:oyranos_alpha.c sources/Config.public_methods_definitions.c`

</code> Now you can add the updates to the function in vimdiff, save the
file, and then replace the function in
*sources/Config.public\_methods\_definitions.c* with the
*gsoc2011-Config.public\_methods\_definitions.c-oyConfig\_Find.c* file.
