---
layout: post
title: "Excel to Ordinal Converter"
categories: snippets
tags: [java, javascript]
comments: true
---

## Ordinal and Excel indexes

Sometimes, it is necessary to convert from some excel indexes (represented as
letters, such as A, B, C) to ordinal indexes (0, 1, 2). This is particularly
useful in scenarios is which a CSV file with a lot of rows is opened/edited on
Excel and should then be mapped in a positional to a CSV parser, such as
[Univocity](https://github.com/uniVocity/univocity-parsers){:target="_blank"}

## Java 8 snippet
<input type="button" value="Copy to Clipboard" onclick="copyToClipboard(0)"/>

```java
// Converts an Excel position(column) to an ordinal position(index)
// To be applied with ExcelCSV->Univocity parsing mechanism
public String excelToOrdinalConverter(String excelColumn) {
    OptionalInt value = IntStream.range(0, excelColumn.length())
        .map(i -> i == 0
            ? (excelColumn.charAt(i) - 64)
            : ((excelColumn.charAt(i-1) - 64) * 26) + (excelColumn.charAt(i) - 64))
        .reduce((a, b) -> b);

    return excelColumn + ": @Parsed(index = " + (value.getAsInt() - 1) + ")";
}
```

## Javascript snippet

<input type="button" value="Copy to Clipboard" onclick="copyToClipboard(1)"/>
```javascript
// @marcos3m
var foo = function(val) {
  var base = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ', i, j, result = 0;

  for (i = 0, j = val.length - 1; i < val.length; i += 1, j -= 1) {
    result += Math.pow(base.length, j) * (base.indexOf(val[i]) + 1);
  }

  return result;
};

console.log(['IQ'].map(foo));
```
