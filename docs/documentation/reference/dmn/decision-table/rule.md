---

title: 'DMN Decision Table Rule'
sidebar_position: 30

menu:
  main:
    name: "Rule"
    identifier: "dmn-ref-decision-table-rules"
    parent: "dmn-ref-decision-table"
    description: "Specify Conditions and Conclusions"

aliases: [reference/dmn11/decision-table/rule/]
---

![Example img](./img/rule.png)Rule" class="no-lightbox

A decision table can have one or more rules. Each rule contains input and
output entries. The input entries are the condition and the output entries the
conclusion of the rule. If each input entry (condition) is satisfied, then the
rule is satisfied and the decision result contains the output entries
(conclusion) of this rule.

A rule is represented by a `rule` element inside a `decisionTable` XML element.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/" id="definitions" name="definitions" namespace="http://operaton.org/schema/1.0/dmn">
  <decision id="dish" name="Dish">
    <decisionTable id="decisionTable">
      <!-- ... -->
      <rule id="rule2-950612891-2">
        <inputEntry id="inputEntry21">
          <text>"Winter"</text>
        </inputEntry>
        <inputEntry id="inputEntry22">
          <text><![CDATA[$\leq$ 8]]></text>
        </inputEntry>
        <outputEntry id="outputEntry2">
          <text>"Roastbeef"</text>
        </outputEntry>
      </rule>
      <!-- ... -->
    </decisionTable>
  </decision>
</definitions>
```

# Input Entry (Condition)

![Example img](./img/input-entry.png)Input Entry" class="no-lightbox

A rule can have one or more input entries, which are the conditions of the rule.
Each input entry contains an expression in a `text` element as child of an
`inputEntry` XML element.

The input entry is satisfied when the evaluated expression returns `true`.

```xml
<inputEntry id="inputEntry41">
  <text>"Spring"</text>
</inputEntry>
```

## Expression Language of an Input Entry

The expression language of the input entry can be specified by the
`expressionLanguage` attribute on the `inputEntry` XML element. The supported
expression languages are listed in the [User Guide][supported EL].

```xml
<inputEntry id="inputEntry41"
            expressionLanguage="juel">
  <text>cellInput == "Spring"</text>
</inputEntry>
```

If no expression language is set then the global expression
language, which is set on the `definitions` XML element, is used.

```xml
<definitions id="definitions"
             name="definitions"
             xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/"
             expressionLanguage="groovy"
             namespace="http://operaton.org/schema/1.0/dmn">
  <!-- ... -->
</definitions>
```

In case no global expression language is set, the default expression language
is used instead. The default expression language for input entries is [FEEL].
Please refer to the [User Guide][default EL] to read more about expression
languages.


# Output Entry (Conclusion)

![Example img](./img/output-entry.png)Output Entry" class="no-lightbox

A rule can have one or more output entries, which are the conclusions of the
rule. Each output entry contains an expression in a `text` element as child of
an `outputEntry` XML element.

```xml
<outputEntry id="outputEntry4">
  <text>"Steak"</text>
</outputEntry>
```

## Expression Language of an Output Entry

The expression language of the expression can be specified by the
`expressionLanguage` attribute on the `outputEntry` XML element. The supported
expression languages are listed in the [User Guide][supported EL].

```xml
<outputEntry id="outputEntry4"
            expressionLanguage="groovy">
  <text>"Steak"</text>
</outputEntry>
```

If no expression language is set then the global expression
language, which is set on the `definitions` XML element, is used.

```xml
<definitions id="definitions"
             name="definitions"
             xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/"
             expressionLanguage="groovy"
             namespace="http://operaton.org/schema/1.0/dmn">
  <!-- ... -->
</definitions>
```

In case no global expression language is set, the default expression language
is used instead. The default expression language for output entries is JUEL.
Please refer to the [User Guide][default EL] to read more about expression
languages.

# Empty Entries

## Empty Input Entry

In case an input entry is irrelevant for a rule, the expression is empty, which
is always satisfied.

```xml
<inputEntry id="inputEntry41">
  <text/>
</inputEntry>
```

If FEEL is used as expression language, then an empty input entry is represented
by a `-`. Otherwise, the expression is empty.

## Empty Output Entry

If the output entry is empty, then the output is ignored and not part of the
decision table result.

```xml
<outputEntry id="outputEntry4">
  <text/>
</outputEntry>
```

## Both Entries Empty

In case of a rule has an empty input and output entry, the outcome of evaluation
will be determined with precedence of the empty input outcome.

# Description

![Example img](./img/description.png)Description" class="no-lightbox

A rule can be annotated with a description that provides additional
information. The description text is set inside the `description` XML element.

```xml
<rule id="rule4">
  <description>Save money</description>
  <!-- ... -->
</rule>
```


[supported EL]: ../user-guide/dmn-engine/expressions-and-scripts.md#supported-expression-languages
[default EL]: ../user-guide/dmn-engine/expressions-and-scripts.md#default-expression-languages
[FEEL]: ../reference/dmn/feel/index.md
