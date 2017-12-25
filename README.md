# (struct pack)
[![Build Status](https://travis-ci.org/weinholt/struct-pack.svg?branch=master)](https://travis-ci.org/weinholt/struct-pack)

This is an R6RS library for working with packed byte structures. It is
similar to `struct` in Python or `pack` and `unpack` in Perl.

## API

```Scheme
(import (struct pack))
```

This library uses format strings which specify binary fields. The
format strings are read left-to-right and have the following
interpretation:

 - `c` - s8, a signed byte;
 - `C` - u8, an unsigned byte;
 - `s` - s16, a signed 16-bit word;
 - `S` - u16, an unsigned 16-bit word;
 - `l` - s32, a signed 32-bit word;
 - `L` - u32, an unsigned 32-bit word;
 - `q` - s64, a signed 64-bit word;
 - `Q` - u64, an unsigned 64-bit word;
 - `f` - an IEEE-754 single-precision number;
 - `d` - an IEEE-754 double-precision number;
 - `x` - one byte of padding (zero);
 - `a` - enable automatic natural alignment (default);
 - `u` - disable automatic natural alignment;
 - `!` and `>` - the following fields will have big-endian
     (network) byte order;
 - `<` - the following fields will have little-endian byte order;
 - `=` - the following fields will have native endianness;
 - whitespace - ignored;
 - decimals - repeat the next format character N times.

By default fields are aligned. Padding bytes are inserted to align
fields to their natural alignment so that e.g. a 32-bit field is
aligned to a 4 byte offset.

### (unpack fmt bytevector [offset])

Returns as many values as there are fields in *fmt*. The values are
fetched from *bytevector* starting at *offset* (by default 0). For
example, if the format string is `"C"`, this translates into a
`bytevector-u8-ref` call.

```Scheme
(import (struct pack))
(unpack "!xd" (pack "!xd" 3.14))
;; => 3.14
```

```Scheme
(number->string (unpack "!L" #vu8(#x00 #xFB #x42 #xE3)) 16)
;; => "FB42E3"
```

```Scheme
(unpack "!2CS" #vu8(1 2 0 3))
;; => 1
;; => 2
;; => 3
```

### (pack fmt values ...)

Returns a new bytevector containing the values encoded as per the
*fmt* string.

```Scheme
(pack "!CCS" 1 2 3)
;; => #vu8(1 2 0 3)
```

```Scheme
(pack "!CSC" 1 2 3)
;; => #vu8(1 0 0 2 3)
```

```Scheme
(pack "!SS" (question-qtype x) (question-qclass x))
;; ===>
(let ((bv (make-bytevector 4)))
  (pack! "!SS" bv 0 (question-qtype x) (question-qclass x))
  bv)
;; ===>
(let ((bv (make-bytevector 4)))
  (let ((bv bv) (off 0))
    (bytevector-u16-set! bv 0 (question-qtype x)
                         (endianness big))
    (bytevector-u16-set! bv 2 (question-qclass x)
                         (endianness big))
    (values))
  bv)
```

### (pack! fmt bytevector offset values ...)

The same as `pack`, except it modifies *bytevector* in place and
returns no values.

### (get-unpack binary-input-port fmt)

Reads `(format-size fmt)` bytes from *binary-input-port* and unpacks
them according to the format string. Returns the same values as
`unpack` would.

```Scheme
(get-unpack port "4xCCxCC7x")
;; ===>
(let ((bv (get-bytevector-n port 16))
      (off 0))
  (values (bytevector-u8-ref bv 4) (bytevector-u8-ref bv 5)
          (bytevector-u8-ref bv 7) (bytevector-u8-ref bv 8)))
```

### (put-pack binary-output-port fmt values ...)

Packs and writes the values to *binary-output-port*. Equivalent to
`(put-bytevector binary-output-port (pack fmt values ...)`, but may
be implemented more efficiently.

New in version 1.1.0.

### (format-size fmt)

The number of bytes the fields in the format string would use if
packed together, including any padding.

```Scheme
(format-size "!xQ")
;; => 16
```

```Scheme
(format-size "!uxQ")
;; => 9
```
