.. _zend.uri.introduction:

Zend\\Uri Introduction
======================

The ``Zend\Uri`` component provides classes for representing, parsing, composing
and manipulating `Uniform Resource Identifiers`_ (URIs) [#]_. While originally
created to service other components, such as ``Zend\Http\``, ``Zend\Uri`` 
is also quite useful as a standalone component. 

``Zend\Uri`` was designed to follow the **Generic URI Syntax**. This means
it generically supports URIs of any scheme (and not just specific shcemes such 
as `http` or `file`); In addition, both absolute URIs and relative URIs are 
supported, for URI schemes where such a distinctions exist. 

For some URI schemes, ``Zend\Uri`` offers a :ref:`scheme-specific subclass <zend.uri.subclasses>` 
of ``Zend\Uri\Uri``, providing scheme specific functionality and validation rules
beyond those described in the Generic URI Syntax specifications. 

In addition, the ``Zend\Uri\UriFactory`` class offers a :ref:`factory for creating
scheme-specific objects <zend.uri.factory>` from arbitrary strings where the 
scheme is not known in advance. 

.. _zend.uri.definition:

What is a URI?
--------------
The term `Uniform Resource Identifier` (`URI`) is used in different contexts 
and some confusing, conflicting or plain wrong definitions of it could be found. 
In addition, it is often interchanged with related terms such as `Uniform 
Resource Locator` (`URL`) or `Uniform Resource Name` (`URN`). ``Zend\Uri`` 
terminology and semantics follow the definition from RFC-3986, which is widely 
regarded as the official definition. 

In plain terms, a `URI` is a string of characters used to identify a resource,
which could be a physical, digital or abstract. A URI can be classified as 
either a `Uniform Resource Locators` (`URL`) or a `Uniform Resource Name` 
(`URN`), or both. 

A `URN` is a URI which identifies a resource by its name in a specific namespace. 
For example, `urn:ietf:rfc:3986` is a URN identifying IETF RFC document number
3986. 

A `URL` is a URI which specify the location of a resource, answering the 
question "How can this resource be found and accessed?" in addition to identifying
the resource. For example, the URI `http://tools.ietf.org/html/rfc3986` identifys 
the RFC-3986 resource but also describes how it can be located and fetched - that
is, via the `http` protocol on the server `tools.ietf.org`, at the specified path. 

HTTP URIs are also URLs, and in the context of World Wide Web resources these
two terms can be used almost interchanably. Since HTTP URIs are arguably the 
most common and well known types of URIs today, the terms `URI` and `URL` are 
often confused or used as synonyms in popular publications. However, IETF and 
W3C recommendations all prefer the usage of the term `URI` in technical 
pulications. 

.. note:: 

  **Absolute vs. Partial URIs**

  In contrast to some beliefs, a URI is not a partial URL or vice verse. URIs 
  and URLs can be either absolute (full) or relative (partial). The type and 
  meaning of a partial URI is inferred from the context in which it is found. 

It may be important to clarify that URIs are designed as a mechanism to identify
resources - not access or manipulate them. Like wise, ``Zend\Uri`` is designed
to describe and manipulate **URIs**, not the resources they identify. For this
pupose, other components such as ``Zend\Http`` exist. 

.. _zend.uri.creation:

Creating New URI Objects
------------------------
In most cases, URI objects are created from an existing URI string (which can be 
either partial or complete) by passing it as an argument to the ``Uri`` 
constructor: 

.. _zend.uri.creation.example-1:
.. rubric:: Creating a new URI from a string:

.. code-block:: php
   :linenos:

   // Create a new URI from a string:
   $uri = new Zend\Uri\Uri('ftp://myhost.local/path/to/file.ext');

   // Create a new, partial URI:
   $uri = new Zend\Uri\Uri('/path/to/file.ext'); 

This is in fact equivalent to creating a new ``Uri`` object and then using the 
`parse()` method: 

.. _zend.uri.creation.example-2:

.. code-block:: php
   :linenos: 

   // Create a new URI from a string:
   $uri = new Zend\Uri\Uri
   $uri->parse('ftp://myhost.local/path/to/file.ext');

The provided URI string will be parsed into its different parts. If for some 
reason the string cannot be parsed as a URI, a ``Zend\Uri\Exception\InvalidArgumentException``
will be thrown. However, these cases are usually more rare than one might think. 

.. note::

  If you know in advance that the URI you are parsing or constructing is of 
  one of the schemes for which a specific subclass of ``Zend\Uri\Uri`` is 
  provided, it is recommended to instantiate that subclass instead. See 
  :ref:`this section <zend.uri.subclasses>` for a list of provided subclasses. 

  For creating URI objects of specific subclasses where the URI scheme is not 
  known in advance, use the :ref:`Zend\\Uri\\UriFactory class <zend.uri.factory>`.

You can also start with an empty URI and compose its different parts 
programatically:

.. _zend.uri.creation.example-3: 
.. rubric:: Composing a new URI from scratch

.. code-block:: php
   :linenos:

   // Programmatically create a 'file:' URI for the current file
   $uri = new Zend\Uri\Uri;
   $uri->setScheme('file')
       ->setPath(__FILE__);

.. _zend.uri.manipulation:


Composing a URI object back into a string
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In most cases, you will need to convert a validated or manipulated URI object
back to a string before you can use it in output, network protocols, etc. 

This can be done via the ``toString()`` method. This method is also wrapped by 
the ``__toString()`` magic method, which means you can simply cast a ``Zend\Uri\Uri``
object to a string or use it in any string context: 

.. _zend.uri.tostring.example-1: 
.. rubric:: Stringifying a URI object

.. code-block:: php
   :linenos:
   
   // Create a URI object
   $uri = new Zend\Uri\Uri('urn:isbn:0451450523');
	
   // Print it as a string
   $uriStr = $uri->toString();
   echo $uriStr; 

   // .. this is equivalent: 
   $uriStr = (string) $uri;
   echo $uriStr; 

   // .. and in fact since echo expect a string, you can just to this:
   echo $uri; 


.. _zend.uri.uri_parts:

Accessing and Manipulating URI Parts
------------------------------------
A URI is composed of 8 designated parts, all of which except for the path are 
optional: `scheme`, `user info`, `host`, `port`, `path`, `query` and `fragment`.
These parts are composed as a single URI in the following manner:

.. code-block::
   scheme://user-info@host:port/path?query#fragment

For example, in the URI :
.. code-block::
   http://billy:pilgrim@www.example.com:8080/path/to/file.ext?q=foo&b=baz#somefragment

- The scheme is `http`
- The user information is `billy:pilgrim`
- The host is `www.example.com`
- The port is `8080`
- The path is `/path/to/file.ext`
- The query is `q=foo&b=baz`
- The fragment is `somefragment`

``Zend\Uri\Uri`` offers fluent API for manipulating and accessing each one of 
the parts without affecting the entire URI. 

.. _zend.uri.uri_parts.accessing:

Accessing URI Parts
^^^^^^^^^^^^^^^^^^^
The following accessor methods are provided:

- `Uri->getScheme();`   // Get the scheme of the URI
- `Uri->getUserinfo();` // Get the user information part of the URI
- `Uri->getHost();`     // Get the host part of the URI
- `Uri->getPort();`     // Get the port part of the URI
- `Uri->getPath();`     // Get the path of the URI
- `Uri->getQuery();`    // Get the query part of the URI as a single string
- `Uri->getFragment();` // Get the fragment part of the URI

Each method may return `null` if the requested part is not defined. 

(CODE SAMPLE) 

In addition, you can get the query part of the URI split into key/value pairs 
(as is common, for example, in HTTP URI queries), using the `Uri->getQueryAsArray()` 
method: 

(CODE SAMPLE)

.. _zend.uri.uri_parts.modifying:

Modifying URI Parts
^^^^^^^^^^^^^^^^^^^
The following mutator methods are provided: 

- `Uri->setScheme();`   // Set the scheme of the URI
- `Uri->setUserinfo();` // Set the user information part of the URI
- `Uri->setHost();`     // Set the host part of the URI
- `Uri->setPort();`     // Set the port part of the URI
- `Uri->setPath();`     // Set the path of the URI
- `Uri->setQuery();`    // Set the query part of the URI as a single string
- `Uri->setFragment();` // Set the fragment part of the URI

All accessor functions return the URI object being manipulated, thus providing 
a fluent interface (AKA method chaining). For each of these functions, you may 
pass `null` as a value, effectively unsetting the URI part. 

(CODE SAMPLE) 

In addition, the setQuery() method will accept an associative array - in which 
case it will compose the query out of the key => value pairs in the array:

(CODE SAMPLE) 

.. _zend.uri.uri_parts.encoding:

A Note on URI Encoding
^^^^^^^^^^^^^^^^^^^^^^^
Zend\Uri\Uri attempts to be as permissive as possible in the input it accepts. 
When composing URI objects back into a string, it will make sure all parts that 
cannot be represented in their original form are properly encoded based on the
rules of the Generic URI Syntax. 

To avoid ambiguity, spaces in paths, queries and fragments are encoded as '%20' 
and not as a '+' symbol which is ambiguous and who's usage to represent the 
space character is deprecated.  

(CODE SAMPLE)

.. _zend.uri.validation:

URI Validation
--------------
When using ``Zend\Uri\UriFactory::factory()`` the given *URI* will always be 
validated and a ``Zend\Uri\Exception\InvalidArgumentException`` will be thrown
when the *URI* is invalid. However, after the ``Zend\Uri\UriInterface`` is 
instantiated for a new *URI* or an existing valid one, it is possible that the
*URI* can later become invalid after it is manipulated.

.. _zend.uri.instance-methods.valid.example-1:

.. rubric:: Validating a Zend\Uri\* Object

.. code-block:: php
   :linenos:

   $uri = Zend\Uri\UriFactory::factory('http://www.zend.com');

   $isValid = $uri->isValid();  // TRUE

The ``isValid()`` instance method provides a means to check that the *URI* 
object is still valid.


.. _zend.uri.normalization:

URI Normalization
-----------------


.. _zend.uri.resolution:

Resolving Relative URIs 
-----------------------


.. _zend.uri.helpermethods:

Useful URI-related Static Helper Methods
----------------------------------------


.. _`Uniform Resource Identifiers`: http://www.w3.org/Addressing/

.. [#] See http://www.ietf.org/rfc/rfc3986.txt for more information on URIs
