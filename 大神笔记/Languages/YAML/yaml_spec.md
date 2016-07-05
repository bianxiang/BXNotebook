1.2

{{
两种风格：flow和block。例如[1,2,3]是flow风格（内联），

    - 1
    - 2
    - 3

是block风格。
}}

## 文档状态

The primary objective of this revision is to bring YAML into compliance with JSON as an official subset. YAML 1.2 is compatible with 1.1 for most practical applications - this is a minor revision. An expected source of incompatibility with prior versions of YAML, especially the syck implementation, is the change in implicit typing rules. We have removed unique implicit typing rules and have updated these rules to align them with JSON's productions. In this version of YAML, boolean values may be serialized as “true” or “false”; the empty scalar as “null”. Unquoted numeric values are a superset of JSON's numeric production. Other changes in the specification were the removal of the Unicode line breaks and production bug fixes. We also define 3 built-in implicit typing rule sets: untyped, strict JSON, and a more flexible YAML rule set that extends JSON typing.

The difference between late 1.0 drafts which syck 0.55 implements and the 1.1 revision of this specification is much more extensive. We fixed usability issues with the tagging syntax. In particular, the single exclamation was re-defined for private types and a simple prefixing mechanism was introduced. This revision also fixed many production edge cases and introduced a type repository. Therefore, there are several incompatibilities between syck and this revision as well.

The list of known errors in this specification is available at http://yaml.org/spec/1.2/errata.html. This revision contains fixes for all errors known as of 2009-10-01.

## Abstract

YAML is a human-friendly, cross language, Unicode based data serialization language designed around the **common native data types** of agile programming languages.

## 1. 介绍

本文档既是YAML语言的介绍，又是完整的规范。

YAML achieves a unique cleanness by minimizing the amount of structural characters and allowing the data to show itself in a natural and meaningful way. 例如，缩进表示结构，冒号分割键值对，中划线用于列表。

数据结构有很多，但只要三种结构就能充分表达各种结构：映射（哈希/字典）、序列（数组/列表）和标量（字符串/数字）。YAML leverages these primitives, and adds a simple typing system and aliasing mechanism to form a complete language for serializing any native data structure.

### 1.1. 目标

- YAML is easily readable by humans.
- YAML data is portable between programming languages.
- YAML matches the native data structures of agile languages.
- YAML has a consistent model to support generic tools.
- YAML supports one-pass processing.
- YAML is expressive and extensible.
- YAML is easy to implement and use.

### 1.2. Prior Art

The syntax of YAML was motivated by Internet Mail (RFC0822) and remains partially compatible with that standard. Further, borrowing from MIME (RFC2045), YAML’s top-level production is a stream of independent documents, ideal for message-based distributed processing systems.

YAML使用缩进表示范围。YAML also allows the use of traditional indicator-based scoping similar to JSON’s and Perl’s.

YAML’s double-quoted style uses familiar C-style escape sequences. This enables ASCII encoding of non-printable or 8-bit (ISO 8859-1) characters such as “\x3B”. Non-printable 16-bit Unicode and 32-bit (ISO/IEC 10646) characters are supported with escape sequences such as “\u003B” and “\U0000003B”.

Motivated by HTML’s end-of-line normalization, YAML中，一个换行会被转化为一个空格，空行被解析为换行符。这使得对于标量内容（字符串），自由换行不影响语义。

Like XML’s SOAP, YAML supports serializing a graph of native data structures through an aliasing mechanism. Also like SOAP, YAML provides for application-defined types. This allows YAML to represent rich data structures required for modern distributed computing. YAML provides globally unique type names using a namespace mechanism inspired by Java’s DNS-based package naming convention and XML’s URI-based namespaces. In addition, YAML allows for private types specific to a single application.

YAML was designed to support incremental interfaces that include both input (“getNextEvent()”) and output (“sendNextEvent()”) one-pass interfaces. Together, these enable YAML to support the processing of large documents (e.g. transaction logs) or continuous streams (e.g. feeds from a production machine).

### 1.3. 与JSON的关系

In addition, YAML ventures beyond the lowest common denominator data types, requiring more complex processing when crossing between different programming environments.

YAML可以看做是JSON的超集。比如任一JSON文件都是有效的YAML。

JSON's RFC4627 requires that mappings keys merely “SHOULD” be unique, while YAML insists they “MUST” be. 因此只有当JSON文件中没有重复键时才是有效的YAML文件。

### 1.4. 与XML的关系

Newcomers to YAML often search for its correlation to the eXtensible Markup Language (XML). Although the two languages may actually compete in several application domains, there is no direct correlation between them.

YAML is primarily a data serialization language. XML was designed to be backwards compatible with the Standard Generalized Markup Language (SGML), which was designed to support structured documentation. XML therefore had many design constraints placed on it that YAML does not share. XML is a pioneer in many domains, YAML is the result of lessons learned from XML and other technologies.

It should be mentioned that there are ongoing efforts to define standard XML/YAML mappings. This generally requires that a subset of each language be used. For more information on using both XML and YAML, please visit http://yaml.org/xml.

### 1.5. 术语

This specification uses key words based on RFC2119 to indicate requirement level. In particular, the following words are used to describe the actions of a YAML processor:

- May
The word may, or the adjective optional, mean that conforming YAML processors are permitted to, but need not behave as described.
- Should
The word should, or the adjective recommended, mean that there could be reasons for a YAML processor to deviate from the behavior described, but that such deviation could hurt interoperability and should therefore be advertised with appropriate notice.
- Must
The word must, or the term required or shall, mean that the behavior described is an absolute requirement of the specification.

The rest of this document is arranged as follows. Chapter 2 provides a short preview of the main YAML features. Chapter 3 describes the YAML information model, and the processes for converting from and to this model and the YAML text format. The bulk of the document, chapters 4 through 9, formally define this text format. Finally, chapter 10 recommends basic YAML schemas.

## 2. 预览

不要指望开始就理解本节所有内容，本节只是为了促进了解后续内容。

### 2.1. 集合

Block集合使用缩进表示范围。一行一项。Block序列的每一项前加中划线和空格（`- `）。映射使用冒号和空格（`: `）分割键值对。注释以井号开头（`#`）。

Example 2.1. 标量序列

    - Mark McGwire
    - Sammy Sosa
    - Ken Griffey

Example 2.2.标量到标量的映射

    hr:  65    # Home runs
    avg: 0.278 # Batting average
    rbi: 147   # Runs Batted In

Example 2.3. 标量到序列的映射

    american:
      - Boston Red Sox
      - Detroit Tigers
      - New York Yankees
    national:
      - New York Mets
      - Chicago Cubs
      - Atlanta Braves

Example 2.4. 映射的序列

    -
      name: Mark McGwire
      hr:   65
      avg:  0.278
    -
      name: Sammy Sosa
      hr:   63
      avg:  0.288

YAML还支持一种flow风格。这种风格使用显式的标示符而不是缩进表示范围。流式的序列用逗号分隔项，最外面保卫中括号。流式的映射使用大括号。

Example 2.5. Sequence of Sequences

    - [name        , hr, avg  ]
    - [Mark McGwire, 65, 0.278]
    - [Sammy Sosa  , 63, 0.288]

Example 2.6. Mapping of Mappings

    Mark McGwire: {hr: 65, avg: 0.278}
    Sammy Sosa: {
        hr: 63,
        avg: 0.288
      }

#### 2.2. 结构

YAML使用三个中划线（`---`）分隔指令和文档内容。或仅表示文档的开始（如果前面没有指令）。在通讯信道中，三个中划线（`...`）表示文档结束，没有新文档了。

Example 2.7. 流中的两个文档（每个前面有一个注释）

    # Ranking of 1998 home runs
    ---
    - Mark McGwire
    - Sammy Sosa
    - Ken Griffey

    # Team ranking
    ---
    - Chicago Cubs
    - St Louis Cardinals

Example 2.8.  Play by Play Feed

    ---
    time: 20:03:20
    player: Sammy Sosa
    action: strike (miss)
    ...
    ---
    time: 20:03:47
    player: Sammy Sosa
    action: grand slam
    ...

若要重用节点（对象），首先利用`&`锚记，然后使用`*`应用。

Example 2.9.  Single Document with

    ---
    hr: # 1998 hr ranking
      - Mark McGwire
      - Sammy Sosa
    rbi:
      # 1998 rbi ranking
      - Sammy Sosa
      - Ken Griffey

Example 2.10.  Node for “Sammy Sosa”

    ---
    hr:
      - Mark McGwire
      # Following node labeled SS
      - &SS Sammy Sosa
    rbi:
      - *SS # Subsequent occurrence
      - Ken Griffey

问号加空格（`? `）表示一个复杂的映射键。在block集合中，键值对可以直接出现在中划线、冒号或问号空格的后面：

Example 2.11. Mapping between Sequences

    ? - Detroit Tigers
      - Chicago cubs
    :
      - 2001-07-23

    ? [ New York Yankees,
        Atlanta Braves ]
    : [ 2001-07-02, 2001-08-12,
        2001-08-14 ]

Example 2.12. Compact Nested Mapping

    ---
    # Products purchased
    - item    : Super Hoop
      quantity: 1
    - item    : Basketball
      quantity: 4
    - item    : Big Shoes
      quantity: 1

### 2.3. 标量

标量内容可以以块的形式书写。如果使用literal风格（`|`表示），所有换行都将保留。若使用folded风格，（`>`表示），换行会被转化为空格，除非遇到空行或更加缩进的行。

Example 2.13. In literals, newlines are preserved

    # ASCII Art
    --- |
      \//||\/||
      // ||  ||__

Example 2.14. In the folded scalars, newlines become spaces

    --- >
      Mark McGwire's
      year was crippled
      by a knee injury.

Example 2.15. Folded newlines are preserved for "more indented" and blank lines

    >
     Sammy Sosa completed another
     fine season with great stats.

       63 Home Runs
       0.288 Batting Average

     What a year!

Example 2.16. Indentation determines scope

    name: Mark McGwire
    accomplishment: >
      Mark set a major league
      home run record in 1998.
    stats: |
      65 Home Runs
      0.278 Batting Average

YAML的流式标量包含plain风格和两种引号的风格。双引号风格保留转义字符。单引号风格用于不需要转义时。所有的流式标量，换行都会被转化为空格。

Example 2.17. Quoted Scalars

    unicode: "Sosa did fine.\u263A"
    control: "\b1998\t1999\t2000\n"
    hex esc: "\x0d\x0a is \r\n"

    single: '"Howdy!" he cried.'
    quoted: ' # Not a ''comment''.'
    tie-fighter: '|\-*-/|'

Example 2.18. Multi-line Flow Scalars

    plain:
      This unquoted scalar
      spans many lines.

    quoted: "So does this
      quoted scalar.\n"

### 2.4. Tags

In YAML, untagged nodes are given a type depending on the application. The examples in this specification generally use the [seq](http://yaml.org/spec/1.2/spec.html#tag/repository/seq), [map](http://yaml.org/spec/1.2/spec.html#tag/repository/map) and [str](http://yaml.org/spec/1.2/spec.html#tag/repository/str) types from the fail safe schema. A few examples also use the [int](http://yaml.org/spec/1.2/spec.html#tag/repository/int), [float](http://yaml.org/spec/1.2/spec.html#tag/repository/float), and [null](http://yaml.org/spec/1.2/spec.html#tag/repository/null) types from the JSON schema. The repository includes additional types such as [binary](http://yaml.org/type/binary.html), [omap](http://yaml.org/type/omap.html), [set](http://yaml.org/type/set.html) and others.

Example 2.19. Integers

    canonical: 12345
    decimal: +12345
    octal: 0o14
    hexadecimal: 0xC


Example 2.20. Floating Point

    canonical: 1.23015e+3
    exponential: 12.3015e+02
    fixed: 1230.15
    negative infinity: -.inf
    not a number: .NaN

Example 2.21. Miscellaneous

    null:
    booleans: [ true, false ]
    string: '012345'

Example 2.22. Timestamps

    canonical: 2001-12-15T02:59:43.1Z
    iso8601: 2001-12-14t21:59:43.10-05:00
    spaced: 2001-12-14 21:59:43.10 -5
    date: 2002-12-14

Explicit typing is denoted with a tag using the exclamation point (“!”) symbol. Global tags are URIs and may be specified in a tag shorthand notation using a handle. Application-specific local tags may also be used.

Example 2.23. Various Explicit Tags

    ---
    not-date: !!str 2002-04-28

    picture: !!binary |
     R0lGODlhDAAMAIQAAP//9/X
     17unp5WZmZgAAAOfn515eXv
     Pz7Y6OjuDg4J+fn5OTk6enp
     56enmleECcgggoBADs=

    application specific tag: !something |
     The semantics of the tag
     above may be different for
     different documents.


Example 2.24. Global Tags

    %TAG ! tag:clarkevans.com,2002:
    --- !shape
      # Use the ! handle for presenting
      # tag:clarkevans.com,2002:circle
    - !circle
      center: &ORIGIN {x: 73, y: 129}
      radius: 7
    - !line
      start: *ORIGIN
      finish: { x: 89, y: 102 }
    - !label
      start: *ORIGIN
      color: 0xFFEEBB
      text: Pretty vector drawing.

Example 2.25. Unordered Sets

    # Sets are represented as a
    # Mapping where each key is
    # associated with a null value
    --- !!set
    ? Mark McGwire
    ? Sammy Sosa
    ? Ken Griff

    Example 2.26. Ordered Mappings

    # Ordered maps are represented as
    # A sequence of mappings, with
    # each mapping having one key
    --- !!omap
    - Mark McGwire: 65
    - Sammy Sosa: 63
    - Ken Griffy: 58

### 2.5. Full Length Example

Below are two full-length examples of YAML. On the left is a sample invoice; on the right is a sample log file.

Example 2.27. Invoice

    --- !<tag:clarkevans.com,2002:invoice>
    invoice: 34843
    date   : 2001-01-23
    bill-to: &id001
        given  : Chris
        family : Dumars
        address:
            lines: |
                458 Walkman Dr.
                Suite #292
            city    : Royal Oak
            state   : MI
            postal  : 48046
    ship-to: *id001
    product:
        - sku         : BL394D
          quantity    : 4
          description : Basketball
          price       : 450.00
        - sku         : BL4438H
          quantity    : 1
          description : Super Hoop
          price       : 2392.00
    tax  : 251.42
    total: 4443.52
    comments:
        Late afternoon is best.
        Backup contact is Nancy
        Billsmer @ 338-4338.

Example 2.28. Log File

    ---
    Time: 2001-11-23 15:01:42 -5
    User: ed
    Warning:
      This is an error message
      for the log file
    ---
    Time: 2001-11-23 15:02:31 -5
    User: ed
    Warning:
      A slightly different error
      message.
    ---
    Date: 2001-11-23 15:03:17 -5
    User: ed
    Fatal:
      Unknown variable "bar"
    Stack:
      - file: TopClass.py
        line: 23
        code: |
          x = MoreObject("345\n")
      - file: MoreClass.py
        line: 58
        code: |-
          foo = bar














