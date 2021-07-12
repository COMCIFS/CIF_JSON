# JSON representation of CIF information (DRAFT)

## Introduction

The Crystallographic Information Framework (CIF) is a set of specifications for describing
scientific information.  One or more *data values* are attached to *data names* to form
*data items*. Data items are collected together into *data blocks*.  Data blocks are in turn
collected into *data files*.  Data names, the possible values that can be attached to
them, and data name membership in *categories* (tabulations), are described in CIF 
*dictionaries*. In the following standard, *CIF information* refers to any scientific
information that can be defined and encapsulated using the above model.  The CIF1 and CIF2
syntaxes describe closely related file formats for expressing CIF information in a
form suitable for long-term archiving and transfer.

JSON is a lightweight language for serialisation and transfer of data.
JSON is often used as a "pipeline" language to transfer information
between programs within a single system, and also as a way of
transferring information between internet applications.  JSON
libraries are widely available and therefore it is relatively simple to
implement parsing and output of JSON.

A standard way of expressing CIF information in JSON would allow
programs from different authors running within a single context (such
as a web browser or operating system) to interoperate with minimal work. The following
CIF-JSON standard is intended to facilitate such interoperability.

## The JSON representation of CIF information

In the following, *name* refers to a JSON name, which is double-quoted. *Value* refers to a JSON value, which may be a quoted
string or one of the JSON values `false` or `null`. *Item* refers to a JSON name:value pair. *Object* refers to a JSON  object,
a comma-separated collection of items, starting with `{` and ending with `}`. *Array* refers to a JSON array, a comma-separated
collection of values, starting with `[` and ending with `]`. CIF-related datastructures
are described in the introduction.

### Encoding and characterset requirements

1. The JSON file must conform to the i-JSON standard [RFC7493](https://tools.ietf.org/html/rfc7493)
1. All names used for CIF data block headers, CIF data names, and CIF save block headers must be represented in CIF-JSON in
Unicode case-normal form (also referred to as “lower-case” below), and otherwise conform to the characterset restrictions
of the CIF2 syntax.

### The CIF-JSON object

1. A CIF data file (a collection of CIF data blocks) is represented as a single JSON object (the *top-level object*).
1. The top-level object contains a single JSON item, which must be named `CIF-JSON`. The value of this item is a
JSON object, referred to below as the *CIF-JSON object*.
1. A CIF data block is represented by an item within the CIF-JSON object. This item is referred 
to here as the *JSON datablock item*.  As noted above, the name of this item must 
be in case-normal form and conform to the characterset restrictions of the CIF2 syntax for data block headers.
1. The CIF-JSON object may contain multiple JSON datablock items.
1. A CIF data item is an item in a JSON datablock object (*JSON data item*). The item name is the CIF data name, 
including the underscore, in case-normal form.
1. The value of the JSON data item is a JSON array (referred to as the "JSON array data value" below).
1. The JSON array data value contains the one or more CIF data values associated with the CIF data name. 
1. Entries at the same position in JSON array data values correspond to one another if the data names belong to the same
category. In other words, such entries would belong to the same row if these data items were tabulated.
1. CIF data values are represented within JSON array data values as follows:
   1. CIF string values are represented as JSON string values
   1. CIF number values are represented as JSON string values formatted according to the 
   `<Numeric>` production in International Tables
   for Crystallography, Volume G, Section 2.2.7.3 paragraph 57. 
    3. The special CIF value `.` (null) is represented as JSON `false`.
    4. The special CIF value `?` (unknown) is represented as JSON `null`.
    5. (CIF2 only).  CIF2 list data values are represented as JSON arrays. The data values appearing
  in the array are represented in the same way as non-list data values.
    6. (CIF2 only).  CIF2 table data values are represented as JSON objects. The names in the object
  are the same as the names in the CIF table, including case. The values in the CIF table are represented in the same
  way as other CIF data values

1. If the CIF data block includes save frames (currently only used in dictionaries), 
the JSON datablock object must contain the special item named `Frames`. The value of this item
is a JSON object. Each item in this object corresponds to a save frame
found in the data block. The value for each of these save frame items is a JSON datablock object, represented
in the same way as a CIF data block.
10. An optional item named `Metadata` may be present in the CIF-JSON object. The item name
must begin with a capital 'M' to distinguish it from a normal data block name. The `Metadata` item contains
information that is useful for conversion of CIF-JSON objects to other syntaxes
(for example, CIF syntax files) and information about the CIF-JSON version.  The following items are defined for the `Metadata` object:
    1. `cif-version`: a JSON string giving the minimum CIF syntax version required to express the contents of the object; currently `1.1` or `2.0` are available
    1. `schema-name`: always `CIF-JSON`
    1. `schema-version`: a JSON string giving the version of the CIF-JSON schema that this JSON object conforms to. CIF-JSON
    versioning follows [semantic versioning priniciples](http://semver.org/spec/v2.0.0.html).
    1. `schema-uri`: a URI for the CIF-JSON schema.
10. (Reserved names). All JSON names starting with an upper-case letter and appearing in JSON datablock 
objects are reserved for future development.

## Comments

1. Unlooped datavalues are treated as belonging to single-row
loops. Note that CIF syntax files allow similar behaviour for datanames defined by DDL2 
and DDLm dictionaries.
1. This specification contains some features to allow straightforward
conversion to JSON from files in CIF syntax. However, round-tripping
through CIF-JSON will not preserve features of CIF syntax that are not
used by CIF dictionaries. In particular, block and data name order, and
the type of delimiters used, if any, are not expressed in CIF-JSON. If
such high-fidelity transformation is required (for example, for CIF
syntax validation) the COD-JSON format used by the freely available
[COD tools](http://wiki.crystallography.net/cod-tools/) is
recommended.
1. JSON numbers are not used to represent CIF numbers as it is generally
impossible to determine whether 
or not an arbitrary data value appearing in a CIF syntax file is a number. Future
versions of this standard may offer a mechanism for selective encoding of CIF 
data values as numbers.
1.  When translating from CIF syntax files, there is no requirement to copy the
data block header names when creating the corresponding JSON item names, as
these header names have no significance in the CIF standards.  A conservative
approach would be to maintain a straightforward correspondence with the original CIF
file, perhaps even preserving the initial `data_` string for easy visual identification.
1.  Implementers should always check `schema-version` when reading CIF-JSON objects. 
Any changes in the major version number imply changes that may cause software written
against previous versions to function incorrectly.
1.  The single top-level item named 'CIF-JSON' is intended to allow rapid identification of
CIF-JSON files before the whole stream is parsed.
1.  Multiple top-level objects may be included in a single JSON array to facilitate
transmission of multiple CIF-JSON objects.

## Example

### CIF syntax file:


    #\#CIF_2.0
    data_example
      _dataname.a   syzygy
      _Flight.vector    [0.25 1.2(15) -0.01(12)]
      _dataname.table   {"save":222 "mode":full "url":"http:/bit.ly/2"}
      _Flight.Bearing  221.45(7)
      loop_
        _x.id
        _y
        _z
        _alpha
        1    4.23(14)     [a a a c]    1.5e-6(2)
        2   11.9(3)       [c a c a]    2.1e-6(11)
        3    0.2(4)       [b a a a]    0.0051(4)
        4     .              .             ?
        
      loop_
        _Q.key
        _Q.access
        xxp     {"s":2  "k":-5}
        yyx     {"s":1  "k":-2}
        
      _dataname.chapter   1.2
      _dataname.verylong
    ;<whatever>\\
    <whatever>This contains one very long line \
    <whatever>that we wrap around using the \
    <whatever>excellent CIF2 line expansion protocol.
    ;
 
     data_Another_Block
       _abc    xyz
       save_internal
          _abc   yzx
          loop_
             _r.fruit
             _r.colour
             apple    red
             pear     green
       save_

### Equivalent CIF-JSON:

```json
{"CIF-JSON":
    {"Metadata":{"cif-version":"2.0",
                 "schema-name":"CIF-JSON",
                 "schema-version":"1.0.0",
                 "schema-uri":"http://www.iucr.org/resources/cif/cif-json.txt"
                 },
     "example":
        {"_dataname.a":["syzygy"],
         "_flight.vector":["0.25","1.2(15)","-0.01(12)"],
         "_dataname.table":[{"save":"222", "mode":"full", "url":"http:/bit.ly/2"}],
         "_flight.bearing":["221.45(7)"],
         "_x.id":["1","2","3","4"],
         "_y":["4.23(14)","11.9(3)","0.2(4)",false],
         "_z":[["a","a","a","c"],
               ["c","a","c","a"],
               ["b","a","a","a"],
               false],
         "_alpha":["1.5e-6(2)","2.1e-6(11)","5.1e-3(4)",null],
         "_q.key":["xxp","yyx"],
         "_q.access":[{"s":"2",  "k":"-5"},{"s":"1",  "k":"-2"}],
         "_dataname.chapter":["1.2"],
         "_dataname.verylong":["This contains one very long line that we wrap around using the excellent CIF2 line expansion protocol."]
         },
     "another_block":{
        "_abc":["xyz"],
        "Frames":
           {"internal":{"_abc":["yzx"],
                        "_r.fruit":["apple","pear"],
                        "_r.colour":["red","green"]}
                        }
           }
    }
}
```
