### Issue

*When should an element or type be declared global versus when should it be declared local?*



### Introduction

[Recall that a component (element, complexType, or simpleType) is "global" if it is an immediate child of <schema>, whereas it is "local" if it is not an immediate child of <schema>, i.e., it is nested within another component.]

What advice would you give to someone who was to ask you, "In general, when should an element (or type) be declared global versus when should it be declared local"? The purpose of this document is to provide answers to that question.

Below is a snippet of an XML instance document. We will explore the different design strategies using this example.

```
    <Book>        <Title>Illusions</Title>        <Author>Richard Bach</Author>    </Book> 
```



### Russian Doll Design

This design approach has the schema structure mirror the instance document structure, e.g., declare a Book element and within it declare a Title element followed by an Author element:



```
    <xsd:element name="Book">         <xsd:complexType>             <xsd:sequence>                 <xsd:element name="Title" type="xsd:string"/>                 <xsd:element name="Author" type="xsd:string"/>             </xsd:sequence>         </xsd:complexType>     </element> 
```

The instance document has all its components bundled together. Likewise, the schema is designed to bundle together all its element declarations.

This design represents one end of the design spectrum.



### Salami Slice Design

The Salami Slice design represents the other end of the design spectrum. With this design we disassemble the instance document into its individual components. In the schema we define each component (as an element declaration), and then assemble them together:



```
    <xsd:element name="Title" type="xsd:string"/>     <xsd:element name="Author" type="xsd:string"/>     <xsd:element name="Book">        <xsd:complexType>             <xsd:sequence>                 <xsd:element ref="Title"/>                 <xsd:element ref="Author"/>            </xsd:sequence>         </xsd:complexType>     </xsd:element> 
```

Note how the schema declared each component individually (Title, and Author) and then assembled them together (by ref'ing them) in the creation of the Book component.

These two designs represent opposite ends of the design spectrum.

To understand these designs it may help to think in terms of boxes, where a box represents an element or type:

- The Russian Doll design corresponds to having a single box, and it has nested within it boxes, which in turn have boxes nested within them, and so on. (boxes within boxes, just like a Russian Doll!)
- The Salami Slice design corresponds to having many separate boxes which are assembled together (separate boxes combined together, just like Salami slices brought together in a sandwich!)

Let's examine the characteristics of each of the two designs. (In so doing it will yield insights into another design.)



### Russian Doll Design Characteristics

[1] *Opaque content.* The content of Book is opaque to other schemas, and to other parts of the same schema. The impact of this is that none of the types or elements within Book are reusable.

[2] *Localized scope.* The region of the schema where the Title and Author element declarations are applicable is localized to within the Book element. The impact of this is that if the schema has set elementFormDefault="unqualified" then the namespaces of Title and Author are hidden (localized) within the schema.

[3] *Compact.* Everything is bundled together into a tidy, single unit.

[4] *Decoupled.* With this design approach each component is self-contained (i.e., they don't interact with other components). Consequently, changes to the components will have limited impact. For example, if the components within Book changes it will have a limited impact since they are not coupled to components outside of Book.

[5] *Cohesive.* With this design approach all the related data is grouped together into self-contained components, i.e., the components are cohesive.



### Salami Slice Design Characteristics

[1] *Transparent content.* The components which make up Book are visible to other schemas, and to other parts of the same schema. The impact of this is that the types and elements within Book are reusable.

[2] *Global scope.* All components have global scope. The impact of this is that, irrespective of the value of elementFormDefault, the namespaces of Title and Author will be exposed in instance documents.

[3] *Verbose.* Everything is laid out and clearly visible.

[4] *Coupled.* In our example we saw that the Book element depends on the Title and Author elements. If those elements were to change it would impact the Book element. Thus, this design produces a set of interconnected (coupled) components.

[5] *Cohesive.* With this design approach all the related data is also grouped together into self-contained components. Thus, the components are cohesive.

The two design approaches differ in a couple of important ways:

- The Russian Doll design facilitates hiding (localizing) namespace complexities. The Salami Slice design does not.
- The Salami Slice design facilitates component reuse. The Russian Doll design does not.

Is there a design which facilitates hiding (localizing) namespace complexities, **and** facilitates component reuse? Yes there is!

Consider the Book example again. An alternative design is to create a global type definition which nests the Title and Author element declarations within it:

```
    <xsd:complexType name="Publication">        <xsd:sequence>            <xsd:element name="Title" type="xsd:string"/>             <xsd:element name="Author" type="xsd:string"/>        </xsd:sequence>    </xsd:complexType>     <xsd:element name="Book" type="Publication"/> 
```

This design has both benefits:

- it is capable of hiding (localizing) the namespace complexity of Title and Author, and
- it has a reusable Publication type component.



### Venetian Blind Design

With this design approach we disassemble the problem into individual components, as the Salami Slice design does, but instead of creating element declarations, we create type definitions. Here's what our example looks like with this design approach:



​    <xsd:simpleType name="Title">         <xsd:restriction base="xsd:string">             <xsd:enumeration value="Mr."/>              <xsd:enumeration value="Mrs."/>              <xsd:enumeration value="Dr."/>         </xsd:restriction>     </xsd:simpleType>      <xsd:simpleType name="Name">         <xsd:restriction base="xsd:string">             <xsd:minLength value="1"/>          </xsd:restriction>     </xsd:simpleType>      <xsd:complexType name="Publication">         <xsd:sequence>             <xsd:element name="Title" type="Title"/>              <xsd:element name="Author" type="Name"/>         </xsd:sequence>     </xsd:complexType>      <xsd:element name="Book" type="Publication"/>

This design has:

- maximized reuse (there are four reusable components - the Title type, the Name type, the Publication type, and the Book element)
- maximized the potential to hide (localize) namespaces [note how this has been phrased: "maximized the *potential* ..." Whether, in fact, the namespaces of Title and Author are hidden or exposed, is determined by the elementFormDefault "switch"].

The Venetian Blind design espouses these guidelines ...

- Design your schema to maximize the *potential* for hiding (localizing) namespace complexities.
- Use elementFormDefault to act as a *switch* for controlling namespace exposure - if you want element namespaces exposed in instance documents, simply turn the elementFormDefault switch to "on" (i.e, set elementFormDefault= "qualified"); if you don't want element namespaces exposed in instance documents, simply turn the elementFormDefault switch to "off" (i.e., set elementFormDefault="unqualified").
- Design your schema to maximize reuse.
- Use type definitions as the main form of component reuse.
- Nest element declarations within type definitions.

Let's compare the Venetian Blind design with the Salami Slice design. Recall our example:

**Salami Slice Design:**

`    <xsd:element name="Title" type="xsd:string"/>     <xsd:element name="Author" type="xsd:string"/>     <xsd:element name="Book">        <xsd:complexType>             <xsd:sequence>                 <xsd:element ref="Title"/>                 <xsd:element ref="Author" />            </xsd:sequence>         </xsd:complexType>     </xsd:element> `

The Salami Slice design also results in creating reusable (element) components, but it has absolutely no potential for namespace hiding.

"*However*", you argue, "*Suppose that I want namespaces exposed in instance documents.* [We have seen cases where this is desired.] *So the Salami Slice design is a good approach for me. Right?*"

Let's think about this for a moment. What if at a later date you change your mind and wish to hide namespaces (what if your users hate seeing all those namespace qualifiers in instance documents)? You will need to redesign your schema (possibly scraping it and starting over).

Better to adopt the Venetian Blind Design, which allows you to control whether namespaces are hidden or exposed by simply setting the value of elementFormDefault. No redesign of your schema is needed as you switch from exposing to hiding, or vice versa.

[That said ... your particular project may need to sacrifice the ability to turn on/off namespace exposure because you require instance documents to be able to use element substitution. In such circumstances the Salami Slice design approach is the only viable alternative.]

Here are the characteristics of the Venetian Blind Design.



### Venetian Blind Design Characteristics:

[1] *Maximum reuse.* The primary component of reuse are type definitions.

[2] *Maximum namespace hiding.* Element declarations are nested within types, thus maximizing the potential for namespace hiding.

[3] *Easy exposure switching.* Whether namespaces are hidden (localized) in the schema or exposed in instance documents is controlled by the elementFormDefault switch.

[4] *Coupled.* This design generates a set of components which are interconnected (i.e., dependent).

[5] *Cohesive.* As with the other designs, the components group together related data. Thus, the components are cohesive.



### Best Practice

[1] The Venetian Blind design is the one to choose where your schemas require the flexibility to turn namespace exposure on or off with a simple switch, and where component reuse is important.

[2] Where your task requires that you make available to instance document authors the option to use element substitution, then use the Salami Slice design.

[3] Where mimimizing size and coupling of components is of utmost concern then use the Russian Doll design.



### Complete Example

This example demonstrates the three designs. Consider an instance document containing two components that are associated to each other by ID/IDREF attributes. Here's the first component:



`    <Person id="rbach>        <Name>Richard Bach</Name>    </Person> `

The Person element contains an id attribute to uniquely identify it.

Now that we have a Person component it can be referenced by the Book component:



`    <Book>        <Title>Illusions</Title>        <Author idref="rbach"/>    </Book> `

"Book is comprised of a Title followed by an Author. The Author is a Person whose name is Richard Bach."

In this design there are two main components ("objects"), Person and Book. The Author element bridges (associates) the two components. [One could imagine an O-O Object model of Person and Book mapping to these two XML components.]

Let's see how each of the three design approaches would model the Person and Book components.



#### Russian Doll Design

This design says to keep things compact and simple by inlining element declarations. Here is how Person and Book are be represented using this design:



`   <xsd:element name="Person">      <xsd:complexType>         <xsd:sequence>            <xsd:element name="Name" type="xsd:string"/>         </xsd:sequence>         <xsd:attribute name="id" type="xsd:ID" use="required"/>      </xsd:complexType>   </xsd:element>    <xsd:element name="Book">      <xsd:complexType>         <xsd:sequence>            <xsd:element name="Title" type="xsd:string"/>            <xsd:element name="Author">               <xsd:complexType>                  <xsd:attribute name="idref" type="xsd:IDREF"                                  use="required"/>               </xsd:complexType>            </xsd:element>         </xsd:sequence>      </xsd:complexType>   </xsd:element> `

As is characteristic of the Russian Doll design, all the elements and attributes are nested together in a tidy, compact bundle.



#### Salami Slice Design:

With this design all the components are "spread out", declared individually, and then aggregated back together. Here's how Person and Book are represented using this design:



   <xsd:element name="Name" type="xsd:string"/>     <xsd:attribute name="id" type="xsd:ID"/>     <xsd:element name="Person">       <xsd:complexType>          <xsd:sequence>             <xsd:element ref="Name"/>          </xsd:sequence>          <xsd:attribute ref="id" use="required"/>       </xsd:complexType>    </xsd:element>     <xsd:element name="Title" type="xsd:string"/>     <xsd:attribute name="idref" type="xsd:IDREF"/>     <xsd:element name="Author">       <xsd:complexType>          <xsd:attribute ref="idref" use="required"/>       </xsd:complexType>    </xsd:element>     <xsd:element name="Book">       <xsd:complexType>          <xsd:sequence>             <xsd:element ref="Title"/>             <xsd:element ref="Author"/>          </xsd:sequence>       </xsd:complexType>    </xsd:element>

The notable characteristic of this design is that all the elements and attributes are global. Consequently, their namespaces will always be exposed in instance documents, irrespective of the value of the elementFormDefault "switch".



#### Venetian Blind Design

With this approach we strive for the reuse that the Salami Slice design provides, plus the ability to toggle namespace exposure/hiding that the Russian Doll design provides. This is accomplished by "spreading out" components as types, and nesting element declarations within the types. Here's how Person and Book is represented using this design:



   <xsd:complexType name="Person">       <xsd:sequence>          <xsd:element name="Name" type="xsd:string"/>       </xsd:sequence>       <xsd:attribute name="id" type="xsd:ID" use="required"/>    </xsd:complexType>     <xsd:complexType name="Author">       <xsd:attribute name="idref" type="xsd:IDREF" use="required"/>    </xsd:complexType>     <xsd:complexType name="Book">       <xsd:sequence>          <xsd:element name="Title" type="xsd:string"/>          <xsd:element name="Author" type="Author"/>       </xsd:sequence>    </xsd:complexType>     <xsd:element name="Person" type="PersonType"/>     <xsd:element name="Book" type="BookType"/>

Note that the elements are nested within the types. This enables us to toggle on/off namespace exposure in instance documents.

The Russian Doll and Venetian Blind designs dramatically distinguish themselves from the Salami Slice design in the instance documents. The instance documents corresponding to the Russian Doll and Venetian Blind designs are simple and unencumbered by namespace qualifiers. Above we assumed that the Person and Book elements were declared in the same namespace as a Catalog element. Let's suppose that they were defined in a separate namespace, and then <import>ed into the catalog schema. Here's a representative instance document for the Russian Doll and Venetian Blind designs:



`<cat:Catalog xmlns:cat="http://www.catalog.org"             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"             xsi:schemaLocation=                        "http://www.catalog.org                         Catalog.xsd">   <Person id="rbach">      <Name>Richard Bach</Name>   </Person>   <Book>      <Title>Illusions</Title>      <Author idref="rbach"/>   </Book> </cat:Catalog> `

Where the Catalog schema got the Person and Book elements is totally transparent to the instance document. That namespace knowledge is hidden within the schema.

On the other hand, the instance document for the Salami Slice design is filled with many namespace qualifiers:



`<cat:Catalog xmlns:cat="http://www.catalog.org"             xmlns:a="http://www.person-book.org"             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"             xsi:schemaLocation=                        "http://www.catalog.org                         Catalog.xsd">   <a:Person a:id="rbach">      <a:Name>Richard Bach</a:Name>   </a:Person>   <a:Book>      <a:Title>Illusions</a:Title>      <a:Author a:idref="rbach"/>   </a:Book> </cat:Catalog> `

Where the Catalog schema obtained the Person and Book elements is visibly exposed in the instance document. And this is true irrespective of the value of elementFormDefault. This is, of course, due to the fact that this design declares all the elements globally. This is a really good example to demonstrate the inability of the Salami Slice design to hide namespaces.



### Impact of Coupling and Cohesion on the Global vs Local Decision

The programming world has long employed these two design principles:

- minimize dependencies between programs (i.e., minimize *coupling* between programs)
- group together related pieces of information (i.e., maximize *cohesiveness* of programs).

These principles help to create programs which can be independently tested and reasoned upon, and help to minimize change in one program from impacting another program.

What do these principles have to do with XML and Schema design? To see, let's recast the two principles into data modeling terms:

- minimize dependencies between elements, both in the instance document as well as in the schema (i.e., minimize "coupling" between elements)
- group together related pieces of data (i.e., maximize "cohesiveness" of data).

Thus these principles are also desirable features of XML documents and schema documents.

Let's demonstrate applicability of the principles to XML data modeling with an example. Consider this snippet of XML:

`   <Book>      <Title>Illusions</Title>      <Author>Richard Bach</Author>      <Cost>12.85</Cost>   </Book>    <Person>      <Name>Richard Bach</Name>   </Person> `

Here we see some data, structured using XML syntax. If we think of Book and Person as components, then these components are very loosely coupled (in fact, not coupled at all), and the related data is nicely bundled together in the appropriate component (i.e., high cohesiveness).

This instance document exhibits minimal coupling and maximal cohesiveness of its components. This instance document is well designed with regards to the coupling/cohesion properties.

Schemas are also XML documents. Thus we should be able to analyze a schema to see if its components are decoupled and cohesive. Let's analyze the Russian Doll and the Venetian Blind designs to see how coupled/cohesive are the components that these schema designs produce.

Here's the Schema for the above Book and Person elements using the Russian Doll Design:

`   <xsd:element name="Book">      <xsd:complexType>         <xsd:sequence>            <xsd:element name="Title" type="xsd:string"/>            <xsd:element name="Author" type="xsd:string"/>            <xsd:element name="Cost">                <xsd:simpleType>                   <xsd:restriction base="xsd:decimal">                      <xsd:scale value="2"/>                   </xsd:restriction>                </xsd:simpleType>             </xsd:element>         </xsd:sequence>      </xsd:complexType>   </xsd:element>    <xsd:element name="Person">      <xsd:complexType>         <xsd:sequence>            <xsd:element name="Name" type="xsd:string"/>         </xsd:sequence>      </xsd:complexType>   </xsd:element> `

Recall that the Russian Doll design mirrors the instance document structure; thus in this schema we see two self-contained element declarations, which mirror the two self-contained elements in the instance document.

Here's what the schema looks like using the Venetian Blind Design:

   <xsd:simpleType name="money">       <xsd:restriction base="xsd:decimal">          <xsd:scale value="2"/>       </xsd:restriction>    </xsd:simpleType>     <xsd:complexType name="BookType">       <xsd:sequence>          <xsd:element name="Title" type="xsd:string"/>          <xsd:element name="Author" type="xsd:string"/>          <xsd:element name="Cost" type="money"/>       </xsd:sequence>    </xsd:complexType>     <xsd:complexType name="PersonType">       <xsd:sequence>          <xsd:element name="Name" type="xsd:string"/>       </xsd:sequence>    </xsd:complexType>     <xsd:element name="Book" type="BookType"/>    <xsd:element name="Person" type="PersonType"/>

Recall that the Venetian Blind design spreads out the components into type definitions and then reuses the types.

The instance document we saw earlier is valid for either of these schema designs. Thus both designs enable us to create instance documents which exhibit minimal coupling and maximal cohesiveness.

How do the two schema designs measure up with respect to coupling and cohesiveness of the components they create?

- Both schema designs bundled together Title and Author into a Book component. Both bundled Name into a Person component. The Venetian Blind has an additional component - money. The money component bundled together the features of (U.S.) money. Both designs do a good job of bringing together related data. Thus, both schemas exhibit a high degree of cohesiveness.
- The Russian Doll design creates standalone Book and Person components. The Venetian Blind design creates a BookType which depends upon the money type. The Book element depends upon the BookType. The Person element depends upon the PersonType. Thus, the Venetian Blind schema has a interdependent set of components. That is, it is a more coupled design.

How do the two designs measure up in terms of reusable components:

- The schema that was designed using the Russian Doll design has just two big elements - Book and Person. Elements are by nature not very reusable.
- The schema that was designed using the Venetian Blind design has three highly reusable components - money, BookType, and PersonType. As well, it has the Book and Person elements.

Here's a table which generalizes what we have seen in our example:



| Principle           | Russian Doll components | Venetian Blind components |
| ------------------- | ----------------------- | ------------------------- |
| Cohesion            | High                    | High                      |
| Coupling            | Low                     | High                      |
| Reusable Components | Low                     | High                      |



In general, the Russian Doll design results in components that are highly cohesive, with minimal coupling, but with few reusable components. The Venetian Blind design results in components that are also highly cohesive, with many reusable components, but the components have high coupling.



------



### Design Names

"**Russian Doll**" captures the idea of nested containment vividly. Also, in many cases, there needs to be isomorphism between the containing and the contained (lists; headings), which this metaphor captures as well.

"**Salami Slice**" captures both the disassembly process, the resulting flat look of the schema, and implies reassembly as well (into a sandwich).

"**Venetian Blind**" captures the ability to expose or hide namespaces with a simple switch, and the assembly of slats ca