Network Working Group                                          D. Hable
Request for Comments: 9999                              
Category: Standards Track                                  April 10 2026


         HEYAAAAML: Human-Enjoyable Yet Another Agonizingly
              Agonizingly Agonizingly Markup Language
                   (Configuration File Format)

Status of This Memo

   This document specifies a configuration file format for the Internet
   community.  Distribution of this memo is unlimited.  The authors
   accept no liability for late-night debugging sessions caused by
   indentation errors in competing formats.

Abstract

   This document defines HEYAAAAML (pronounced "hey-AH-mul"), a
   human-first configuration file format.  HEYAAAAML occupies the
   space between formats that are too terse for humans (JSON) and too
   clever for parsers (YAML).  It is designed with the conviction that
   configuration files are read by humans roughly one hundred times
   for every one time they are written, and that the experience of
   reading one should not require a therapy session afterward.

   HEYAAAAML supports typed scalar values, nested sections, lists,
   inline comments, and multi-line strings, all without relying on
   significant whitespace for semantic meaning.

Table of Contents

   1.  Introduction
   2.  Conventions and Terminology
   3.  Design Principles
   4.  File Extension and MIME Type
   5.  Character Encoding
   6.  Lexical Structure
   7.  Data Types
   8.  Sections and Nesting
   9.  Lists
   10. Multi-line Strings
   11. Includes
   12. Comments
   13. The "a" Rule (Informative)
   14. Formal Grammar (ABNF)
   15. Comparison with Existing Formats
   16. Security Considerations
   17. Examples

1.  Introduction

   The landscape of configuration file formats is rich with options
   that each make reasonable tradeoffs:

      - JSON is ubiquitous but forbids comments, demands quoting of
        all keys, and trails commas like breadcrumbs through a forest.

      - YAML is human-readable until it interprets "no" as a boolean,
        "3.10" as a float, and the country code for Norway as false.

      - TOML is thoughtful but deeply nested structures become
        unwieldy, and its table syntax has been known to cause
        spontaneous whiteboard sessions.

      - INI files are simple but too simple, like bringing a spoon
        to a sword fight.

   HEYAAAAML takes the position that a configuration format should be:

      (a) Unambiguous without requiring a PhD in parser theory.
      (b) Commentable, because "why" matters more than "what."
      (c) Typeful without type coercion surprises.
      (d) Nestable without becoming unreadable.
      (e) Fun to say out loud in a meeting.

   The name itself is an acronym: Human-Enjoyable Yet Another
   Agonizingly Agonizingly Agonizingly Markup Language.  The three
   repetitions of "Agonizingly" reflect the three existing formats
   (JSON, YAML, TOML) that inspired this work through a mixture of
   admiration and frustration.

2.  Conventions and Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in
   this document are to be interpreted as described in RFC 2119.

   The key word "SERIOUSLY" is to be interpreted as the author
   expressing mild exasperation.

   "Value"     - A typed datum: string, integer, float, boolean, or
                  null.
   "Key"       - A bare or quoted identifier that names a value.
   "Section"   - A named group of key-value pairs, delimited by
                  braces.
   "Document"  - A complete HEYAAAAML file constituting one
                  configuration unit.

3.  Design Principles

   3.1.  Explicit Delimiters Over Significant Whitespace

      Nesting is indicated by curly braces `{` and `}`, never by
      indentation level.  Whitespace (spaces, tabs, newlines) is
      insignificant outside of quoted strings.

      Rationale: An invisible character SHOULD NOT determine whether
      your database connects to production or staging.

   3.2.  Explicit Types Over Coercion

      Values are never silently coerced.  The string "true" is a
      string.  The boolean true is a boolean.  The country code "NO"
      is always a string.

   3.3.  Comments Are First-Class Citizens

      Comments can appear anywhere whitespace can appear.  They are
      introduced with `#` and extend to the end of the line.

   3.4.  One Obvious Way to Do Each Thing

      There is one syntax for strings, one syntax for sections, and
      one syntax for lists.  "One" is the number HEYAAAAML is most
      fond of.

4.  File Extension and MIME Type

   File Extension:   .hml

   MIME Type:        application/heyaaaaml

   Implementations MAY also recognize the extension .heyaaaaml for
   users who enjoy typing.

5.  Character Encoding

   HEYAAAAML documents MUST be encoded in UTF-8.  A byte order mark
   (BOM) at the start of the file MUST be silently ignored by parsers,
   though its presence SHOULD trigger a disappointed sigh.

6.  Lexical Structure

   6.1.  Keys

      A key is either a bare key or a quoted key.

      Bare keys may contain ASCII letters, digits, hyphens, and
      underscores:

         server-name = "atlas"
         max_retries = 5

      Quoted keys may contain any Unicode character and are enclosed
      in double quotes:

         "content-type" = "application/json"
         "emoji-key-🔑" = "yes, really"

      Keys MUST NOT be empty.  An empty key is not a key; it is a
      cry for help.

   6.2.  Assignment

      Keys and values are separated by the equals sign `=`:

         key = value

      Whitespace around the equals sign is OPTIONAL but RECOMMENDED
      for readability.

   6.3.  Terminators

      Each key-value pair is terminated by a newline or semicolon.
      Semicolons allow multiple assignments on one line:

         host = "localhost"; port = 5432

      Trailing semicolons are permitted and ignored.

7.  Data Types

   7.1.  Strings

      Strings are enclosed in double quotes.  Standard escape
      sequences are supported:

         \\    backslash
         \"    double quote
         \n    newline
         \t    tab
         \r    carriage return
         \uXXXX    Unicode code point (4 hex digits)
         \UXXXXXXXX    Unicode code point (8 hex digits)

      Example:

         greeting = "Hello, world!\n"

   7.2.  Integers

      Integers are written in decimal, hexadecimal (0x), octal (0o),
      or binary (0b) notation.  Underscores may be used as visual
      separators:

         count = 42
         mask = 0xFF_00
         perms = 0o755
         flags = 0b1010_0101

      Integers MUST be representable as signed 64-bit values.

   7.3.  Floats

      Floats follow IEEE 754 double-precision representation and use
      a decimal point and/or exponent notation:

         pi = 3.14159
         avogadro = 6.022e23

      Special float values:

         pos_inf = inf
         neg_inf = -inf
         not_a_number = nan

      Bare numeric-looking strings like "3.10" will NOT be
      interpreted as floats.  If you wrote quotes around it, you
      meant a string.  HEYAAAAML respects your intentions.

   7.4.  Booleans

      Booleans are the bare words `true` and `false`.  That is it.

      Specifically, the following are NOT booleans: yes, no, on, off,
      y, n, TRUE, FALSE, True, False, or any other creative spelling.
      Parsers encountering these bare words MUST raise a parse error
      with a helpful message suggesting `true` or `false`.

      SERIOUSLY: "no" is not a boolean.  It is a string that Norway
      did nothing to deserve.

   7.5.  Null

      The bare word `null` represents the absence of a value:

         middle_name = null

      Parsers MUST distinguish between a key set to null and a key
      that is absent from the document entirely.

8.  Sections and Nesting

   Sections group related key-value pairs and are denoted by a name
   followed by curly braces:

      database {
         host = "localhost"
         port = 5432
         credentials {
            user = "admin"
            password = "hunter2"   # we all saw that
         }
      }

   Sections may be nested to arbitrary depth.

   A dotted key syntax is available as shorthand for single values
   inside sections:

      database.host = "localhost"

   This is equivalent to:

      database {
         host = "localhost"
      }

   Dotted keys and explicit sections for the same path MAY coexist
   in a document; their contents are merged.  Duplicate keys within
   the same section MUST cause a parse error.

9.  Lists

   Lists are ordered sequences of values enclosed in square brackets:

      ports = [8080, 8443, 9090]

   Lists may contain values of mixed types (though this is
   DISCOURAGED in practice for the sanity of downstream consumers):

      mixed = [1, "two", true, null]

   Lists may span multiple lines:

      allowed_origins = [
         "https://example.com"
         "https://staging.example.com"
         "https://localhost:3000"
      ]

   Note: commas between list elements are OPTIONAL.  Newlines serve
   as implicit separators.  Trailing commas are permitted.

   Lists may contain nested lists and inline sections:

      servers = [
         { host = "alpha"; port = 80 }
         { host = "bravo"; port = 443 }
      ]

10. Multi-line Strings

   Multi-line strings are enclosed in triple double quotes:

      query = """
         SELECT *
         FROM users
         WHERE active = true
      """

   Leading whitespace is trimmed using the following algorithm:

      1. Discard the first line if it contains only whitespace after
         the opening `"""`.
      2. Determine the common leading whitespace of all non-empty
         lines.
      3. Remove that common prefix from each line.
      4. Discard the last line if it contains only whitespace before
         the closing `"""`.

   This allows multi-line strings to be indented naturally alongside
   the surrounding configuration without embedding unwanted leading
   spaces.

11. Includes

   A document may include another HEYAAAAML file:

      @include "path/to/other.hml"

   The path is resolved relative to the including file's directory.

   Includes are processed as textual insertion at parse time.  Circular
   includes MUST be detected and reported as an error.

   Implementations MAY support glob patterns:

      @include "conf.d/*.hml"

   Files matched by a glob MUST be included in lexicographic order to
   ensure deterministic behavior.

12. Comments

   Comments begin with `#` and continue to the end of the line:

      # This is a comment
      port = 8080  # This is an inline comment

   There are no block comments.  This is intentional.  Block comments
   lead to "commenting out" large sections of configuration, which
   leads to confusion about what is active, which leads to 3 AM
   pages.  Use version control instead.

13. The "a" Rule (Informative)

   This section is non-normative and exists purely as a cultural
   artifact of the format.

   Astute readers will have noted that the name "HEYAAAAML" contains
   exactly five letter a's.  By coincidence or design, the format has
   exactly five scalar data types:

      1. String       (a!)
      2. Integer      (aa!)
      3. Float        (aaa!)
      4. Boolean      (aaaa!)
      5. Null         (aaaaa!)

   Implementations are ENCOURAGED (but not required) to emit the
   appropriate exclamation in debug output when parsing each type.
   For example, a parser encountering a boolean might internally
   note: "aaaa! -- parsed boolean value."

   This behavior is OPTIONAL and has no semantic effect.  It does,
   however, make debug logs more entertaining.

14. Formal Grammar (ABNF)

   The following ABNF grammar describes the syntax of HEYAAAAML.
   It follows the conventions of RFC 5234.

   document       = *( statement / comment / ws )

   statement      = key-value / section / include

   key-value      = key ws "=" ws value terminator

   key            = bare-key / quoted-key
   bare-key       = 1*( ALPHA / DIGIT / "-" / "_" )
   quoted-key     = DQUOTE *quoted-char DQUOTE
   dotted-key     = key *( "." key )

   section        = bare-key ws "{" document "}"

   value          = string / integer / float / boolean / null
                  / list / inline-section

   string         = basic-string / multi-line-string
   basic-string   = DQUOTE *string-char DQUOTE
   multi-line-string = 3DQUOTE ml-body 3DQUOTE
   string-char    = %x20-21 / %x23-5B / %x5D-7E
                  / escape-seq / utf8-non-ascii
   escape-seq     = "\" ( "\" / DQUOTE / "n" / "t" / "r"
                  / "u" 4HEXDIG / "U" 8HEXDIG )

   integer        = ["-"] ( dec-int / hex-int / oct-int / bin-int )
   dec-int        = DIGIT *( DIGIT / "_" )
   hex-int        = "0x" HEXDIG *( HEXDIG / "_" )
   oct-int        = "0o" %x30-37 *( %x30-37 / "_" )
   bin-int        = "0b" BIT *( BIT / "_" )
   BIT            = "0" / "1"

   float          = ["-"] dec-int "." dec-int [ exponent ]
                  / ["-"] dec-int exponent
                  / "inf" / "-inf" / "nan"
   exponent       = ( "e" / "E" ) ["-" / "+"] dec-int

   boolean        = "true" / "false"

   null           = "null"

   list           = "[" ws *( value [ws ("," / ws)] ) ws "]"

   inline-section = "{" ws *( key-value [ws (";" / ws)] ) ws "}"

   include        = "@include" ws basic-string

   comment        = "#" *( %x20-7E / utf8-non-ascii )
   terminator     = newline / ";"
   ws             = *( SP / HTAB / newline )
   newline        = LF / CRLF

15. Comparison with Existing Formats

   +-------------------+------+------+------+-----+-----------+
   | Feature           | JSON | YAML | TOML | INI | HEYAAAAML |
   +-------------------+------+------+------+-----+-----------+
   | Comments          |  No  | Yes  | Yes  | Yes |    Yes    |
   | Explicit types    |  Yes |  No  | Yes  | No  |    Yes    |
   | No type coercion  |  Yes |  No  | Yes  | N/A |    Yes    |
   | Nested sections   |  Yes | Yes  | Ehh  | No  |    Yes    |
   | Multi-line strings|  No  | Yes  | Yes  | No  |    Yes    |
   | Trailing commas   |  No  |  --  | No   | --  |    Yes    |
   | File includes     |  No  |  No  | No   | Ext |    Yes    |
   | Whitespace-safe   |  Yes |  NO  | Yes  | Yes |    Yes    |
   | Fun to say        |  No  |  No  | Meh  | No  |    Yes    |
   +-------------------+------+------+------+-----+-----------+

16. Security Considerations

   16.1.  File Includes

      The @include directive reads files from the filesystem.
      Implementations MUST restrict include paths to prevent
      directory traversal attacks (e.g., @include "../../etc/passwd").
      Implementations SHOULD limit include depth to prevent denial of
      service via deeply recursive includes.  A limit of 64 levels
      is RECOMMENDED.

   16.2.  Resource Limits

      Parsers SHOULD impose limits on:
         - Maximum document size
         - Maximum nesting depth of sections
         - Maximum length of strings
         - Maximum number of keys

      These limits prevent denial-of-service via pathologically
      crafted documents.

   16.3.  Sensitive Values

      HEYAAAAML is a plain text format.  Sensitive values such as
      passwords, API keys, and secrets SHOULD NOT be stored directly.
      Implementations are ENCOURAGED to support environment variable
      references or external secret store integration as an extension.

      The example in Section 8 uses "hunter2" as a password.  Do not
      do this.  We know you know.  We know you know we know.

17. Examples

   17.1.  Web Application Configuration

      # heyaaaaml web app config
      app-name = "my-cool-service"
      version = "2.1.0"
      debug = false

      server {
         host = "0.0.0.0"
         port = 8080
         workers = 4

         tls {
            enabled = true
            cert = "/etc/ssl/cert.pem"
            key = "/etc/ssl/key.pem"
         }
      }

      database {
         url = "postgres://localhost:5432/mydb"
         pool-size = 20
         timeout = 30
      }

      logging {
         level = "info"
         format = "json"
         outputs = ["stdout", "/var/log/app.log"]
      }

      features = [
         { name = "dark-mode"; enabled = true }
         { name = "new-dashboard"; enabled = false }
         { name = "beta-api"; enabled = true }
      ]

   17.2.  Minimal Document

      # The smallest valid HEYAAAAML document is an empty file.
      # This comment makes it slightly less minimal.

   17.3.  Multi-file Setup

      # main.hml
      app-name = "distributed-config"
      @include "database.hml"
      @include "conf.d/*.hml"
