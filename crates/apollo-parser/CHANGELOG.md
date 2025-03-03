# Changelog

All notable changes to `apollo-parser` will be documented in this file.

This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!-- # [x.x.x] (unreleased) - 2021-mm-dd

> Important: X breaking changes below, indicated by **BREAKING**

## BREAKING

## Features

## Fixes

## Maintenance

## Documentation -->
# [0.2.1](https://crates.io/crates/apollo-parser/0.2.1) - 2021-01-26
## Fixes
- **fix(apollo-parser): add ignored tokens to TYPE nodes in correct place - [lrlna], [issue/143] [pull/153]**

  This fixes the location of ignored tokens (COMMA, WHITESPACE) inside a TYPE node.

  Before this commit this sort of query

  ```graphql
  mutation MyMutation($custId: Int!, $b: String) {
    myMutation(custId: $custId)
  }
  ```

  would result the `ast.document.to_string()` to have this output:

  ```graphql
  mutation MyMutation($custId: , Int!$b:  String) {
      myMutation(custId: $custId)
  }
  ```

  which is incorrect. The `to_string()` now results in the exact same output, as
  the AST created is correct.

  [lrlna]: https://github.com/lrlna
  [issue/143]: https://github.com/apollographql/apollo-rs/issues/143
  [pull/153]: https://github.com/apollographql/apollo-rs/pull/153

- **fix(apollo-parser): bump BANG token when creating NON_NULL_TYPE - [lrlna], [issue/142] [pull/146]**

  We are missing BANG token in the AST when a NON_NULL_TYPE gets created.
  Although the node created is indeed NON_NULL_TYPE, it's also important to keep
  the original set of tokens. The followin example now works:

  ```rust
  let mutation = r#"
  mutation MyMutation($custId: Int!) {
    myMutation(custId: $custId)
  }"#;

  let parser = Parser::new(mutation);
  let ast = parser.parse();
  assert_eq!(ast.errors.len(), 0);

  let doc = ast.document();
  assert_eq(&doc, &mutation);
  ```

  [lrlna]: https://github.com/lrlna
  [issue/142]: https://github.com/apollographql/apollo-rs/issues/142
  [pull/146]: https://github.com/apollographql/apollo-rs/pull/146

# [0.2.0](https://crates.io/crates/apollo-parser/0.2.0) - 2021-12-22
## Breaking
- **impl Iterator for ast.errors() - [o0Ignition0o], [issue/119] [pull/120]**

  `ast.errors()` now return an Iterator. This makes it a bit easier for users to process any errors returned by the Parser. Below is the new usage example:

  ```rust

  let query = r#"
  type Query {
      "A simple type for getting started!"
      hello: String
      cats(cat: [String]! = ): [String]!
  }"#;


  let parser = Parser::new(&query);
  let ast = parser.parse();

  assert!(ast.errors.len(), 1);

  for err in ast.errors() { // no longer need to .iter() on this
      // process errors in a way that's useful for your implementation
      dbg!(&err);
  }
  ```

  [o0Ignition0o]: https://github.com/o0Ignition0o
  [issue/119]: https://github.com/apollographql/apollo-rs/issues/119
  [pull/120]: https://github.com/apollographql/apollo-rs/pull/120

## Fixes
- **fix: properly create TYPE's NAMED_TYPE, LIST_TYPE, NON_NULL_TYPE - [lrlna], [issue/125] [pull/127]**

  Whenever a NAMED_TYPED, LIST_TYPE, NON_NULL_TYPE are created, they are
  automatically get created as part of the TYPE node, so we do not need to start
  it manually. This fix makes it possible to once again do:

  ```rust
  if let ast::Type::NamedType(name) = var.ty().unwrap() {
      assert_eq!(name.name().unwrap().text().as_ref(), "Int")
  }
  ```
  [lrlna]: https://github.com/lrlna
  [issue/125]: https://github.com/apollographql/apollo-rs/issues/125
  [pull/127]: https://github.com/apollographql/apollo-rs/pull/127

- **fix: create an error when SelectionSet is empty in operation definition - [lrlna], [pull/134]**

  An Operation Definition must have a selection set with values, so this query
  `query {}` should also come with an error.

  [lrlna]: https://github.com/lrlna
  [pull/134]: https://github.com/apollographql/apollo-rs/pull/134

- **fix: variable definition can have a LIST_TYPE - [lrlna], [issue/131] [pull/135]**

  Variable definition was previously not accepting a LIST_TYPE, which is
  incorrect. This commit fixes this issue.

  [lrlna]: https://github.com/lrlna
  [issue/131]: https://github.com/apollographql/apollo-rs/issues/131
  [pull/135]: https://github.com/apollographql/apollo-rs/pull/135

## Maintenance
- **chore: typo in README - [lrlna], [c598d3]**

  [lrlna]: https://github.com/lrlna
  [c598d3]: https://github.com/apollographql/apollo-rs/commit/c598d33bc9e80f767804bf5a88a7a1d6f400e832

- **fuzzing for apollo-parser - [Geal], [pull/122]**

  The fuzz test checks for lexer and parser errors and stops early.

  The following fuzz-encountered errors are fixed:
  - panics on the following input:
  ```
  "
  ```
  - crash on partial block string opening token
  ```
  ""
  ```
  - infinite loop on unfollowed 'extend' ident

  The parser fuzzer catches errors in the lexer and returns early. It
  will not avoid infinite loops and running out of memory in the lexer.

  [Geal]: https://github.com/Geal
  [pull/122]: https://github.com/apollographql/apollo-rs/pull/122

- **chore: run clippy in CI on benchmark directories - [lrlna], [pull/123]**

  [lrlna]: https://github.com/lrlna
  [pull/123]: https://github.com/apollographql/apollo-rs/pull/123

- **chore: add tests for untermiated strings and invalid type system extensions - [lrlna], [pull/124]**

  Follows up on [#122] and adds tests for the incorrectly lexed and parsed
  inputs that fuzzing discovered.

  This commit also changes logic around having an "unexpected end of data" for
  `""` string. This now gets lexed into a `StringValue` token.

  [lrlna]: https://github.com/lrlna
  [pull/124]: https://github.com/apollographql/apollo-rs/pull/124
  [#122]: https://github.com/apollographql/apollo-rs/pull/122

- **chore: allow dead code in xtask's ast_src - [lrlna], [pull/128]**

  [lrlna]: https://github.com/lrlna
  [pull/128]: https://github.com/apollographql/apollo-rs/pull/128

- **chore: add a test for nested SELECTION_SETs - [lrlna], [pull/137]**

  This will mostly act as an example in case users are looking for how to work
  with nested selections and get their FIELD/INLINE_FRAGMENT/FRAGMENT_SPREAD.

  [lrlna]: https://github.com/lrlna
  [pull/137]: https://github.com/apollographql/apollo-rs/pull/137