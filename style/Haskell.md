Haskell Style Guide
===================

(Originally from [Johan Tibell's style guide][tibbe-style-guide], with
modifications from [the Snap framework's style
guide][snap-style-guide], additions from the [GHC Coding Style
Guidelines][ghc-style] and further inspiration from the style used at
[Well-Typed LLP][well-typed], as seen e.g. in
[distributed-process][distributed-process].)

This is a short document describing the preferred Haskell coding style
at Tweag I/O. When something isn't covered by this guide you should
stay consistent with the code in the other modules.

[tibbe-style-guide]:
http://github.com/tibbe/haskell-style-guide/blob/master/haskell-style.md
[snap-style-guide]:
http://snapframework.com/docs/style-guide
[ghc-style]:
http://hackage.haskell.org/trac/ghc/wiki/Commentary/CodingStyle
[well-typed]:
http://well-typed.com
[distributed-process]:
http://hackage.haskell.org/package/distributed-process

General Style
-------------

(section adopted from the [GHC Coding Style Guidelines][ghc-style].)

It's much better to write code that is transparent than to write code
that is short.

Often it's better to write out the code longhand than to reuse
a generic abstraction (not always, of course). Sometimes it's better
to duplicate some similar code than to try to construct an elaborate
generalisation with only two instances. Remember: other people have to
be able to quickly understand what you've done, and overuse of
abstractions just serves to obscure the really tricky stuff, and
there's no shortage of that in this project.

The general rule is to stick to the same coding style as is already
used in the file you're editing. If you must make stylistic changes,
commit them separately from functional changes, so that someone
looking back through the change logs can easily distinguish them.

Formatting
----------

### Line Length

Maximum line length is *80 characters*. Comments should be wrapped
accordingly. There should be no trailing whitespace anywhere in your
code. This makes git diff output look ugly and causes spurious merge
conflicts.

In Emacs, you can add the following code to your `init.el` file to
enforce this:

```elisp
(add-hook 'haskell-mode-hook (lambda () (set-fill-column 80)))
(add-hook 'haskell-mode-hook
          (lambda ()
             (add-hook 'before-save-hook 'delete-trailing-whitespace t t)))
```

### Indentation

Tabs are illegal. Use spaces for indenting. Use *2 spaces* for each
indentation level. The only exception is for code blocks inside
a definition, which should be indented with *4 spaces*. Indent the
`where` keyword two spaces to set it apart from the rest of the code
and indent the definitions in a `where` clause 2 spaces. Guards are
usually indented 2 spaces. Some examples:

```Haskell
sayHello :: IO ()
sayHello = do
    name <- getLine
    putStrLn $ greeting name
  where
    greeting name = "Hello, " ++ name ++ "!"

filter :: (a -> Bool) -> [a] -> [a]
filter _ [] = []
filter p (x:xs)
  | p x = x : filter p xs
  | otherwise = filter p xs
```

As a general rule, *indentation of a line should not depend on the
length of any identifier in preceding lines*, only on layout
constraints.

### Blank Lines

One blank line between top-level definitions. No blank lines between
type signatures and function definitions. Add one blank line between
functions in a type class instance declaration if the functions bodies
are large. Use your judgement.

### Whitespace

Surround binary operators with a single space on either side. Use your
better judgement for the insertion of spaces around arithmetic
operators but always be consistent about whitespace on either side of
a binary operator. Don't insert a space after a lambda.

Don't vertically align type signatures, patterns in function
declarations and comments. This just causes spurious merge conflicts.

### Data Declarations

Align the constructors in a data type definition. Example:

```Haskell
data Tree a
  = Branch !a !(Tree a) !(Tree a)
  | Leaf
```

Format records as follows:

```Haskell
data Person = Person
  { firstName :: !String  -- ^ First name
  , lastName :: !String  -- ^ Last name
  , age :: !Int -- ^ Age
  } deriving (Eq, Show)
```

### List Declarations

Align the elements in the list. Example:
```Haskell
exceptions =
    [ InvalidStatusCode
    , MissingContentHeader
    , InternalServerError
    ]
```

### Pragmas

Put pragmas immediately following the function they apply to.
Example:
```Haskell
id :: a -> a
id x = x
{-# INLINE id #-}
```
In the case of data type definitions you must put the pragma before
the type it applies to. Example:
```Haskell
data Array e = Array
  {-# UNPACK #-} !Int
  !ByteArray
```

### Hanging Lambdas

You may or may not indent the code following a "hanging" lambda. Use
your judgement. Some examples:

```Haskell
bar :: IO ()
bar =
    forM_ [1, 2, 3] $ \n -> do
      putStrLn "Here comes a number!"
      print n

foo :: IO ()
foo =
    alloca 10 $ \a ->
    alloca 20 $ \b ->
    cFunction a b
```

### Export Lists

Format export lists as follows:

```Haskell
module Data.Set
  ( -- * The @Set@ type
    Set
  , empty
  , singleton
    -- * Querying
  , member
  ) where
```

### If-then-else expressions

Generally, guards should be preferred over if-then-else expressions,
where possible. if-then-else is preferred to case analysis on
a boolean. Short cases should usually be put on a single line (when
line length allows it).

When writing non-monadic code (i.e. when not using `do`) and guards
can't be used, you can align if-then-else expressions like you would
normal expressions:

```Haskell
foo =
    if ...
    then ...
    else ...
```

In monadic code, so long as you use the Haskell 2010 dialect and
above, you can use the same alignment as above. A different alignment
rule for monadic code is no longer necessary.

### Case expressions

The alternatives in a case expression can be indented using either of
the two following styles:

```Haskell
foobar = case something of
    Just j  -> foo
    Nothing -> bar
```

or as

```Haskell
foobar =
    case something of
      Nothing -> foo
      Just j -> bar
```
Align the `->` arrows when it helps readability.

Imports
-------

Imports should be grouped in the following order:

1. local application/library specific imports
2. related third party imports
3. standard library imports

Put a blank line between each group of imports. The imports in each
group should be sorted alphabetically, by module name.

Always use explicit import lists or `qualified` imports for standard
and third party libraries. This makes the code more robust against
changes in these libraries. Exception: the Prelude.

Use your judgement when it comes to local application/library specific
imports. On the one hand, they make for more maintainable code because
identifiers that are removed from the imported module will be caught
early and identifiers added to the imported module do not risk
clashing with local identifiers. They also serve as documentation as
to which parts of a module are actually required.

However, explicit import lists are also much more verbose, and slow
down development. Moreover, in a collaborative environment, explicit
import lists can cause spurious conflicts, since two otherwise
unrelated changes to a file may both require changes to the same
import list.

The qualifier for well known modules, such as `ByteString` can be
shortened further, eg `BS`. But in general, prefer descriptive
qualifiers rather than one letter ones. For example

```Haskell
import qualified Data.Map as Map                  -- good
import qualified Data.Map as M                    -- not so good
```

Comments
--------

### Punctuation

Write proper sentences; start with a capital letter and use proper
punctuation.

### Top-Level Definitions

Comment every top level function (particularly exported functions),
and provide a type signature; use Haddock syntax in the comments.
Comment every exported data type. Function example:

```Haskell
-- | Send a message on a socket. The socket must be in a connected
-- state. Returns the number of bytes sent. Applications are
-- responsible for ensuring that all data has been sent.
send :: Socket      -- ^ Connected socket
     -> ByteString  -- ^ Data to send
     -> IO Int      -- ^ Bytes sent
```
For functions the documentation should give enough information to
apply the function without looking at the function's definition.

Record example:
```Haskell
-- | Bla bla bla.
data Person = Person
  { age :: !Int -- ^ Age
  , name :: !String -- ^ First name
  }
```
For fields that require longer comments format them like so:

```Haskell
data Record = Record
  { -- | This is a very very very long comment that is split over
    -- multiple lines.
    field1 :: !Text
    -- | This is a second very very very long comment that is split
    -- over multiple lines.
  , field2 :: !Int
  }
```
### End-of-Line Comments

Separate end-of-line comments from the code using *at least* 2 spaces.
Your editor might let you set a preferred column for end-of-line
comments. Align comments for data type definitions. Some examples:
```Haskell
data Parser = Parser
  !Int                                  -- Current position
  !ByteString                           -- Remaining input

foo :: Int -> Int
foo n = salt * 32 + 9
  where
    salt = 453645243                      -- Magic hash salt.
```
### Links

Use in-line links economically. You are encouraged to add links for
API names. It is not necessary to add links for all API names in
a Haddock comment. We therefore recommend adding a link to an API name
if:

* The user might actually want to click on it for more information (in
  your judgment), and

* Only for the first occurrence of each API name in the comment (don't
  bother repeating a link)

Naming
------

Use mixed-case when naming functions and camel-case when naming data
types.

For readability reasons, don't capitalize all letters when using an
abbreviation. For example, write `HttpServer` instead of `HTTPServer`.
Exception: Two letter abbreviations, e.g. `IO`.

### Records

Where appropriate, add an unabbreviated prefix to the name of record
fields. Example:

```Haskell
-- | Messages consist of their typeRep fingerprint and their encoding
data Message = Message
  { messageFingerprint :: !Fingerprint
  , messageEncoding :: !BSL.ByteString
  }
```

This is not necessary for modules that export only one data type *and*
are meant to be imported qualified.

### Modules

Use singular when naming modules e.g. use `Data.Map` and
`Data.ByteString.Internal` instead of `Data.Maps` and
`Data.ByteString.Internals`.

Functions
---------

Use of catamorphisms (e.g. `foldr`, `foldl`) and other special case
higher-order functions (such as `map`) is preferable to writing
structurally recursive functions with explicit recursion. This is
because a fold makes it a type error to forget the base case -
a problem with the code that could otherwise go undetected. It also
documents the fact that the function is supposed to be structurally
recursive, as opposed to functions that use general recursion.

Dealing with laziness
---------------------

By default, use strict data types and lazy functions.

### Data types

Constructor fields should be strict, unless there's an explicit reason
to make them lazy. This avoids many common pitfalls caused by too much
laziness and reduces the number of brain cycles the programmer has to
spend thinking about evaluation order.

```Haskell
-- Good
data Point = Point
  { pointX :: !Double  -- ^ X coordinate
  , pointY :: !Double  -- ^ Y coordinate
  }

-- Bad
data Point = Point
  { pointX :: Double  -- ^ X coordinate
  , pointY :: Double  -- ^ Y coordinate
  }
```

Additionally, unpacking simple fields often improves performance and
reduces memory usage:

```Haskell
data Point = Point
  { pointX :: {-# UNPACK #-} !Double  -- ^ X coordinate
  , pointY :: {-# UNPACK #-} !Double  -- ^ Y coordinate
  }
```
As an alternative to the `UNPACK` pragma, you can put
```Haskell
{-# OPTIONS_GHC -funbox-strict-fields #-}
```
at the top of the file. Including this flag in the file inself instead
of e.g. in the Cabal file is preferable as the optimization will be
applied even if someone compiles the file using other means (i.e. the
optimization is attached to the source code it belongs to).

Note that `-funbox-strict-fields` applies to all strict fields, not
just small fields (e.g. `Double` or `Int`). If you're using GHC 7.4 or
later you can use `NOUNPACK` to selectively opt-out for the unpacking
enabled by `-funbox-strict-fields`.

### Functions

Have function arguments be lazy unless you explicitly need them to be
strict.

The most common case when you need strict function arguments is in
recursion with an accumulator:

```Haskell
mysum :: [Int] -> Int
mysum = go 0
  where
    go !acc []    = acc
    go acc (x:xs) = go (acc + x) xs
```

Misc
----

### Warnings

Code should be compilable with `-Wall -Werror`, i.e. there should be
no warnings.

### Debug facilities

Use of `unsafePerformIO`-based debugging facilites, such as `Debug.Trace` should not be
committed to any stable branches. If you need logging, use a proper method and ship it,
so that further development benefits from your debugging efforts.

### Lenses

Where appropriate, define lenses for all fields in a record. Use the
`_` prefix to name fields. When using the [lens package][lens], use
`makeClassy` [where possible][lens-makeClassy] to generate lenses
rather than `makeLenses`. This is to make it easy to export all lenses
for a record all at once.

```Haskell
module Person
  ( Person(..)
  , HasPerson(..)
  ) where

data Person = Person
  { _firstName :: !String  -- ^ First name
  , _lastName  :: !String  -- ^ Last name
  , _age       :: !Int     -- ^ Age
  } deriving (Eq, Show)

makeClassy ''Person
```

For consistency, if a record has lenses defined, always use the lens
to get or set the field of a record (rather than `_fieldName`). Field
names should only be used to initialize the record.

[lens]: http://hackage.haskell.org/package/lens
[lens-makeClassy]: http://hackage.haskell.org/package/lens-4.3.3/docs/Control-Lens-TH.html#v:makeClassy
