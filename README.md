##Page Object Model

Each Page of a Schwartz Document has an Object Model, similar to the Document Object Model (DOM) familiar to web developers. This Object Model is a heirarchical tree structure of objects representing the graphical objects displayed on the Schwartz Canvas.

When you click the **Generate** button, Schwartz produces a source code representation of the currently selected Page by traversing the current Page Object Model using the currently selected Template.

So Templates are used to specify the source code generated from each Page of a Schwartz Document. This Template-based source code generation is what makes Schwartz unique. 

Schwartz has several built-in Templates for popular programming languages like Objective-C, JavaScript, Swift, and SVG. The built-in Templates are not editable, but users can create their own Templates from scratch, or by starting from an existing built-in Template.

Templates are managed in the Template Window (Accessible via the Main Menu > Window > Templates Window or `⌃⎇⌘T`). There you can explore the built-in Templates or create your own.

##Template Tags

The Schwartz Template syntax is very similar to JSP, ASP, or Django. Default tag delimiters like `<%=` `%>` and `<%` `%>` are easily configurable on a per-template basis in the Templates Window (⌃⎇⌘T).

The default tag delimiters were chosen as they mix well with C-inspired languages like ObjC, C, and JavaScript in which curly braces are used often, but angle braces are not. If you prefer `<%` `%>` and `<%=` `%>`, you can use those on your user-defined templates by specifying the tag delimiters in the Templates Window UI.

###Print Tag

**Print Tags** print the value an expression to the text output:

```jsp
My name is <%= name %>.
```

Whitespace around the tag delimiters is optional.

Builtin **Filters** are available:

```jsp
My name is <%=firstName|capitalize%> <%=lastName|capitalize%>, and I'm a <%=profession|lowercase%>.

Mah kitteh sez "<%= lolSpeak|trim|uppercase %>".

<%='Hello World!'|replace:'hello', 'Goodbye Cruel', 'i'%>"

<%= degrees|round %>

<%=degrees|fmt:'%0.1f'%>

<%=degrees|ceil|fmt:'%0.1f'%>

<%= 'now'|fmtDate:'EEE, MMM d, yy' %>
```

###If Tag

**If Tags** offer conditional rendering based on input variables at render time:

```jsp
<% if testVal <= expectedMax || force %>
    Text 1.
<% elif shortCircuit or ('foo' == bar and 'baz' != bat) %>
    Text 2.
<% else %>
    Default Text.
<% /if %>
```

*(Note the boolean test expressions in this example are nonsense, and just intended to demonstrate some of the expression language features.)*

###For Tag

**For Tags** can loop thru arbitrary numerical ranges, and may nest:

```jsp
<% for i in 0 to 10 %>
    <% for j in 0 to 2 %>
        <%=i%>:<%=j%>
    <% /for %>
<% /for %>
```

Numerical ranges may iterate in reverse order, and also offer a "step" option specified after the `by` keyword

```jsp
<% for i in 70 to 60 by 2 %>
    <%=i%><% if not currentLoop.isLast %>,<% /if %>
<% /for %>
```

Prints:

    70,68,66,64,62,60

Note that each For Tag offers access to a `currentLoop` variable which provides information like `currentIndex`, `isFirst`, `isLast`, and `parentLoop`.

For Tags can also loop thru variables representing Cocoa collection objects like `NSArray`, or `NSSet`:

```jsp
<% for obj in vec %>
    <%=obj%>
<% /for %>
```

and `NSDictionary` (note the convenient unpacking of *both key and value*):

```jsp
<% for key, val in dict %>
    <%=key%>:<%=val%>
<% /for %>
```

###Skip Tag

**Skip Tags** can be used to skip the remainder of the current iteration of a For Tag loop. 

Skip Tags are are similar to the `continue` statement in most C-inspired languages (like JavaScript, Java, C++, ObjC, etc) with one important enhancement.

Skip Tags may contain an optional boolean expression. If the expression evaluates true, the Skip Tag is respected, and the remainder of the current iteration of the enclosing For Tag is skipped. However, if the expression evaluates false, the skip tag is ignored, and the current iteration of the For Tag carries on as normal.

The expression within the Skip Tag is essentially a syntactical shortcut. The two following forms are semantically equivalent, but the second is more convenient:

```jsp
<% for i in 1 to 3 %>
    <% if i == 2 %>
        <% skip %>
    <% /if %>
    <%=i%>
<% /if %>
```

```jsp
<% for i in 1 to 3 %>
    <% skip i == 2 %>
    <%=i%>
<% /if %>
```

Both examples produce the following output:

    13

If no expression is present in the Skip Tag, it is always respected, and the current iteration of the enclosing For Tag is always skipped.

###Trim and Indent Tags

As with any templating mechanism, whitespace handling is often a significant concern. Schwartz includes two optional tags that can be used to simplify whitespace handling.

The **Trim Tag** is a block tag that trims both the leading and trailing whitespace from any lines contained within their body content.

Also, any lines inside the Trim Tag bodies containg only other Tags are removed from the output entirely. All empty lines are preserved in the output (but their leading and trailing whitespace is trimmed).

So the following:

```jsp
<% trim %>
    <% if true %>
                            Make it so.
    <% /if %>
<% /trim %>
```

Produces a single line with all leading and trailing whitespace trimed:

    Make it so.

Indentation withing Trim Tags may be controlled with nested **Indent Tags**. The following:

```jsp
<% trim %>
    <% if true %>
        <% indent %>
            Make it so.
        <% /indent %>
    <% /if %>
<% /trim %>
```

Produces a single line indented by 4 spaces:

        Make it so.

###Tag Extensibility

You can implement your own custom Tags by subclassing `TDTag` and overriding `-[TDTag doTagInContext:]`.

##Template Expression Language

As you have seen in the examples above, many tags may contain simple expressions which should be familiar to anyone with experience using JavaScript.

###Logical Expressions

Logical **And** **Or** and **Not** may be expressed using either the familiar JavaScript operators (`&&`, `||`, `!`), or their english equivalents:

```jsp
a && b 

a and b

a || b

a or b

!a

not a
```

###Equality Expressions

Variable equality and inequality may be tested using either the familiar JavaScript operators (`==`, `!=`), or their equivalents (`eq`, `ne`):

```jsp
a == b 

a eq b

a != b

a ne b
```

Note that `a = b` is a syntax error, as assignments are not allowed in the expression language, and the correct equality operator is `==`, not `=`.

###Comparison Expressions

Variables may be compared using either the familiar JavaScript operators (`<`, `<=`, `>`, `>=`), or their equivalents (`lt`, `le`, `gt`, `ge`):

```jsp
a < b 

a lt b

a <= b 

a le b

a > b

a gt b

a >= b

a ge b
```

###Arithmetic Expressions

Arithmetic may be performed using either the familiar JavaScript operators:

```jsp
a + b

a - b

a * b

a / b
```

The modulo operator for finding the remainder after division of one number by another is supported:

```jsp
a % b
```

And explicity negative numbers are supported:

```jsp
-a
```

###Path Expressions

Properties of objects may be reached using a chain of property references called a Path Expression:

```jsp
person.address.zipCode
```

###Boolean Literals

Boolean literals are available matching the JavaScript and Objective-C languages:

```jsp
true

false

YES

NO
```

###Number Literals

Number literals may appear either as integers or as floating point numbers with an optional exponent:

```jsp
42

3.14

16.162e10−36
```

###String Literals

String literals may be wrapped in either single or double quotes:

```jsp
"I'm surrounded by assholes."

'Evil will always triumph, because Good is dumb.'
```

###Sub Expressions

Any expression may be wrapped in parentheses for clarity or to alter the order of operations.

```jsp
(a + b) / c

((a or b) and (c or d))
```

