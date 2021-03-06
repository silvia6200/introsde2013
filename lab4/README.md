class: center, middle, inverse

# XML schemas, Java Annotations, JAXB & Dozer 
Introduction to Service Design and Engineering 2013/2014. 
<br>*Lab session #4*
<br>**University of Trento** 

---

## Outline

* XML schema definition (XSD) overview
* Java Annotation minimal example
* Introduction to JAXB
* Example: from schema to java representations
* Example: Generate an XML document from an Object Model
* Exercise
* Dozer
* Assignment


---

## XSD: XML Schema Definition (1)

* An XML schema describes the structure of an XML document
* An XML Schema is written in XML
* It is an XML‐based alternative to DTD (document type definition - which is yet another set of markups to learn)
* XML Schema is a W3C Recommendation: http://www.w3.org/2001/XMLSchema

---

## XSD: XML Schema Definition (1)

* An XML Schema defines:
	* **elements** that can appear in a document
	* **attributes** that can appear in a document
	* **data types** for elements and attributes
	* which elements are **child** elements
	* the **order** of child elements
	* whether an element is **empty** or can include **text**
	* **default and fixed values** for elements and attributes

---

## Example 1: XSD (1)

* Open the [Example 1](https://github.com/cdparra/introsde2013/blob/master/lab4/Example1.xsd)
```xml 
<xsd:schema 
    xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    <xsd:complexType name="personType">
        <xsd:sequence>
            <xsd:element name="firstName" type="xsd:string"				/>
            <xsd:element name="lastName"  type="xsd:string"				/>
            <xsd:element name="birthDate" type="xsd:date"				/>
            <xsd:element name="age"       type="xsd:integer"			
                minOccurs="0" maxOccurs="1"/>
            <xsd:element name="healthProfile" type="healthProfileType"	
                minOccurs="0" maxOccurs="1"/>
        </xsd:sequence>
		<xsd:attribute name="id" type="xsd:integer"/>
    </xsd:complexType>
    <xsd:complexType name="healthProfileType">
        <xsd:sequence>
            <xsd:element name="weight" type="xsd:decimal"/>
            <xsd:element name="height"  type="xsd:decimal"/>
         </xsd:sequence>
    </xsd:complexType>
    <xsd:element name="person" type="personType"/>
</xsd:schema>
```

---


## Example 1: XSD (2)

```xml 
	<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema">
    	...
   </xsd:schema>
```

* **xmlns:xsd="…"** indicates that the elements and data types used in this schema 
	* come from the *http://www.w3.org/2001/XMLSchema* namespace
	* should be prefixed with xsd:

---


## XML Schema built-in data types

* The most common built-in data types are:
	* xsd:string
	* xsd:decimal
	* xsd:integer
	* xsd:boolean
	* xsd:date
	* xsd:time
* The complete built-in data type hierarchy
	* http://www.w3.org/TR/xmlschema-2/#built-in-datatypes

---


## Example 1: XSD (3)

* XML Elements
```xml 
	<firstName>George R. R.</firstName>
	<lastName>Martin</lastName>
	<birthDate>1970-06-21</birthDate>
```

* Corresponding XSD definitions
```xml
	<xsd:element name="firstName" type="xsd:string" />
	<xsd:element name="lastName"  type="xsd:string" />
    <xsd:element name="birthDate" type="xsd:date"   />
```

---

## Complex Data Types

* A complex element is an XML element that contains other elements and/or attributes. 
* There are four kinds of complex elements:
	* empty elements
	* elements that contain only other elements
	* elements that contain only text (and attributes)
	* elements that contain both other elements and text
* For each kind, there are many ways to write it in a XSD document. We see only one way, that is compatible with JAXB

---

## Example 1: XSD (4)

### Empty Elements

* XML Elements
```xml 
	<person id="12345">
	…
	</person>
```

* Corresponding XSD definitions
```xml 
	…
	<xsd:element name="person" type="personType"/>
    <xsd:complexType name="personType">c
    </xsd:complexType>
```

---


## Example 1: XSD (5)

### Elements that contain only Elements 

* XML Elements
```xml 
	<person>
		<firstName>George R. R.</firstName>
		<lastName>Martin</lastName>
		<birthDate>1970-06-21</birthDate>
	</person>
```

* Corresponding XSD definitions
```xml 
	<xsd:element name="person" type="personType"/>
    <xsd:complexType name="personType">
        <xsd:sequence>
            <xsd:element name="firstName" type="xsd:string"/>
            <xsd:element name="lastName"  type="xsd:string"/>
            <xsd:element name="birthDate" type="xsd:date"/>
         </xsd:sequence>
    </xsd:complexType>
```

---


## Example 2: XSD (6)

### Elements that contain only Text and Attributes 

* XML Elements
```xml 
	<shoesize country="france">35</shoesize>
```

* Corresponding XSD definitions
```xml 
	<xsd:element name="shoesize" type="shoesizeType"/>
    <xsd:complexType name="shoesizeType">
        <xsd:simpleContent>
        	<xsd:extension base="xsd:integer">
        		<xsd:attribute name="country" type="xsd:string"/>
        	</xsd:extension>
         </xsd:simpleContent>
    </xsd:complexType>
```

---

## Example 3: XSD (7)

### Elements that contain both elements and text 

* XML Elements
```html 
	<p> 
		<b>Mixed content</b> lets you embed <i>child elements</i>
	</p>
```

* Corresponding XSD definitions
```xml 
    <xsd:complexType name="chunkType" mixed="true">
        <xsd:choice maxOccurs="unbounded">
        	<xsd:element name="b" type="chunkType"/>
        	<xsd:element name="i" type="chunkType"/>
        </xsd:choice>
    </xsd:complexType>
    <xsd:complexType name="TextType" >
        <xsd:sequence>
        	<xsd:element name="p" type="chunkType"/>
        </xsd:sequence>
    </xsd:complexType>
```

* **What was new here?**

---

## XSD Indicators

* **Order indicators** are used to define the order of the elements
	* all, choice, sequence
* **Occurrence indicators** are used to define how often an element can occur
	* maxOccurs, minOccurs
* **Group indicators** are used to define related sets of elements.
	* group name, attributeGroup name


---

## Example 4: XSD (8)

### Group names

```xml 
    <xsd:group name="persongroup">
        <xsd:sequence>
            <xsd:element name="firstName" type="xsd:string"/>
            <xsd:element name="lastName"  type="xsd:string"/>
            <xsd:element name="birthDate" type="xsd:date"/>
         </xsd:sequence>
    </xsd:group>
    <xsd:element name="person" type="personinfo"/>
    <xsd:complexType name="personinfo" >
        <xsd:sequence>
        	<xsd:group ref="persongroup"/>
            <xsd:element name="country" type="xsd:string"/>
        </xsd:sequence>
    </xsd:complexType>
```

---


## Example 5: XSD (9)

### AttributeGroup name

```xml 
    <xsd:attributeGroup name="elementAttrGroup">
        <xsd:attribute name="refURI" type="xsd:anyURI" use="optional">
        </xsd:attribute>
        <xsd:attribute name="id" type="xsd:integer">
        </xsd:attribute>
    </xsd:attributeGroup>
    <xsd:complexType name="SomeEntity" >
        <xsd:simpleContent>
            <xsd:extension base="xsd:string">
                <xsd:attributeGroup ref="elementAttrGroup"/>
            </xsd:extension>
         </xsd:simpleContent>
    </xsd:complexType>
```

---

## Example 6: XSD (10)

### Sustitution

```xml 
<xs:element name="name" type="xs:string"/>
<xs:element name="navn" substitutionGroup="name"/>
```

```xml
<xs:element name="name" type="xs:string"/>
<xs:element name="navn" substitutionGroup="name"/>
<xs:complexType name="custinfo">
  <xs:sequence>
    <xs:element ref="name"/>
  </xs:sequence>
</xs:complexType>
<xs:element name="customer" type="custinfo"/>
<xs:element name="kunde" substitutionGroup="customer"/>
```

* Valid XMLs

```xml
<customer>
  <name>John Smith</name>
</customer>
```

or

```xml
<kunde>
  <navn>John Smith</navn>
</kunde>
```

---

## Java Annotations


* Annotations provide data about a program that is not part of the program itself.
* They have no direct effect on the operation of the code they annotate.
* Annotations can be applied to a program's  declarations of classes, fields, methods, and other program  elements.


```java
@XmlRootElement // this is a java annotation
@XmlType(name = "", propOrder = { "publisher", "edition", "title", "author" })
public class Catalog {
	private String publisher;
	private String edition;
	private String title;
	private String author;
	...
	@XmlAttribute
	public String journal;
```

---

## Java Annotations: Uses

* **Information for the compiler:** Annotations can be used by the compiler to detect errors or suppress warnings.
* **Compile-time and deployment-time processing:** Software tools can process annotation information to generate code, XML files, and so forth.
* **Runtime processing:** Some annotations are available to be examined at runtime

---


## Introduction to JAXB

* JAXB = **J**ava **A**rchitecture for **X**ML **B**inding
* Java standard that defines how Java objects are converted **from** and **to** XML. 
* As opposed to XPATH, now we can map XML to a set of Java Classes and restrict our Java program to java objects (not a document tree)
* JAXB Provides two main features: 
	* the ability to **marshal** (i.e., convert) Java objects into XML
	* the ability to **un-marshal** XML back into Java objects
* https://jaxb.java.net/

---

## JAXB Architecture

![](https://raw.github.com/cdparra/introsde2013/master/lab4/resources/JAXB-Architecture.png)


---

## JAXB Binding Process

![](https://raw.github.com/cdparra/introsde2013/master/lab4/resources/JAXB-BindingProcess.png)

---

## JAXB Annotations

* **@XmlRootElement(namespace = "namespace"):** defines the root element for an XML tree
* **@XmlType(propOrder = { "field2", "field1",.. }):** allows to define the order in which the fields are written in the XML file
* **@XmlElement(name = "neuName"):** defines the XML element which will be used. Only need to be used if the neuNeu is different then the JavaBeans Name

---

## Example 6: JAXB Annotations

```java
@XmlRootElement(name = "book")
// If you want you can define the order in which the fields are written
// Optional
@XmlType(propOrder = { "author", "name", "publisher", "isbn" })
public class Book {
  //
  private String name;
  private String author;
  private String publisher;
  private String isbn;
  //
  // If you like the variable name, e.g. "name", you can easily change this
  // name for your XML-Output:
  @XmlElement(name = "title")
  public String getName() {
    return name;
  }
```

---

## Example 6: JAXB Annotations

```java
//This statement means that class "Bookstore.java" is the root-element of our example
@XmlRootElement(namespace = "de.vogella.xml.jaxb.model")
public class Bookstore {
  //
  // XmLElementWrapper generates a wrapper element around XML representation
  @XmlElementWrapper(name = "bookList")
  // XmlElement sets the name of the entities
  @XmlElement(name = "book")
  private ArrayList<Book> bookList;
  private String name;
  private String location;
```

---

## Example 6: XML from/to Object Model 

```java
public class BookMain {
  private static final String BOOKSTORE_XML = "./bookstore-jaxb.xml";
  public static void main(String[] args) throws JAXBException, IOException {
    ArrayList<Book> bookList = new ArrayList<Book>();
    // create books
    Book book1 = new Book();
    book1.setIsbn("978-0060554736");
    book1.setName("The Game");
    book1.setAuthor("Neil Strauss");
    book1.setPublisher("Harpercollins");
    bookList.add(book1);
    //
    Book book2 = new Book();
    book2.setIsbn("978-3832180577");
    book2.setName("Feuchtgebiete");
    book2.setAuthor("Charlotte Roche");
    book2.setPublisher("Dumont Buchverlag");
    bookList.add(book2);
    //
    // create bookstore, assigning book
    Bookstore bookstore = new Bookstore();
    bookstore.setName("Fraport Bookstore");
    bookstore.setLocation("Frankfurt Airport");
    bookstore.setBookList(bookList);
	…
```

---

## Example 6: XML from/to Object Model 

```java
	...
    // create JAXB context and instantiate marshaller
    JAXBContext context = JAXBContext.newInstance(Bookstore.class);
    Marshaller m = context.createMarshaller();
    m.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, Boolean.TRUE);
	//
    // Write to System.out
    m.marshal(bookstore, System.out);
	//
    // Write to File
    m.marshal(bookstore, new File(BOOKSTORE_XML));
	//
    // get variables from our xml file, created before
    System.out.println();
    System.out.println("Output from our XML File: ");
   	…
```

---

## Example 6: XML from/to Object Model 

```java
	...
    Unmarshaller um = context.createUnmarshaller();
    Bookstore bookstore2 = (Bookstore) um.unmarshal(new FileReader(BOOKSTORE_XML));
    ArrayList<Book> list = bookstore2.getBooksList();
    for (Book book : list) {
      System.out.println("Book: " + book.getName() + " from "
          + book.getAuthor());
    }
  }
} 
```

---

## Exercise 1: XML from Object Model



---

## Assignment #1

* Replace the HashMap db in the HealthProfile Reader with a xml file as follows

```xml
<people>
    <person>
        <firstname>George R. R.</firstname>
        <lastname>Martin</lastname>
        <healthprofile>
            <weitgh>120</weitgh>
            <height>1.65</height>
        </healthprofile>
    </person>
    <!-- add more people to the db --> 
</people>
```

* Use xpath to implement methods like getWeight and getHeight
* Make a function which prints all people in the list with detail
* A function which accepts name as parameter and print that particular person HealthProfile
* A function which accepts weight and an operator (=, > , <) as parameters and prints people that fulfill that condition.


## References:

* XML Schemas
	* http://www.w3schools.com/schema/default.asp
	* http://www.xfront.com/files/xml-schema.html
	* Validator: http://tools.decisionsoft.com/schemaValidate/
* JAXB
* Dozer