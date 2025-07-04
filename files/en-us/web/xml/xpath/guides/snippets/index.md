---
title: XPath snippets
slug: Web/XML/XPath/Guides/Snippets
page-type: guide
sidebar: xmlsidebar
---

This article provides some XPath code snippets — examples of how to implement a few **utility functions**. These functions are based on standard interfaces that expose XPath functionality to JavaScript code. The snippets are functions you can use in the real world in your own code.

### Node-specific evaluator function

The following custom utility function can be used to evaluate XPath expressions on given XML nodes. The first argument is a DOM node or Document object, while the second is a string defining an XPath expression.

#### Example: Defining a custom node-specific `evaluateXPath()` utility function

```js
// Evaluate an XPath expression aExpression against a given DOM node
// or Document object (aNode), returning the results as an array
// thanks wanderingstan at morethanwarm dot mail dot com for the
// initial work.
function evaluateXPath(aNode, aExpr) {
  const xpe = new XPathEvaluator();
  const nsResolver =
    aNode.ownerDocument === null
      ? aNode.documentElement
      : aNode.ownerDocument.documentElement;
  const result = xpe.evaluate(aExpr, aNode, nsResolver, 0, null);
  const found = [];
  let res;
  while ((res = result.iterateNext())) found.push(res);
  return found;
}
```

Note that the `documentElement` should only be used if you are sure the namespace prefixes in the XPath expression match those in the document you want to query (and that no default namespace is being used). Otherwise, you have to provide your own implementation of XPathNSResolver.

#### Sample usage

Assume we have the following XML document (see also [Parsing and serializing XML](/en-US/docs/Web/XML/Guides/Parsing_and_serializing_XML)):

##### Example: An XML document to use with the custom `evaluateXPath()` utility function

```xml
<?xml version="1.0"?>
<people>
  <person first-name="eric" middle-initial="H" last-name="jung">
    <address street="321 south st" city="denver" state="co" country="usa"/>
    <address street="123 main st" city="arlington" state="ma" country="usa"/>
  </person>

  <person first-name="jed" last-name="brown">
    <address street="321 north st" city="atlanta" state="ga" country="usa"/>
    <address street="123 west st" city="seattle" state="wa" country="usa"/>
    <address street="321 south avenue" city="denver" state="co" country="usa"/>
  </person>
</people>
```

You can now "query" the document with XPath expressions. Although walking the DOM tree can achieve similar results, using XPath expressions is much quicker and more powerful. If you can rely on `id` attributes, `document.getElementById()` is still powerful, but it's not nearly as powerful as XPath. Here are some examples.

##### Example: JavaScript code with the custom `evaluateXPath()` utility function

```js
// display the last names of all people in the doc
let results = evaluateXPath(people, "//person/@last-name");
for (const i in results)
  console.log(`Person #${i} has the last name ${results[i].value}`);

// get the 2nd person node
results = evaluateXPath(people, "/people/person[2]");

// get all the person nodes that have addresses in denver
results = evaluateXPath(people, "//person[address/@city='denver']");

// get all the addresses that have "south" in the street name
results = evaluateXPath(people, "//address[contains(@street, 'south')]");
console.log(results.length);
```

### docEvaluateArray

The following is a utility function to get (ordered) XPath results into an array, regardless of whether there is a special need for namespace resolvers, etc. It avoids the more complex syntax of [`document.evaluate()`](/en-US/docs/Web/API/Document/evaluate) for cases when it is not required as well as the need to use the special iterators on [`XPathResult`](/en-US/docs/Web/API/XPathResult) (by returning an array instead).

#### Example: Defining a `docEvaluateArray()` utility function

```js
// Example usage:
// const els = docEvaluateArray('//a');
// console.log(els[0].nodeName); // gives 'A' in HTML document with at least one link

function docEvaluateArray(
  expr,
  context,
  doc = context ? context.ownerDocument : document,
  resolver = null,
) {
  let i;
  const a = [];
  context ||= doc;

  const result = doc.evaluate(
    expr,
    context,
    resolver,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null,
  );
  for (let i = 0; i < result.snapshotLength; i++) {
    a.push(result.snapshotItem(i));
  }
  return a;
}
```

### getXPathForElement

The following function allows one to pass an element and an XML document to find a unique string XPath expression leading back to that element.

#### Example: Defining a `getXPathForElement()` utility function

```js
function getXPathForElement(el, xml) {
  let xpath = "";
  let pos, tempItem2;

  while (el !== xml.documentElement) {
    pos = 0;
    tempItem2 = el;
    while (tempItem2) {
      if (tempItem2.nodeType === 1 && tempItem2.nodeName === el.nodeName) {
        // If it is ELEMENT_NODE of the same name
        pos += 1;
      }
      tempItem2 = tempItem2.previousSibling;
    }

    xpath = `*[name()='${el.nodeName}' and namespace-uri()='${
      el.namespaceURI ?? ""
    }'][${pos}]/${xpath}`;

    el = el.parentNode;
  }
  xpath = `/*[name()='${xml.documentElement.nodeName}' and namespace-uri()='${
    el.namespaceURI ?? ""
  }']/${xpath}`;
  xpath = xpath.replace(/\/$/, "");
  return xpath;
}
```

### Resources

- [XPath](/en-US/docs/Web/XML/XPath)
- [Forum discussion on this topic](https://forums.mozillazine.org/viewtopic.php?t=229106)

## See also

- [Introduction to using XPath in JavaScript](/en-US/docs/Web/XML/XPath/Guides/Introduction_to_using_XPath_in_JavaScript)
