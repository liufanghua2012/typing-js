typing.js
=========

A simple and intuitive object type definition and validation library for JavaScript.


###Samples:

```JavaScript
// imports
var typing = require('typing');

// import the built-in types
var int = typing.int;
var str = typing.str;
var tuple = typing.tuple;
````

*Test case 1*
```JavaScript
// define the employee type, int(1): integer >= 1, str(1,50): string with length between 1 to 50
var t_employee = tuple(int(1), str(1,50), tuple(str,str));

// valid t_employee data
assert(typing.check(t_employee, [123, 'todd', ['1355-0011-107', 'CA 5607']]));

// invalid t_employee data, id must be >= 1
assert(false == typing.check(t_employee, [0, 'todd', ['1355-0011-107', 'CA 5607']]));
```

*Test case 2*

```JavaScript
var t_response = {
    status : {
        code : int,
        message : str
    },
    data : nullable(table(int(1), str(1,50), tuple(str,str)))
};

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
```
