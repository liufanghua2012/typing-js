#A Quick Guide to Typing.js

##1. Overview

Typing.js is a simple, intuitive and expressive type checking and JSON schema validation library for Node.js. It goes beyond the JSON schema validation, it actually looks like a runtime type system for JavaScript. 

**Features:**

1. a set of commonly used built-in types, i.e. int, str, array, tuple, table;
2. C++ template like type definition with array, tuple and table;
3. structure based pattern matching
4. recursive type

##2. Sample: 
```JavaScript
// import module and the built-in types
var typing = require('typing');
var int = typing.int;
var str = typing.str;
...
var tuple = typing.tuple;
var table = typing.table;

// primitive type checking
typing.check(str(3,10), 'hello')

// composite type checking
typing.check(tuple(int, str, tuple(str, str)), [23, 'todd', ['82301588', 'todd@mail.com']])

// pattern matching
typing.check(
    { 
        id : int, 
        male : bool, 
        name : str(1,50), 
        contact : { tel : str, email : str }
    }, 
    { 
        id : 23, 
        male : true, 
        name : 'todd', 
        contact : { tel : '82301588', email : 'tood@mail.com' }
    }
)
```

##3. Usage:

```Javascript
typing.check(type, data); //true: matched; false: not matched
```

where ```type``` can be:

1) Function with ```__name__``` and ```__check__``` property. 

For example, the built-in type ```bool``` is defined as:

```Javascript
function bool() {}
bool.__name__ = 'bool';
bool.__check__ = function(value) { return true === value || false === value; }
```

```typing.check(type, data)``` will call ```type.__check__(data)``` in this case. You can define your own types this way, however, be aware that typing comes with a set of built-in types, which can be used to construct complex types. 

2) JSON object.

For example, 

```Javascript
{
    status : {
        code : int,
        message : str
    },
    data : table(int, str, tuple(str, str))
}
```

```typing.check(type, data)``` will perform pattern matching between type and data based on the structure and recursively check the type of each property. 

3) String.

For example, 

```Javascript
// define 3x3 matrix type under the name 'matrix_3x2'
typing.define('matrix_3x2', tuple(tuple(int, int), tuple(int, int), tuple(int, int)));

// check type with the type name
assert(typing.check('matrix_3x3', [[11, 12], [21, 22], [31, 32]]));
```

**3.1. Define custom type with the built-in types**

```JavaScript
// int(1): integer >= 1;
// str(1,50): string with length between 1 to 50;
// tuple: array with specified type and number of elements
var t_employee = tuple(int(1), str(1,50), tuple(str,str));

// matched
assert(typing.check(t_employee, [123, 'todd', ['1355-0011-107', 'CD 5607']]));

// not matched, id must be >= 1
assert(false == typing.check(t_employee, [0, 'todd', ['1355-0011-107', 'CD 5607']]));
```

**3.2. Define custom type in JSON**

```JavaScript
// typing will do pattern matching based the type defined in JSON
// nullable : null or the wrapped type
// table: equivalent of array(tuple)
var t_response = {
    status : {
        code : int,
        message : str
    },
    data : nullable(table(int(1), str(1,50), tuple(str,str)))
};

// matched
assert(typing.check(t_response, {
    status : { 
        code : 200, 
        message : 'OK'
    },
    data : [
        [1, 'Todd', ['1355-0011-107', 'CA 5607']],
        [2, 'April', ['1582-0011-108', 'CA 5607']],
        [3, 'Rex', ['1522-1011-138', 'CA 1008']]
    ]
});

// matched
assert(typing.check(t_response, {
    status : { 
        code : 404, 
        message : 'NOT FOUND'
    }
});

// not matched, status.message is missing
assert(typing.check(t_response, {
    status : {
        code : 300
    }
});
```

**3.3. Define recursive type** 
```Javascript
// define a recursive binary tree type under the name 'tree'
// nullalbe(type): extend the wrapped type to accept null value
// type(name): define a lazy resolved type
typing.define('tree', {
    value : int,
    left : nullable(type('tree')),
    right : nullable(type('tree'))
});

// matched
assert(typing.check('tree', {
    value : 1,
    left : {
        value : 2,
        left : { value : 3 }
    },
    right : {
        value : 4,
        right : { value : 5 }
    }
}));
```

##4. Built-in Types##

**1. any**

```any``` matches any value in JavaScript including null and undefined. Examples:

```Javascript
typing.check(any, null); //true
typing.check(any, undefined); //true
typing.check(any, 123); //true
typing.check(any, 'hello typing.js'); //true
typing.check(any, {}); //true
typing.check(any, []); //true
typing.check(any, function(){}); //true
```

**2. bool**

```bool``` matches ```true``` or ```false```. Examples:

```JavaScript
typing.check(bool, true); //true
typing.check(bool, false); //true
```

**3. int**

```int``` matches integers. You can specify the minimal and maximal value by ```int(min)``` or ```int(min,max)```. Examples:

```JavaScript
typing.check(int, -103); //true, no min and max
typing.check(int, 'hello'); //false
typing.check(int(100), 99); //false, matches integer >= 100
typing.check(int(0,1000), 1000); //true, matches integer >= 0 and <= 1000
```

**4. str**

```str``` matches strings. You can specify the minimal and maximal lenght by ```str(min)``` or ```str(min,max)```. Examples:

```JavaScript
typing.check(str, null); //true
typing.check(str, ''); //true
typing.check(str(0), null); //true
typing.check(str(0), ''); //true
typing.check(str(3), 'foo'); //true, matches string with length >= 3
typing.check(str(4), 'foo'); //false
typing.check(str(1,3), ''); //false, matches string with length >= 1 and <= 3 
typing.check(str(1,3), 'hello'); //false, matches string with length >= 1 and <= 3 
```
