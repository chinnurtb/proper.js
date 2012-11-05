# Property testing in Javascript

[![Build Status](https://secure.travis-ci.org/mokele/proper.js.png)](http://travis-ci.org/mokele/proper.js)

## Make

    make

## Test

Runs the tests in priv/*.js

    make test

## Running

Then you can try running the properjs command in the properjs repository:

    ./properjs PATH/TO/file.js Object
    ./properjs PATH/TO/file.js Object PATH/TO/include.js 0 PATH/TO/otherfile.js OtherObject

where `PATH/TO/file.js` is a source file for your code, 
`Object.props` is how properjs discovers its properties, 
and `0` denotes a source file without any properties defined

By default running properjs with no arguments runs the Proper.prop
properties.

## Example

Given that reversing a reversed string should always be the same, we can
write a test that asserts this behaviour.

```javascript
// string.js
String.prototype.reverse = function() {
    return this.split("").reverse().join("");
};
String.props = {
    reverse: function() {
        return FORALL(string(),
            function(s) {
                return s.reverse().reverse() == s;
            }
        );
    }
};
```

Then run it with:

    ./properjs string.js String

## Compiling to use Quviq QuickCheck

By default PropEr is used, but to compile to use your system's Quviq QuickCheck
installation compile with:

    make cleanebin compile_eqc


## Supported Types/Generators/Properties

### `FORALL(type(), type(), ..., function(v1, v2, ...) { return boolean() })`

Each argument to `FORALL` before the last function argument is a
generator for a type that will then be passed to your function.
Needs to return `true` on success and `false` on test failure.

```javascript
var myprop = FORALL(integer(), integer(), integer(),
    function(one, two, three) {
      return typeof one == 'number'
          && typeof two == 'number'
          && typeof three == 'number';
    }
);
```

### `LET(type(), type(), ..., function(v1, v2, ...) { return new_value })`

Allows you to define a custom type based on others. The function 
needs to return the generated value for your new type.

An even number generator would required an integer and multiple it by 2.

```javascript
function even_number() {
    return LET(integer(),
        function(i) {
            return i * 2;
        }
    );
}
```

### `SUCHTHAT(type(), function(v) { return boolean() })`

Returns the value generated by `type()` if the condition function returns
`true`. Otherwise the value will be skipped and a new one generated.
There is a hard limit on the number of suchthat attempts that is not yet
configurable via javascript.

```javascript
function even_number() {
    return SUCHTHAT(integer(),
        function(i) {
            return i % 2 == 0;
        }
    );
}
```

The example of `even_number()` using a LET is prefered over this example
since it does not throw away any generated values.

### `SUCHTHATMAYBE(type(), function(v) { return boolean() })`

The same as `SUCHTHAT` except that if the number of attempts to generate
a valid value is exhausted a value will still be generated, regardless
of whether it satisfies the condition.

### `oneof(type(), type(), ...)` `oneof(list(type()))`

Returns a randomly selection member of its arguments or the first
element if there is only one and it is a list.

```javascript
function board_size() {
    return oneof(9, 13, 19);
}
// equivalent to the above
function board_size() {
    return oneof([9, 13, 19]);
}
```

### Basic Types

 * `integer()`
 * `pos_integer()` positive integer
 * `neg_integer()` negative integer
 * `non_neg_integer()` positive integer or `0`
 * `string()`
 * `boolean()` `true` or `false`
 * `string()`
 * `list(type())` generates a list of its argument
 * `char_code()`
