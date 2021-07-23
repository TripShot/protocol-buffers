[![Build Status](https://github.com/k-bx/protocol-buffers/actions/workflows/haskell-ci.yml/badge.svg)](https://github.com/k-bx/protocol-buffers/actions/workflows/haskell-ci.yml) Haskell Protocol Buffers
====================================================

This the README file for `protocol-buffers`,
`protocol-buffers-descriptors`, and `hprotoc`. These are three
interdependent Haskell packages originally written by Chris Kuklewicz.

Currently, maintainership was taken by Timo von Holtz. It is
planned to only support GHC 8.0 and newer unless someone explicitly
asks for support of earlier versions.

(Needs check) This README was updated most recently to reflect version
`2.0.7`. This code should be compatible with Google protobuf version
`2.3.0`. Changes to keep up with Google protobuf version `2.4.0` are
being considered.

What is this for? What does it do? Why?
---------------------------------------

It is a pure Haskell re-implementation of the Google code at
https://developers.google.com/protocol-buffers/docs/overview which is
"...a language-neutral, platform-neutral, extensible way of
serializing structured data for use in communications protocols, data
storage, and more."  Google's project produces C++, Java, and Python
code.  This one produces Haskell code.

How well does this Haskell package duplicate Google's project?
--------------------------------------------------------------

- This provides non-mutable messages that ought to be wire-compatible
  with Google.

- These messages support extensions.

- These messages support unknown fields if hprotoc is passed the
  proper flag (-u or --unknown_fields).

- ~~This does not generate anything for Services/Methods.~~

- ~~Adding support for services has not been considered.~~

I think that Google's code checks for some policy violations that are
not well documented enough for me to reverse engineer. Some (all?) of
Google's APIs include the possibility of mutable messages. I suspect
that my message reflection is not as useful at runtime as in some of
Google's APIs.

What is protocol-buffers?
-------------------------

The protocol-buffers part is the main library which has two faces:

1. It provides an external API exported by module
`Text.ProtocolBuffers` for users to read and write the binary format
and manipulate the message data structures created by hprotoc.

2. It provides an internal API for the messages under module
`Text.ProtocolBuffers.Header` to implement their tasks.

What is protocol-buffers-descriptor?
------------------------------------

- It uses the `protocol-buffers` package.

- It provides the code generated by hprotoc from `descriptor.proto`
under module `Text.DescriptorProtos`.

- This supports hprotoc which is used to describe proto files and the
code they will generate.

- It provides `Text.DescriptorProtos.Options` which help in looking up
the new style custom options.

What is hprotoc?
----------------

- It uses `protocol-buffers` and `protocol-buffers-descriptor` above.

- It is a command line tool that reads in `.proto` files and produces
  Haskell source trees like Google's protoc.

- ...and it contains a very nice lexer and parser for the `.proto`
  file...

The hprotoc part is a executable program which reads `.proto` files
and uses the `protocol-buffers` package to produce a tree of Haskell
source files.  The program is called `hprotoc`.  Usage is given by the
program itself, the options themselves are processed in order.  It can
take several input search paths, and allow an additional module
prefix, a selectable output directory, and ends with a list of of
proto file to generate from.

The output has to be a tree of modules since each message is given its
own namespace, and a module is the only partitioning of namespace in
Haskell.  The keys for extension fields are defined alongside the
message whose namespace they share.  Since message names are both a
data type and a namespace the filename and the message name match
(aside from the .hs file extension).

And what are the examples and tests sub-directories?
----------------------------------------------------

The examples sub-directory is for duplicating the `addressbook.proto`
example that Google has with its code.  The `ABF` and `ABF2` file are
included as binary addressbooks.  These can be read by the C++
examples from Google, and vice-versa.

The `tests` sub-directory is where I have written some test code to
drive the `UnittestProto` code generated from Google's
`unittest.proto` (and `unittest_import.proto`) files.  The `patchBoot`
file has the needed file patches to fix up the recursive imports (no
longer needed!).

What do I need to compile the code?
-----------------------------------

1. Install [Haskell Stack Tool](https://github.com/commercialhaskell/stack/)
2. Run `stack build`

Alternatively, go with old-fashioned `cabal build`.

How mature is this code?
------------------------

It can write the wire encoding and read it back.  It has been tested
for interoperability against Google's read/write code with
`addressbook.proto`.

`hprotoc` generates and uses the `Text.DescriptorProtos` tree from
Google `descriptor.proto` file.

`hprotoc` has generated code from `Google/protobuf/unittest.proto` and
`Google/protobuf.unittest_import`. These compile after adding hs-boot
files `TestAllExtensions.hs-boot`, `TestFieldOrderings.hs-boot`, and
`TestMutualRecursionA.hs-boot` to resolve mutual recursion. The
`TestEnumWithDupValue` has duplicated values which cause a compilation
warning.

There has been QuickCheck tests done for
`UnittestProto/TestAllType.hs` and
`UnittestProto/TestAllExtensions.hs` in the tests subdirectory. These
pass as of `2008-09-19` for version `0.2.7`. These test that random
messages can be roundtripped to the wire format without changing —
with the caveat that the new extension keys are read back as raw bytes
but compare equal because of the parsing done by (==).

Mutual recursion is a problem?
------------------------------

Not using ghc.  The haskell-src-exts let me generate code with `{-#
SOURCE #-}` annotated imports.  And `hprotoc` generates the needed
hs-boot files for ghc.  And key import cycles are broken by creating
`Key.hs` files, which users can ignore.

How stable is the API?
----------------------

This is the first working release of the code.  I do not promise to
keep any of the API but I am lazy so most things will not change. The
reflection capabilities may get improved/altered. Stricter warnings
and error detection may be added.  Code will move between
protocol-buffers and hprotoc projects.  The internals of reading from
the wire may be improved.

Where is the API documentation?
-------------------------------

Generate haddock with `stack haddock` command.

You can also view API documentation online at
[Hackage page](https://hackage.haskell.org/package/protocol-buffers).

The imports of `Text.ProtocolBuffers` are the public API. The
generated code's API is `Text.ProtocolBuffers.Header`. The only usage
examples are in the `examples` sub-directory and the `tests`
sub-directory. Since the messages are simply Haskell data types most
of the manipulation should be easy.

The main thing that is weird is that messages with extension ranges
get an ExtField record field that holds ... an internal data
structure.  This is currently a `Map` from field number to a rather
complicated existential + GADT combination that should really only be
touched by the `ExtKey` and `MessageAPI` type class methods. The
`ExtField` data constructor is not hidden, though it could be and
probably ought to be.

Note that extension fields are inherently slower, especially in ghci
(though ghc's `-O2` helps quite a bit).

The entire proto file is stored in the top level module in
wire-encoded form and can be accessed as a `FileDescriptorProto`. The
Haskell code also defines its own reflection data types, with one
stored in each generated module and also in a master data type in the
top level module (via `Show` and `Read`).

Who reads this far?
-------------------

I suspect no one ever will.

Why define your own Haskell reflection types in addition to
`FileDescriptorProto`s types?

This allows for the protocol-buffers library package to not depend on
a single thing defined in the `protocol-buffers-descriptor` package.
This lack of recursion made for much simpler bootstrapping and allows
the `descriptor.proto` generated files to be build separately.

While `descriptor.proto` files are a great fit as output from parsing
a proto file they are not as good a fit for code generation. They mix
fields and extension keys, they have all optional fields even though
some things (especially names) are compulsory. They obscure which
descriptors are groups. They have a nested structure which is useful
when resolving the names but not for iterating over for code
generation.

What are the pieces of protocol-buffers doing?
----------------------------------------------

- `Basic.hs` defines the core data types (that are not already in
  `Prelude`) and many classes.
- `Mergeable.hs` defines the standard instances of `Mergeable` for
  combining types.
- `Default.hs` defines the standard default of the basic data types.
- `Reflections.hs` defines the Haskell reflection data types (stored
  with each generated module).
- `Get.hs` is here because I needed a slightly different style of
  binary `Get` monad (see binary and binary-strict packages). This is
  standalone and could be put into any project. It has long comments
  inside.
- `WireMessage.hs` defines 3 things:
  1. The Wire instances for the basic data types
  2. The API for the generated module to use to define their own Wire
     instances
  3. The API for the user to load and save messages This file would
     not compile with ghc-6.8.3 on a G4 (Mac OS X 10.5.4, XCode 3.1)
     without -fvia-C as the cabal file states.
- `Extensions.hs` is rather large because it add everything needed for
  extension fields (see haddock API docs).  It should not export
  ExtField's constructor, but it currently does.
- `Header.hs` re-exports what is needed for the instance messages.
- `ProtocolBuffer.hs` re-exports what is needed for the user API.

What are the pieces of hprotoc doing?
-------------------------------------

`alex` uses `Lexer.x` to generated `Lexer.hs` which slices up the
`.proto` file into tokens.  The `.proto` layout is well designed,
quite unambiguous, and easy to tokenize.  The lexer also does the jobs
of decoding the backslash escape codes in quotes strings, and
interpreting floating point numbers. Errors and unexpected input are
inserted into the token list, with at least line number level
precision.

The `Parser.hs` file has a `Parsec` parser which are really used as
nested parsers (allowing for the type of the user state to change).
The `.proto` grammar is well designed and the system never needs to
backtrack over tokens. The default values and options' values parsed
according to the expected type, and string default are check for valid
utf8 encoding.  (This also import the `Instances.hs` file)

The `Resolve.hs` has code to resolve all the names to a fully
qualified form, including name mangling where necessary. This includes
code to load and parse all the imported `.proto` files, reusing parses
for efficiency, and detecting import loops. The context built from
each imported file is combined to change the `FileDescriptorProto`
into a modified `FileDescriptorProto`. This stage also determines that
extension keys are in a valid extensions range declaration, and enum
default values exists.

The `MakeReflections.hs` file converts the nested `FileDescriptorProto`
into a flatter Haskell reflection data structure. This includes
parsing the default value stored in the `FileDescriptorProto`.

The `BreakRecursion.hs` file builds graphs describing the imports and
works out whether and how to create hs-boot and `Key.hs` files to allow
allow for warning-free compilation with ghc (as of 6.10.1).

The `Gen.hs` file takes a Haskell data structure from
`MakeReflections` and builds a module syntax data structure.  The
syntax data is quite verbose and several helper functions are used to
help with the composition.  The result is easy to print as a string to
a file.

The `ProtoCompile.hs` file is the Main module which defines the
command line program `hprotoc`.  This manages most of the interaction
with the file system (aside from import loading in Resolve).
Everything that is needed is collected into the Options data type
which is passed to "run". The output style can be tweaked by changing
"style" and "myMode".

New oneof implementation
------------------------

Since `protocol-buffers` version 2.6, the upstream `protocol-buffers`
supports [oneof](https://developers.google.com/protocol-buffers/docs/proto?hl=en#oneof)
keyword, which is a union of different data types.
It is very natural to combine the oneof specification into Haskell
ADT, so we implement the feature.

In `hprotoc/oneoftest`, we have an example for this. `school.proto` defines
a collection of members in a school, which is organized into dormitories.
Each member should have common attributes like `id` and `name`, but there are
attributes only specific to students, faculties or administrators.

Therefore, we define property as a oneof field which is one of student, faculty
and admin type. How it is defined should be easily understood from `school.proto`.

Once `protocol-buffers` is installed, using `hprotoc`, we can generate Haskell
source codes. Assuming we run `hprotoc` on the `oneoftest` directory and generate
source code in `hs` directory:

```
oneoftest> hprotoc --proto_path=. --haskell_out=hs school.proto
```

We will have `School.Member` module which defines `Member` by
(I omit qualifier and strictness annotation here.)
```
data Member = Member { id :: Int32
                     , name :: Utf8
                     , property :: Maybe Property
                     }
```
and `School.Member.Property` module defines `Property` (as `oneof`) by
```
data Property = Prop_student {prop_student :: Student }
              | Prop_faculty {prop_faculty :: Faculty }
              | Prop_admin   {prop_admin   :: Admin   }
```
where `Student`, `Faculty` and `Admin` are defined as ordinary nested message
data types in separate modules, respectively. Therefore, the `oneof` feature
is smoothly matched with Haskell sum types. Note that `Maybe` will be always
present for a `oneof` field in the owner data type definition (Here, `Maybe Property`
in the definition of `Member`). This is because of compatibility with other language
implementations that treat `oneof` as a collection of `optional` fields.

In the `oneoftest` directory, we provides a Haskell example in `hprotoc/oneoftest/hs`,
modification of the previous example using lenses in `hprotoc/oneoftest/hs-lens`
and a C++ example in `hprotoc/oneoftest/cpp` to demonstrate how to use. Each example
has `encode` and `decode`. With `encode`, we start from data in memory and generate
a serialized binary file in wire format. Then,`decode` takes the file and present
some information to prove it successfully decoded the binary. One can encode from
Haskell side and decode on C++ side, or vice versa. For building, simply run
`build.sh` in each of `hs` or `cpp` directories. Example with lenses has a `stack.yml`
in it so it could be easily built with `stack build`. For C++, one must previously
install C++ `protobuf` library and `pkg-config`. We assume that `hprotoc` was already
executed as shown above. The version with lenses requires slightly different command:

```
oneoftest> hprotoc --proto_path=. --haskell_out=hs-lens/src school.proto
```

Here are examples.
```
hs >   ./encode serialized.dat
hs >   cd ../cpp
cpp>   ./decode ../hs/serialized.dat
name: "Gryffindor"
members {
  id: 1
  name: "Albus Dumbledore"
  prop_faculty {
    subject: "allmighty"
    title: "headmaster"
  }
}
members {
  id: 2
  name: "Harry Potter"
  prop_student {
    grade: 5
    specialty: "defense of dark arts"
  }
}
```

```
cpp>   ./encode serialized.dat
cpp>   cd ../hs
hs >   ./decode ../cpp/serialized.dat

Right (Dormitory {name = "Gryffindor", members = fromList [Member {id = 1, name = "Albus Dumbledore", property = Just (Prop_faculty {prop_faculty = Faculty {subject = "allmighty", title = Just "headmaster", duty = fromList []}})},Member {id = 2, name = "Harry Potter", property = Just (Prop_student {prop_student = Student {grade = 5, specialty = Just "defense of dark arts"}})}]},"")
```

New code generation for services
--------------------------------

`protocol-buffers` support the definition and generation of RPC services. Though
without specifying a specific transport mechanism. Meaning only stubs are generated.
True to this motto `hprotoc` generates transport agnostic interfaces for services.

Consider a `Search.proto` definition like the following:

````protobuf
message SearchRequest {
    required string query = 1;
    optional int32 page_number = 2;
    optional int32 result_per_page = 3;
}

message SearchResponse {
    repeated string results = 1;
}

message AutocompleteRequest {
    required string query = 1;
    optional int32 max_results = 2;
}

message AutocompleteResponse {
    repeated string results = 1;
}

service SearchService {
    rpc Search (SearchRequest) returns (SearchResponse);
    rpc Autocomplete(AutocompleteRequest) returns (AutocompleteResponse);
}
````

We define two request-response pairs `SearchRequest`, `SearchResponse` and
`AutocompleteRequest`, `AutocompleteResponse`  and a service definition
`SearchService`. The service has two methods `Search` and `Autocomplete`.
Each taking one of the request-response pair as parameter respectively as
output.

For `SearchService` it generates a module roughly looking like this:
(omitting imports, exports and code for other messages):

````haskell
type SearchService = P'.Service '[Search, Autocomplete]

searchService :: SearchService
searchService = P'.Service

type Search = P'.Method ".Search.SearchService.Search" SearchRequest SearchResponse

type Autocomplete = P'.Method ".Search.SearchService.Autocomplete" AutocompleteRequest AutocompleteResponse

search :: Search
search = P'.Method

autocomplete :: Autocomplete
autocomplete = P'.Method
````

A translated service consists of a type-level list itself consisting of `Method` types.
Each `Method` is parameterized with its method name as type-level string (via `DataKinds` extension),
its input parameter type and its output parameter type.

For every RPC method there will be one type alias for some parameterized `Method` type. For convenience `hprotoc`
will generate proxy functions (`searchService`, `search` and `autocomplete`) so
users won't have to tinker with proxying their types manually.

As the `protocol-buffer` does not specify any transport mechanism for services implementors
have to build the transports on their own.
