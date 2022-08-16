# Specification

<!-- panels:start -->
<!-- div:left-panel -->
`shoal` format only describes the configuration structure, it's not suited for data serialization, so details like file encoding or representation of various data types are unspecified and left for interpretation to client applications or parsing libraries implementers.
<!-- panels:end -->
## Comments

Comment sections are started with `;` character and closed by the end of line.
<!-- panels:start -->
<!-- div:left-panel -->
```shoal
;This is a comment
foo = hello world ;this is also a comment
```
<!-- div:right-panel -->
```json
{
  "foo" : "hello world"
}
```
<!-- panels:end -->

## Parameters
<!-- panels:start -->
<!-- div:left-panel -->
Parameters have the format: `<parameter name> = <parameter value>`  
Whitespaces around parameter name and value are trimmed.  
Parameter name and `=` must be on the same line.  
Unquoted parameter value must be on the same line with `=`.  
Quoted parameter value can be multiline, but must start on the same line as `=`.  
`'`, `"` and backticks can be used to create a quoted parameter value.  
If multiline values start with a newline character after the quotation mark, the first empty line is skipped. (See `param4` in the example).  
Quoted parameter values can contain separator symbols used by `shoal` without affecting a configuration structure.
<!-- panels:end -->
<!-- panels:start -->
<!-- div:left-panel -->
```shoal
param1=42
param2 = Hello world
param3 ="Hello world"
param4 = "
  Hello world
  Test
"
param5 = ";this is not a comment"
param6 = "'quoted'"
param7 = '"quoted"'
param8 = `'quoted' and "quoted"`
param9 = "'quoted' and `quoted`"
```
<!-- div:right-panel -->
```json
{
  "param1": 42,
  "param2": "Hello world",
  "param3": "Hello world",
  "param4": "  Hello world\n  Test\n",
  "param5": ";this is not a comment",
  "param6": "'quoted'",
  "param7": "\"quoted\"",
  "param8": "'quoted` and \"quoted\"",
  "param9": "'quoted` and `quoted`",
}
```
<!-- panels:end -->
## Arrays
<!-- panels:start -->
<!-- div:left-panel -->
Arrays have the format: `<array name> = <array element>, <array element>, ...` or `<array name> = [<array element>, ...]`  
Whitespaces around array name and array elements are trimmed.      
Array name and `=` must be on the same line.    
When array elements aren't enclosed with brackets, they must be placed on the same line with `=`.  
When array elements aren't enclosed with brackets, an array must have at least two elements.  
When brackets are used, the opening bracket must be placed on the same line with `=`, while elements and closing bracket can be placed on multiple lines.  
When brackets are used, an array can be empty or have a single element. 
<!-- panels:end -->
<!-- panels:start -->
<!-- div:left-panel -->
```shoal
array1 = apple, orange, pineapple         
array2 = [apple, orange, pineapple]
array3 = [apple, 
          orange, 
          pineapple]          
array4 = [
    apple, 
    orange, 
    pineapple
]
array5 = [apple]
array6 = []
param = "apple, orange, pineapple" ;this is not a list
```
<!-- div:right-panel -->
```json
{
  "array1": ["apple","orange","pineapple"],
  "array2": ["apple","orange","pineapple"],
  "array3": ["apple","orange","pineapple"],
  "array4": ["apple","orange","pineapple"],
  "array5": ["apple"],
  "array6": [],
  "param": "apple, orange, pineapple"
}
```
<!-- panels:end -->
## Structures
<!-- panels:start -->
<!-- div:left-panel -->
Structures have the format:
```
#(structure name):
  (parameters)
  and/or
  (arrays)
  and/or
  (structures)
  and/or
  (arrays of structures)
--- or - or --(structure name) or the end of file   
```
Lines with a structure opening token can contain nothing besides whitespaces or a comment.  
Lines with a structure closing token can contain nothing besides whitespaces or a comment.  
`---` marks the end of the current structure and all its parents and returns the context back to the configuration root.    
`-` marks the end of the current structure and returns the context to its parent structure.  
`--<structure name>` marks the end of the current structure and all its parents up to the specified by the provided name and returns the context to its parent structure.  
A structure can be empty.
<!-- panels:end -->
<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#struct1:
  foo=Hello world
  bar=42    
---

#struct2:
  foo=Hello world
  bar=42    
-

#struct3:
  foo=Hello world
  bar=42    
--struct3

#struct4:
---

#struct5:
    foo=Hello world
    bar=42
    #nestedStruct:
      baz = 0          
---

#struct6:
    foo=Hello world    
    #nestedStruct:
      baz = 0
    -        
    bar=42      
---

#struct7:
    foo=Hello world    
    #nestedStruct:
      baz = 0
    --nestedStruct        
    bar=42      
---

#struct8:
    foo=Hello world    
    #nestedStruct:
      baz = 0
      #nestedStruct2:
        abc = test
    --nestedStruct                
    bar=42      
---

#struct9:
    foo=Hello world    
    #nestedStruct:
      baz = 0
      #nestedStruct2:
        abc = test
      -
    -                
    bar=42      
---
```
<!-- div:right-panel -->
```json
{
  "struct1" : {
    "foo":"Hello world",
    "bar":42 
  },
  "struct2" : {
    "foo":"Hello world",
    "bar":42 
  },
  "struct3" : {
    "foo":"Hello world",
    "bar":42 
  },
  "struct4" : {},
  "struct5" : {
    "foo":"Hello world",
    "bar":42,
    "nestedStruct": {
      "baz": 0  
    }
  },
  "struct6" : {
    "foo":"Hello world",
    "bar":42,
    "nestedStruct": {
      "baz": 0  
    }
  },
  "struct7" : {
    "foo":"Hello world",
    "bar":42,
    "nestedStruct": {
      "baz": 0  
    }
  },
  "struct8" : {
    "foo":"Hello world",
    "bar":42,
    "nestedStruct": {
      "baz": 0,
      "nestedStruct2": {
        "abc": "test"
      }
    }
  },
  "struct9" : {
    "foo":"Hello world",
    "bar":42,
    "nestedStruct": {
      "baz": 0,
      "nestedStruct2": {
        "abc": "test"
      }
    }
  }
}
```
<!-- panels:end -->
## Arrays of structures
<!-- panels:start -->
<!-- div:left-panel -->
Arrays of structures have the format:
```
#(name of array of structures):
### 
(element of array - a structure without opening and closing token)
... zero or more elements separated by ###
###
(element of array)
...
--- or - or --(name of array of structures) or the end of file   
```

Lines with a structure opening token can contain nothing besides whitespaces or a comment.  
The next line after the opening token must contain an array elements separator `###` even when the array is empty.  
Lines with array elements separator can contain nothing besides whitespaces or a comment.  
Lines with a structure closing token can contain nothing besides whitespaces or a comment.  

`---` marks the end of the current array of structures and all its parents and returns the context back to the configuration root.    
`-` marks the end of the current array of structures and returns the context to its parent structure.  
`--<array of structure name>` marks the end of the current array of structures and all its parents up to the specified by the provided name and returns the context to its parent structure.  
An array of structures can be empty.
<!-- panels:end -->
<!-- panels:start -->
<!-- div:left-panel -->
```shoal
#aos1:
###
  foo=Hello world
  bar=42
###
  foo=Test
  bar=9
---

#aos2:
###
  foo=Hello world
  bar=42
###
  foo=Test
  bar=9
-

#aos3:
###
  foo=Hello world
  bar=42
###
  foo=Test
  bar=9
--aos3

#aos4:
###
---

#aos5:
###
  foo=Hello world
  bar=42
  #nestedAos:
  ###
    baz = 0
---

#aos6:
###
  foo=Hello world  
  #nestedAos:
  ###
    baz = 0
  -  
  bar=42
###
  foo=Hello  
  #nestedAos:
  ###    
  --nestedAos  
  bar=9
---
```
<!-- div:right-panel -->
```json
{
  "aos1" : [
    { 
      "foo":"Hello world",
      "bar":42
    },
    {
      "foo":"Test",
      "bar":9
    }
  ],
  "aos2" : [
    { 
      "foo":"Hello world",
      "bar":42
    },
    {
      "foo":"Test",
      "bar":9
    }
  ],
  "aos3" : [
    { 
      "foo":"Hello world",
      "bar":42
    },
    {
      "foo": "Test",
      "bar": 9
    }
  ],
  "aos4" : [],
  "aos5" : [
    { 
      "foo":"Hello world",
      "bar":42,
      "nestedAos": [
        {
           "baz": 0
        }
      ] 
    }
  ],
  "aos6" : [
    { 
      "foo":"Hello world",
      "bar":42,
      "nestedAos": [
        {
           "baz": 0
        }
      ] 
    },
    { 
      "foo":"Hello",
      "bar":9,
      "nestedAos": []
    }
  ]
}

```
<!-- panels:end -->
