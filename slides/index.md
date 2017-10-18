# ATL 

AtlanMod Transformation Language

_AtlanMod team_

_IMT Atlantique, LS2N_

<atlanmod-contact@imt-atlantique.fr>


----

## Context of this work 

The present courseware has been elaborated in the context of the MODELWARE European  IST FP6 project (http://www.modelware-ist.org/).

Co-funded by the European Commission, the MODELWARE project involves 19 partners from 8 European countries. MODELWARE aims to improve software productivity by capitalizing on techniques known as Model-Driven Development (MDD).

To achieve the goal of large-scale adoption of these MDD techniques, MODELWARE promotes the idea of a collaborative development of courseware dedicated to this domain. 

The MDD courseware provided here with the status of open source software is produced under the EPL 1.0 license. 

----

## Prerequisites

- To be able to understand this lecture, a reader should be familiar with the following concepts, languages, and standards:

  - Model Driven Engineering (MDE)
  - The role of model transformations in MDE
  - UML
  - OCL
  - MOF
  - Basic programming concepts


----

## Contents

- Introduction
- Description of ATL
- Example: Class to Relational
- Additional considerations
- Conclusion

----

## Contents

- **Introduction**
  - Definitions
  - Operational context
- Description of ATL
- Example: Class to Relational
- Additional considerations
- Conclusion

----

## Definitions

<div style="position:absolute; left:0px;  width:600px;font-size: 24pt;"  align="left">
- A model transformation is the automatic creation of target models from source models.

- Model transformation is not only about M1 to M1 transformations:

  - M1 to M2: promotion,
  - M2 to M1: demotion,
  - M3 to M1, M3 to M2, etc.
</div>
<p style="position:absolute;  right:0px; top:300px; width:300px;" align="center">
![](resources/three-levels.pdf)
</p>


----

## Operational Context: Small Theory

<img src="resources/operational-context.pdf" alt="Drawing" style="width: 600px;"/>

----

## Operational Context of ATL

<img src="resources/atl-context.pdf" alt="Drawing" style="width: 600px;"/>

----

## Contents

- Introduction
- **Description of ATL**
  - Overview
  - Source pattern
  - Target pattern
  - Execution order
- Example: Class to Relational
- Additional considerations
- Conclusion

----

## ATL Overview

- Source models and target models are distinct:
  - **Source models** are **read-only** (they can only be navigated, not modified),
  - **Target models** are **write-only** (they cannot be navigated).

----

## ATL Overview (continued)

- The language is a declarative-imperative hybrid:
  - Declarative part:
    - Matched rules with automatic traceability support,
    - Side-effect free navigation (and query) language: OCL 2.0
  - Imperative part:
    - Called rules,
    - Action blocks.
- Recommended programming style: declarative

----

## ATL Overview (continued)

- A declarative rule specifies:
  - a source pattern to be matched in the source models,
  - a target pattern to be created in the target models for each match during rule application.
- An imperative rule is basically a procedure:
  - It is called by its name,
  - It may take arguments,
  - It can contain:
    - A declarative target pattern,
    - An action block (i.e. a sequence of statements),
    - Both.

----

## ATL Overview (Continued)

- Applying a declarative rule means:
  - Creating the specified target elements,
  - Initializing the properties of the newly created elements.
- There are three types of declarative rules:
  - **Standard** rules that are applied once for each match,
    - A given set of elements may only be matched by one standard rule,
  - **Lazy** rules that are applied as many times for each match as it is referred to from other rules (possibly never for some matches),
  - **Unique** lazy rules that are applied at most once for each match and only if it is referred to from other rules.

----

## Declarative Rules: Source Pattern

- The source pattern is composed of:
  - A labeled set of types coming from the source metamodels,
  - A guard (Boolean expression) used to filter matches.
- A match corresponds to a set of elements coming from the source models that:
  - Are of the types specified in the source pattern (one element for each type),
  - Satisfy the guard.

----

## Declarative Rules: Target Pattern

- The target pattern is composed of:
  - A labeled set of types coming from the target metamodels,
  - For each element of this set, a set of bindings.
  - A binding specifies the initialization of a property of a target element using an expression.
- For each match, the target pattern is applied:
  - Elements are created in the target models (one for each type of the target pattern),
  - Target elements are initialized by executing the bindings:
    - First evaluating their value,
    - Then assigning this value to the corresponding property.

----

## Execution Order

- Declarative ATL frees the developer from specifying execution order:
  - The order in which rules are matched and applied is not specified.
    - Remark: the match of a lazy or unique lazy rules must be referred to before the rule is applied.
  - The order in which bindings are applied is not specified.

----

## Execution Order (continued)
- The execution of declarative rules can however be kept **deterministic**:
  - The execution of a rule cannot change source models
    - It cannot change a match,
  - Target elements are not navigable
    - The execution of a binding cannot change the value of another.

----

## Contents

- Introduction
- Description of ATL
- **Example: Class to Relational**
  - Overview
  - Source metamodel
  - Target metamodel
  - Rule Class2Table
  - Rule SingleValuedAttribute2Column
  - Rule MultiValuedAttribute2Column
- Additional considerations
- Conclusion

----

## Example: Class to Relational, Overview

- The source metamodel Class is a simplification of class diagrams.
- The target metamodel Relational is a simplification of the relational model.
- ATL declaration of the transformation:

```atl
module Class2Relational;

create Mout : Relational from Min : Class;
```

- The transformation excerpts used in this presentation come from:
http://www.eclipse.org/atl/atlTransformations/#Class2Relational


----

## Source: the Class Metamodel

<img src="resources/class-metamodel.pdf" alt="Drawing" style="width: 600px;"/>

----

## The Class Metamodel in EMFatic

```
package Class; 
	abstract class NamedElt {
		attr String name;
	}

	abstract class Classifier extends NamedElt {}

	class DataType extends Classifier {}

	class Class extends Classifier {
		ordered val Attribute[*]#owner attr;
	}

	class Attribute extends NamedElt {
		attr Boolean multiValued;
		ref Classifier type;
		ref Class#attr owner;
	}
```

----

## The Relational Metamodel

<img src="resources/relational-metamodel.pdf" alt="Drawing" style="width: 600px;"/>

----

## The Relational Metamodel in EMFatic
```
package Relational; 

abstract class Named {
attr String name;
}

class Table extends Named {
ordered val Column[*]#owner col;
ref Column[*]#keyOf key;
}

class Column extends Named {
ref Table#col owner; 	
ref Table[0..1]#key keyOf;
ref Type type;
}

class Type extends Named {}

```


----

## Transformation Overview

- Informal description of main rules
  - Class2Table:
    - A table is created from each class,
    - The columns of the table correspond to the single-valued attributes of the class,
    - A column corresponding to the key of the table is created.


----

## Transformation Overview (Continued)

- Auxiliary rules:
  - SingleValuedAttribute2Column:
    - A column is created from each single-valued attribute.
  - MultiValuedAttribute2Column:
    - A table with two columns is created from each multi-valued attribute,
    - One column refers to the key of the table created from the owner class of the attribute,
    - The second column contains the value of the attribute.

----

## `Class2Table` Rule

- A Table is created for each Class:

```
rule Class2Table {
  from -- source pattern
    c : Class!Class
  to -- target pattern
    t : Relational!Table
}
```

----

## `Class2Table` Rule

- The name of the Table is the name of the Class:

```
rule Class2Table {
  from
    c : Class!Class
  to
    t : Relational!Table (
	  name <- c.name -- a simple binding
	  )
}
```

----

## `Class2Table` Rule

- The columns of the table correspond to the single-valued attributes of the class:

```
rule Class2Table {
  from
    c : Class!Class
  to
    t : Relational!Table (
	    name <- c.name,
		col <- c.attr->select(e | -- a binding
		   not e.multiValued)-- using complex navigation
		)
}
```

- Remark: attributes are automatically resolved into columns by automatic traceability support.

----

## `Class2Table` Rule

- Each Table owns a key containing a unique identifier:

```
rule Class2Table {
  from
    c : Class!Class
  to
    t : Relational!Table (
  name <- c.name,
    col <- c.attr->select(e |
    not e.multiValued
    )->union(Sequence {key}),
      key <- Set {key}
      ),
    key : Relational!Column ( -- another target
    name <- ‘Id’-- pattern element
  ) -- for the key
}
```

----

## `SingleValuedAttribute2Column` Rule

- A Column is created for each single-valued Attribute:

```
rule SingleValuedAttribute2Column {
  from -- the guard is used for selection
    a : Class!Attribute (not a.multiValued)
  to
    c : Relational!Column (
      name <- a.name
	  )
}

```

----

## MultiValuedAttribute2Column

- A Table is created for each multi-valued Attribute, which contains two columns:
  - The identifier of the table created from the class owner of the Attribute
  - The value.

```
rule MultiValuedAttribute2Column {
  from
    a : Class!Attribute (a.multiValued)
  to
    t : Relational!Table (
      name <- a.owner.name + ‘_’ + a.name,
      col <- Sequence {id, value}
    ),
    id : Relational!Column (
      name <- ‘Id’
    ),
    value : Relational!Column (
      name <- a.name
    )
}
```

----

## Contents

- Introduction
- Description of ATL
- Example: Class to Relational
- **Additional considerations**
  - Other ATL features
  - ATL in use
- Conclusion

----

## Other ATL Features: Rule Inheritance

- Rule inheritance, to help structure transformations and reuse rules and patterns:
  - A child rule matches a subset of what its parent rule matches,
    - All the bindings of the parent still make sense for the child,
  - A child rule specializes target elements of its parent rule:
    - Initialization of existing elements may be improved or changed,
    - New elements may be created.

----

## Rule Inheritance Syntax

```
abstract rule R1 {
  -- ...
}

rule R2 extends R1 {
  -- ...
}
```

----

## Refining Mode

- Refining mode for transformations that need to modify only a small part of a model:
  - Since source models are read-only target models must be created from scratch,
  - This can be done by writing copy rules for each elements that are not transformed,
    - This is not very elegant.
	

----

## Refining Model
- In refining mode, the ATL engine automatically copies unmatched elements.
- The developer only specifies what changes.
- ATL semantics is respected: source models are still read-only.
  - An (optimized) engine may modify source models in-place but only commit the changes in the end.
- Syntax: replace `from` by `refining`

```
module A2A; create OUT : MMA refining IN : MMA;
```


----

## ATL in Use

- ATL has been used in a large number of application domains.
- A library of transformations is available at
  - https://www.eclipse.org/atl/atlTransformations/
  - More than 40 scenarios,
  - More than 100 single transformations.
- About 100 sites use ATL for various purpose:
  - Teaching,
  - Research,
  - Industrial development,
  - Etc.

----

## ATL in Use

- ATL tools and documentation are available at:  https://eclipse.org/atl/
- Execution engine:
  - Virtual machine,
  - ATL to bytecode compiler,
- Integrated Development Environment (IDE) for:
  - Editor with syntax highlighting and outline,
  - Execution support with launch configurations,
  - Source-level debugger.
- Documentation:
  - Starter’s guide, User manual, Installation guide, etc.

----

## ATL Development Tools: Perspective, Editor and Outline

<img src="resources/atl-perspective.png" alt="Drawing" style="width: 600px;"/>

----

## ATL Development Tools: Launch Configuration

<img src="resources/launch-configuration.png" alt="Drawing" style="width: 600px;"/>

----

## ATL Development Tools: Source-level Debugger

<img src="resources/atl-debugger.png" alt="Drawing" style="width: 600px;"/>

----

## Contents
- Introduction
- Description of ATL
- Example: Class to Relational
- Additional considerations
- **Conclusion**

----

## Conclusion

- ATL has a simple declarative syntax:
  - Simple problems are generally solved simply.
- ATL supports advanced features:
  - Complex OCL navigation, lazy rules, refining mode, rule inheritance, etc.
  - Many complex problems can be handled declaratively.
- ATL has an imperative part:
  - Any problem can be handled.

----

## Thank You!

- Questions?

- Comments?

----

<!--
Context of this work

The present courseware has been elaborated in the context of the “Usine Logicielle” project (www.usine-logicielle.org) of the cluster System@tic Paris-Région with the support of the Direction Générale des Entreprises, Conseil Régional d‘Ile de France, Conseil Général des Yvelines, Conseil Général de l'Essonne, and Conseil Général des Hauts de Seine.

The MDD courseware provided here with the status of open source software is produced under the EPL 1.0 license. 

Overview

This presentation describes a very simple model transformation example, some kind of ATL "hello world".

It is intended to be extended later.

The presentation is composed of the following parts: 

Prerequisites.

Introduction.

Metamodeling.

Transformation.

Conclusion.

Prerequisites

In the presentation we will not discuss the prerequisites.

The interested reader may look in another presentation to these prerequisites on:

MDE (MOF, XMI, OCL).

Eclipse/EMF (ECORE).

AMMA/ATL.

Introduction

The goal is to present a use case of a model to model transformation written in ATL.

This use case is named: “Families to Persons”.

Initially we have a text describing a list of families.

We want to transform this into another text describing a list of persons.

Goal of the ATL transformation we are going to write

Input of the transformation is a model

Output of the transformation should be a model

Each model conforms to a metamodel

The general picture

What we need to provide

In order to achieve the transformation, we need to provide:

A source metamodel in KM3 ("Families").

A source model (in XMI) conforming to "Families".

A target metamodel in KM3 ("Persons").

A transformation model in ATL ("Families2Persons").

When the ATL transformation is executed, we obtain:

A target model (in XMI) conforming to "Persons".

Definition of the source metamodel "Families"

What is “Families”:

A collection of families.

Each family has a name and is composed of members:

A father

A mother

Several sons

Several daughters

Each family member has a first name.

"Families" metamodel (visual presentation and KM3)

"Persons" metamodel (visual presentation and KM3)

The big picture

Families to Persons Architecture

Families to Persons Architecture

Families to Persons Architecture

Families to Persons: project creation

First we create an ATL project by using the ATL Project Wizard.

Families to Persons: ATL transformation creation

Next we create the ATL transformation. To do this, we use the ATL File Wizard. This will generate automatically the header section.

Families to Persons: header section

The header section names the transformation module and names the variables corresponding to the source and target models ("IN" and "OUT") together with their metamodels ("Persons" and "Families") acting as types. The header section of "Families2Persons" is: 

Families to Persons: helper "isFemale()"

A helper is an auxiliary function that computes a result needed in a rule.

The following helper "isFemale()" computes the gender of the current member:

Families to Persons: helper "familyName"

The family name is not directly contained in class “Member”. The following helper returns the family name by navigating the relation between “Family” and “Member”:

Families to Persons: writing the rules

After the helpers we now write the rules:

Member to Male

Member to Female

Summary of the Transformation 

Families to Persons Architecture

ATL Launch Configuration - 1

ATL Launch Configuration - 2

Summary

We have presented here a "hello world" level basic ATL transformation.

This is not a recommendation on how to program in ATL, just an initial example.

Several questions have not been answered

Like how to transform a text into an XMI-encoded model.

Or how to transform the XMI-encoded result into text.

For any further questions, see the documentation mentioned in the resource page (FAQ, Manual, Examples, etc.).

ATL Resource page

ATL Home page

http://www.eclipse.org/m2m/atl/

ATL Documentation page

http://www.eclipse.org/m2m/atl/doc/

ATL Newsgroup

news://news.eclipse.org/eclipse.modeling.m2m

ATL Wiki

http://wiki.eclipse.org/index.php/ATL

Working on the example

There are a lot of exercise questions that could be based on this simple example.

For example, modify the target metamodel as shown and compute the "grandParent" for any Person.
-->