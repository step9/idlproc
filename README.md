#IDLproc

##Usage

``
idlproc language [options] [--] infile [outfile]
``

`language` specifies the consumer to use and must be given as first argument. Supported: `Pascal`.

`infile` is the first input file.

`outfile` may be given as a file or directory name. If not given, the current directory is assumed. Giving a file name is only valid if `-D` was not used.

### Options

* `-D --dependencies`
  Follow dependencies via `#include` directives
  

##IDL Dialect
IDL2pas uses a restricted version of MIDL with some features added from XPIDL.

### Base Types
The following data types are considered "predefined" and guaranteed to have representations in all consumers.
Const-, Reference- or Pointer-ness that is caused by direction modifiers (see Methods below) are **not** part of the type itself, but instead are generated by the consumer. Interface types are always considered to be passed as instance pointers. Note that this is in contrast to MIDL.

| Type Name(s) | C++ Equivalent | ObjFPC Equivalent |
|--------------|----------------|----------------|
| `Boolean`              | `bool`       | `Boolean`       |
| `UInt8`                | `unsigned char`       | `Byte`       |
| `UInt16`               | `unsigned short`       | `Word`       |
| `UInt32`               | `unsigned long`       | `DWORD`       |
| `UInt64`               | `unsigned long long`       | `QWORD`       |
| `Int8`                 | `char`       | `Smallint`       |
| `Int16`                | `short`       | `Shortint`       |
| `Int32`                | `long`       | `Integer`       |
| `Int64`                | `long long`       | `Int64`       |
| `Pointer`              | `void *`       | `Pointer`       |
| `PtrInt`               | `size_t`       | `PtrInt`       |
| `PtrUInt`              | `usize_t`       | `PtrUInt`       |
| `CString`              | `char *` (default system codepage)      | `PAnsiChar`       |
| `CUTF8String`          | `char *` (encoded as UTF8)      | `PUtf8Char`       |
| `CWString`             | `wchar_t *` (arbitrary encoding)      | `PUnicodeChar`       |
| `IID`                  | `REFIID`      | `TGUID`       |

###Non-Declarative

* `#include "file.idl"`
  Read data from `file.idl` next, return to next line in current file afterwards. Relative paths to the current file are allowed. May occur anywhere a token would be allowed, but must end on a line end.

* Language-specific extensions:
  ```
  %{[lang]
  (language-specific code here)
  %}
  ```
  Emit code verbatim. Optionally give lang on first line to only emit for a specific language consumer. May only occur as top-level declaration.

##Declarations
Declarations may optioanlly be wrapped in a MSIDL-Typelib-`library` definition, but this is not required.

Many Declarations accept Attributes specified before them, they are not spelled in syntax examples here for brevity. Relevant Attributes are given in the Declaration descriptions below.

* Interface
  ```IDL
  interface Name
  {
    [Declarations]
  };
  ```
  Attributes:
  * `forward`: Marks forward declaration, full declaration follows later.
  * `uuid(IID)`: Interface GUID, *required* unless `forward` is set.
  Contents:
  * `Method`
  * `Property`

* Module
  ```IDL
  module Name
  {
    [Declarations]
  };
  ```
  Attributes:
  * `dllname("filename.dll")`: DLL (or SO) to import functions from
  Contents:
  * `Constant`
  * `Method` (will be translated as static imports)

* Alias
  ```IDL
  typedef TypeSpec Name; /* See "Array types" below */
  ```
  Attributes:

* Enum
  ```IDL
  typedef enum
  {
    EnumItem,
    EnumItem = Number
  } Name;
  ```
  Attributes:
  * `enumsize(number)`: byte size of the enum, used to enforce packing

* Struct
  ```IDL
  typedef struct
  {
    TypeSpec Name; /* See "Array types" below */
  } Name;
  ```
  Attributes:
  Contents:
  * `Field`, see above

* Method
  ```IDL
  TypeSpec Name ( [ Parameter [, Parameter] ]);
  Parameter:
      [Attribute] [in|out|inout] TypeSpec Name /* See "Array types" below */
  ```
  Attributes for Parameters:
  * `const`: compiler hint that the parameter will not be modified. Note that this is different from the `const` that may be part of the TypeSpec proper.

  Parameters are by default passed by value. The `[const]` attribute does not change that, it only allows optimized checking for the consuming compiler. The direction modifiers allow changing this:
  
  |         | without `[const]` | with `[const]` |
  |---------|-------------------|----------------|
  | nothing | COM standard      | const          |
  | `in`    | COM standard      | constref       |
  | `out`   | by reference, input value is never used                    | (invalid)      |
  | `inout` | by reference      | (invalid)      |

* Property
  ```IDL
  [readonly] attribute TypeSpec Name;
  ```
  Implies 2 Methods: `TypeSpec GetName()` and `SetName(TypeSpec newval)` (in order). Setter is only generated if `readonly` prefix is not given.


* Array Types
  ```IDL
  TypeSpec Name; /* No Array */
  TypeSpec Name[5]; /* Array with 5 elements, index 0..4 */
  TypeSpec Name[]; /* Dynamic Array (definition as pointer type is preferred for portability) */
  TypeSpec Name[5][4]; /* array[0..4] of array[0..3] of TypeSpec */
  ```

* Callback
  ```IDL
  typedef callback TypeSpec Name ( [ Parameter [, Parameter] ]);
  ```

  Function pointer commonly used for callbacks. Identical to Method decalaration, see above for full description. Note that in constrast to C(++), the pointer is implied and not attached to `Name`.

* Const
  ```IDL
  const TypeSpec Name = Value;
  ```
  Attributes for Parameters:
  * `static_cast`: Hint that the literal `Value` is not directly compatible to `TypeSpec` and may need an explicite type cast.


##Refereces

- [TP Lex & Yacc](http://www.musikwissenschaft.uni-mainz.de/~ag/tply/)
- [Plax & Pyacc Howto](http://wiki.freepascal.org/Plex_and_Pyacc)
- [Freepascal h2pas utility](https://github.com/graemeg/freepascal/tree/master/utils/h2pas)
- [XPIDL Data types](https://developer.mozilla.org/en-US/docs/Mozilla/XPIDL)
- [XPIDL BFN Syntax](https://developer.mozilla.org/en-US/docs/Mozilla/XPIDL/Syntax)
- [MIDL Data types](https://msdn.microsoft.com/en-us/library/windows/desktop/aa367090%28v=vs.85%29.aspx)
- [MIDL Token reference](https://msdn.microsoft.com/en-us/library/windows/desktop/aa367088%28v=vs.85%29.aspx)
- [Inprise RIDL Syntax Examples](http://docwiki.embarcadero.com/RADStudio/Berlin/en/Using_Object_Pascal_or_RIDL_Syntax)











