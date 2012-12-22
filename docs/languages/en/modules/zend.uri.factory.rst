.. _zend.uri.factory:

The URI Object Factory
======================

Creating and Registering Custom URI Classes 
-------------------------------------------

You can specify a custom class to be used when using the ``Zend\Uri\UriFactory``
by registering your class with the Factory using 
``\Zend\Uri\UriFactory::registerScheme()`` which takes the scheme as first 
parameter. This enables you to create your own *URI*-class and instantiate new 
URI objects based on your own custom classes.

The 2nd parameter passed to ``Zend\Uri\UriFactory::registerScheme()`` must be a
string with the name of a class implementing ``Zend\Uri\UriInterface``. The 
class must either be alredy loaded, or be loadable by the autoloader. 

.. _zend.uri.creation.custom.example-1:

.. rubric:: Creating a URI using a custom class

.. code-block:: php
   :linenos:

   // Create a new 'ftp' URI based on a custom class
   use Zend\Uri\UriFactory

   UriFactory::registerScheme('ftp', 'MyNamespace\MyClass');

   $ftpUri = UriFactory::factory(
       'ftp://user@ftp.example.com/path/file'
   );

   // $ftpUri is an instance of MyLibrary\MyClass, which implements 
   // Zend\Uri\UriInterface

.. _zend.uri.manipulation:

Manipulating an Existing URI
----------------------------

To manipulate an existing *URI*, pass the entire *URI* as string to 
``Zend\Uri\UriFactory::factory()``.

.. _zend.uri.manipulation.example-1:

.. rubric:: Manipulating an Existing URI with Zend\\Uri\\UriFactory::factory()

.. code-block:: php
   :linenos:

   // To manipulate an existing URI, pass it in.
   $uri = Zend\Uri\UriFactory::factory('http://www.zend.com');

   // $uri instanceof Zend\Uri\UriInterface

The *URI* will be parsed and validated. If it is found to be invalid, a 
``Zend\Uri\Exception\InvalidArgumentException`` will be thrown immediately. 
Otherwise, ``Zend\Uri\UriFactory::factory()`` will return a class implementing
``Zend\Uri\UriInterface`` that specializes in the scheme to be manipulated.


