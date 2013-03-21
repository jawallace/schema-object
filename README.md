node-schema-object
==================

Designed to enforce schema on Javascript objects. Allows you to specify type, transformation and validation of values via a set of attributes. Support for sub-schemas included.

```
npm install node-schema-object
```

#Very basic usage example
```
var SchemaObject = require('node-schema-object');

// Create User schema
var User = new SchemaObject({
  firstName: String,
  lastName: String,
  birthDate: Date
});

// Initialize instance of user
var user = new User({firstName: 'Scott'});

// Prints {firstName: 'Scott', lastName: undefined, birthDate: undefined}
console.log(user.toObject());
```

#Advanced example
```
var SchemaObject = require('node-schema-object');

// Create custom basic type
var NotEmptyString = {type: String, minLength: 1};

// Create sub-schema for user's Company
var Company = new SchemaObject({
  name: NotEmptyString,
  dateEstablished: Date
});

// Create User schema
var User = new SchemaObject({
  // Basic user information
  firstName: NotEmptyString,
  lastName: NotEmptyString,
  gender: {type: String, enum: ['m', 'f']},
  company: Company,
  
  // Create field which reflects other values but can't be directly modified
  fullName: {type: String, readOnly: true, default: function() {
    return (this.firstName + ' ' + this.lastName).trim();
  }}
});

// Initialize a new instance of the User with a value
var user = new User({firstName: 'Scott', lastName: 'Hovestadt', gender: 'm'});

// Set company name
user.company.name = 'My Company';

// Prints {firstName: 'Scott', lastName: 'Hovestadt', gender: 'm', company: {name: 'My Company', dateEstablished: undefined}, fullName: 'Scott Hovestadt'}
console.log(user.toObject());
```

#Types

Supported types:
- String
- Number
- Boolean
- Date
- Array (including types within Array)
- Object (including typed SchemaObjects for sub-schemas)
- 'alias'
- undefined

When a type is specified, it will be enforced. Typecasting is enforced on all types. If a value cannot be typecasted to the correct type, the original value will remain untouched.

Types can be extended with a variety of attributes. Some attributes are type-specific and some apply to all types.

##General attributes

###transform
Called immediately when value is set and before any typecast is done.
```
name: {type: String, transform: function(value) {
  // Modify the value here...
  return value;
}}
```

###default
Provide default value. You may pass value directly or pass a function which will be executed when the value is retrieved. The function is executed in the context of the object and can use "this" to access other properties.
```
country: {type: String, default: "USA"}
```

###readOnly
If true, the value can be read but cannot be written to. This can be useful for creating fields that reflect other values.
```
fullName: {type: String, readOnly: true, default: function(value) {
  return (this.firstName + ' ' + this.lastName).trim();
}}
```

###invisible
If true, the value can be written to but isn't outputted as an index when toObject() is called. This can be useful for creating aliases that redirect to other indexes but aren't actually present on the object.
```
zip: String,
postalCode: {type: 'alias', invisible: true, alias: 'zip'}
// this.postalCode = 12345 -> this.toObject() -> {zip: '12345'}
```


##String

###stringTransform
Called after value is typecast to string **if** value was successfully typecast but called before all validation.
```
postalCode: {type: String, stringTransform: function(string) {
  // Type will ALWAYS be String, so using string prototype is OK.
  return string.toUpperCase();
}}
```

###regex
Validates string against Regular Expression. If string doesn't match, it's rejected.
```
memberCode: {type: String, regex: new RegExp('^([0-9A-Z]{4})$')}
```

###enum
Validates string against array of strings. If not present, it's rejected.
```
gender: {type: String, enum: ['m', 'f']}
```

###minLength
Enforces minimum string length.
```
notEmpty: {type: String, minLength: 1}
```

###maxLength
Enforces maximum string length.
```
stateAbbrev: {type: String, maxLength: 2}
```


##Number

###min
Number must be > min attribute or it's rejected.
```
positive: {type: Number, min: 0}
```

###max
Number must be < max attribute or it's rejected.
```
negative: {type: Number, max: 0}
```


##Alias

###index (required)
The index key of the property being aliased.
```
zip: String,
postalCode: {type: 'alias', alias: 'zip'}
// this.postalCode = 12345 -> this.toObject() -> {zip: '12345'}
```

#Unit tests
```
String
  typecasting
    ✓ should typecast integer to string 
    ✓ should typecast boolean to string 
    ✓ should join array into string 
    ✓ should reject object 
  regex
    ✓ should only allow values that match regex ^([A-Z]{4})$ 
  enum
    ✓ should allow values in enum 
    ✓ value should remain untouched when non-enum is passed 
    ✓ default must be in enum or is rejected 
    ✓ default should be set when in enum 
  stringTransform
    ✓ should return lowercase 
    ✓ should always be passed a String object and not called if undefined or null 
  read only
    ✓ should always be default value 
  minLength
    ✓ should not allow empty strings 

Number
  typecasting
    ✓ should typecast string to number 
    ✓ should typecast boolean to number 
  min
    ✓ should reject values below min 
  max
    ✓ should reject values above max 

Boolean
  typecasting
    ✓ should typecast string to boolean 
    ✓ should typecast number to boolean 

Object
  accessing properties
    ✓ should set properties without initializing object 
  schema
    ✓ should allow nested schemas 

Array
  typecasting
    ✓ should typecast all array elements to string 
    ✓ should transform all strings to lowercase 

Date
  typecasting
    ✓ should typecast string "June 21, 1988" to date 
    ✓ should typecast string "06/21/1988" to date 
    ✓ should typecast string "6/21/1988" to date 
    ✓ should reject nonsense strings 
    ✓ should typecast integer timestamp seconds to date 
    ✓ should typecast integer timestamp milliseconds to date 
    ✓ should return Date object within index when toObject() is called 
    ✓ should reject boolean 
    ✓ should reject array 
    ✓ should reject object 

any type
  transform
    ✓ should turn any string to lowercase but not touch other values 
  default
    ✓ default as function + readOnly to combine properties into single readOnly property 
  alias
    ✓ should allow alias to be used to set values 
    ✓ should allow alias to pre-transform values 

converting schemaobject to plain object
  ✓ should have index string with value "hello" 
```
