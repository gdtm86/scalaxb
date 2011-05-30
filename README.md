scalaxb
=======

scalaxb is an XML data-binding tool for Scala that supports W3C XML 
Schema (xsd) as the input file.

Status
------

This is still at ALPHA state, and many things may not work.
I'd really appreciate if you could run it against your favorite xsd
file and let me know the result.

sbt-scalaxb
-----------

To call `compile-xsd` from sbt, put this in your Plugins.scala:

    import sbt._

    class Plugins(info: ProjectInfo) extends PluginDefinition(info) {
      val scalaxb = "org.scalaxb" % "sbt-scalaxb" % "0.6.0"
      
      val scalaToolsNexusSnapshots = "Scala Tools Nexus Snapshots" at "http://nexus.scala-tools.org/content/repositories/snapshots/"
      val scalaToolsNexusReleases  = "Scala Tools Nexus Releases" at "http://nexus.scala-tools.org/content/repositories/releases/"
    }

Installation
------------

scalaxb is tested only under Scala 2.8.1 and 2.9.0-1, but the sbaz distribution is only available for 2.9.0-1:

    $ sudo sbaz update
    $ sudo sbaz install scalaxb

and upgrade it if you've installed it before:

    $ sudo sbaz update
    $ sudo sbaz upgrade

or build from source:

    $ git clone git://github.com/eed3si9n/scalaxb.git scalaxb
    $ cd scalaxb
    $ sbt "project scalaxb" sbaz-pack

See the file called INSTALL for details.

Usage
-----

    $ scalaxb [options] <schema_file>...

      -d <directory> | --outdir <directory>
            generated files will go into <directory>
      -p <package> | --package <package>
            specifies the target package
      -p:<namespaceURI>=<package> | --package:<namespaceURI>=<package>
            specifies the target package for <namespaceURI>
      --class-prefix <prefix>
            prefixes generated class names
      --param-prefix <prefix>
            prefixes generated parameter names
      --wrap-contents <complexType>
            wraps inner contents into a seperate case class
      -v | --verbose
            be extra verbose
      <schema_file>
            input schema to be converted

Documents
---------

Further info is available at [scalaxb.org](http://scalaxb.org/).

Example
-------

Suppose you have address.xsd:

    <schema targetNamespace="http://www.example.com/IPO"
            xmlns="http://www.w3.org/2001/XMLSchema"
            xmlns:ipo="http://www.example.com/IPO">
      <complexType name="Address">
        <sequence>
          <element name="name"   type="string"/>
          <element name="street" type="string"/>
          <element name="city"   type="string"/>
        </sequence>
      </complexType>
    </schema>

You then run the following:

    $ scalaxb address.xsd -p ipo
    
You should see output like:

    generated ./address.scala.
    generated ./xmlprotocol.scala.
    generated ./scalaxb.scala.
    
address.scala contains a case class representing the complex type:

    // Generated by <a href="http://scalaxb.org/">scalaxb</a>.
    package ipo

    case class Address(name: String,
      street: String,
      city: String)
      
xmlprotocol.scala contains `XMLProtocol` trait, which defines typeclass instances
to convert XML documents into `Address`, and `Address` back to XML.
It also defines the package object for `ipo` package, which inherits `XMLProtocol`.

    // Generated by <a href="http://scalaxb.org/">scalaxb</a>.
    package ipo

    /**
    usage:
    import scalaxb._
    import ipo._
    
    val obj = fromXML[Foo](node)
    val document = toXML[Foo](obj, "foo", defaultScope)
    **/
    object `package` extends XMLProtocol { }

    trait XMLProtocol extends scalaxb.XMLStandardTypes {
      val defaultScope = scalaxb.toScope(Some("unq") -> "http://www.example.com/IPO",
        Some("ipo") -> "http://www.example.com/IPO",
        Some("xsi") -> "http://www.w3.org/2001/XMLSchema-instance")
      implicit lazy val IpoAddressFormat: scalaxb.XMLFormat[ipo.Address] = new DefaultIpoAddressFormat {}

      trait DefaultIpoAddressFormat extends scalaxb.ElemNameParser[ipo.Address] {
        val targetNamespace: Option[String] = Some("http://www.example.com/IPO")

        override def typeName: Option[String] = Some("Address")

        def parser(node: scala.xml.Node, stack: List[scalaxb.ElemName]): Parser[ipo.Address] =
          (scalaxb.ElemName(None, "name")) ~ 
          (scalaxb.ElemName(None, "street")) ~ 
          (scalaxb.ElemName(None, "city")) ^^
          { case p1 ~ p2 ~ p3 =>
          ipo.Address(scalaxb.fromXML[String](p1, scalaxb.ElemName(node) :: stack),
            scalaxb.fromXML[String](p2, scalaxb.ElemName(node) :: stack),
            scalaxb.fromXML[String](p3, scalaxb.ElemName(node) :: stack)) }

        def writesChildNodes(__obj: ipo.Address, __scope: scala.xml.NamespaceBinding): Seq[scala.xml.Node] =
          Seq.concat(scalaxb.toXML[String](__obj.name, None, Some("name"), __scope, false),
            scalaxb.toXML[String](__obj.street, None, Some("street"), __scope, false),
            scalaxb.toXML[String](__obj.city, None, Some("city"), __scope, false))
      }
    }

Finally, scalaxb.scala contains some helper classes to use the generated code.

How to use the generated code
-----------------------------

In your program, include all three of the generated files.
To convert an XML node into a case class object, call `scalaxb.fromXML[A]`:

    val shipTo = scalaxb.fromXML[ipo.Address](<shipTo>
        <name>foo</name>
        <street>1537 Paper Street</street>
        <city>Wilmington</city>
      </shipTo>)

In the above code, Scala compiler supplies `XMLFormat[ipo.Address]` into the implicit parameter off of the package object.
It's actually `ipo.DefaultIpoAddressFormat.` Now you can write `shipTo.name` to retrieve the name value.

To convert the object back to XML, call `scalaxb.toXML[A]`:

    val document = scalaxb.toXML[ipo.Address](shipTo, "shipTo", ipo.defaultScope)

Bug Reporting
-------------

You can send bug reports to [Issues](http://github.com/eed3si9n/scalaxb/issues),
send me a [tweet to @eed3si9n](http://twitter.com/eed3si9n), or email.

Licensing
---------

It's the MIT License. See the file called LICENSE.
     
Contacts
--------

- eed3si9n at gmail dot com
- [@eed3si9n](http://twitter.com/eed3si9n)
